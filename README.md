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
```
node_cpu_seconds_total{cpu="*",mode="user"}
node_cpu_seconds_total{cpu="*",mode="idle"} 
node_cpu_seconds_total{cpu="*",mode="iowait"}
node_memory_MemAvailable_bytes
node_memory_MemFree_bytes
node_disk_read_time_seconds_total
node_disk_write_time_seconds_total
node_network_transmit_errs_total
```

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

Видимо, да, осознает
```bash
vagrant@vagrant:~$ dmesg | grep -i virt
[    0.000000] DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
[    0.006701] CPU MTRRs all blank - virtualized system.
[    0.053142] Booting paravirtualized kernel on KVM
[    0.494018] Performance Events: PMU not available due to virtualization, using software events only.
[   19.904456] systemd[1]: Detected virtualization oracle.
```

>5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
### Ответ
```bash
vagrant@vagrant:~$ /sbin/sysctl -n fs.nr_open
1048576
```
Это лимит максимального количества открытых файлов в системе.


```bash
vagrant@vagrant:~$ ulimit --help | grep open
      -n        the maximum number of open file descriptors
 vagrant@vagrant:~$ ulimit -n
1024
```
 ulimit определяет лимиты на пользовательские процессы
              
6>. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
### Ответ
```bash
root@vagrant:/home/vagrant# unshare -f --pid --mount-proc tail -f /etc/hosts &
[1] 3074
root@vagrant:/home/vagrant# ps -ef  | grep tail | grep hosts
root        3074    3067  0 15:59 pts/1    00:00:00 unshare -f --pid --mount-proc tail -f /etc/hosts
root        3075    3074  0 15:59 pts/1    00:00:00 tail -f /etc/hosts
root@vagrant:/home/vagrant# nsenter --target 3075 --pid --mount
root@vagrant:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   5512   592 pts/1    S    15:59   0:00 tail -f /etc/hosts
root           2  0.0  0.2   7236  4084 pts/1    S    16:01   0:00 -bash
root          13  0.0  0.1   8892  3408 pts/1    R+   16:01   0:00 ps aux
```

7>. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

###Ответ

Концепция Fork Bomb — вредоносная программа, которая порождает себя n-раз, отбросив цепную реакцию (рекурсия) и тем самым быстро исчерпав ресурсы системы.
В более читаемом виде приведенный код на bash можно представить как 
```bash
bomb() {
   bomb | bomb &
   };
bomb
```
