# devops-DZ3.4
# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

>1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.
### Ответ
```bash
root@vagrant:/home/vagrant# cat /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
[Service]
User=node_exporter
Group=node_exporter
Type=simple
EnvironmentFile=-/etc/default/node_exporter
ExecStart=/usr/local/bin/node_exporter $OPTIONS
root@vagrant:/home/vagrant# cat /etc/default/node_exporter
####Test config for Node Exporter
OPTIONS="-h"

root@vagrant:/home/vagrant# systemctl daemon-reload
root@vagrant:/home/vagrant# systemctl start node_exporter
vagrant@vagrant:~$ sudo systemctl enable node_exporter
Created symlink /etc/systemd/system/multi-user.target.wants/node_exporter.service → /etc/systemd/system/node_exporter.service.
vagrant@vagrant:~$ sudo systemctl status node_exporter
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-06-28 18:50:16 UTC; 7s ago
   Main PID: 2084 (node_exporter)
      Tasks: 5 (limit: 2278)
     Memory: 2.5M
     CGroup: /system.slice/node_exporter.service
             └─2084 /usr/local/bin/node_exporter

После vagrant reload

vagrant@vagrant:~$ sudo systemctl status node_exporter
vagrant@vagrant:~$ sudo systemctl status node_exporter
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-06-28 19:10:16 UTC; 7s ago
   Main PID: 2084 (node_exporter)
      Tasks: 5 (limit: 2278)
     Memory: 2.5M
     CGroup: /system.slice/node_exporter.service
             └─3042 /usr/local/bin/node_exporter
```
>2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.
### Ответ
```bash
vagrant@vagrant:~$ wget http://localhost:9100/metrics

```
Можно выбрать
node_cpu_seconds_total{cpu="*",mode="user"} 2.8
node_cpu_seconds_total{cpu="*",mode="idle"} 1764.3
node_cpu_seconds_total{cpu="*",mode="iowait"} 1.9
node_memory_MemAvailable_bytes
node_memory_MemFree_bytes
node_disk_read_time_seconds_total
node_disk_write_time_seconds_total
node_network_transmit_errs_total


>3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.
### Ответ

```bash
root@vagrant:/home/vagrant# apt list netdata
Listing... Done
netdata/focal,now 1.19.0-3ubuntu1 all [installed]
```
Ох, много чего собирается ))))

>4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
### Ответ
```bash
```

>5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
### Ответ
```bash
```

6>. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
### Ответ
```bash
```

7>. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?
