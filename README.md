# lesson_5
Практические навыки работы с ZFS
# Домашнее задание
Практические навыки работы с ZFS
# Исполнитель
Павел Смирнов
# Цель
научится самостоятельно устанавливать ZFS, настраивать пулы, изучить основные возможности ZFS;

Описание:
1. Определить алгоритм с наилучшим сжатием:
  - определить, какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);
  - создать 4 файловых системы, на каждой применить свой алгоритм сжатия;
  - для сжатия использовать либо текстовый файл, либо группу файлов.
2. Определить настройки пула.
  - С помощью команды zfs import собрать pool ZFS.
  Командами zfs определить настройки:
  - размер хранилища;
  - тип pool;
  - значение recordsize;
  - какое сжатие используется;
  - какая контрольная сумма используется.
3. Работа со снапшотами:
  - скопировать файл из удаленной директории;
  - восстановить файл локально. zfs receive;
  - найти зашифрованное сообщение в файле secret_message.

# Среда выполнения
  VM VirtualBox 7.0.10, OS Ubuntu 24.04.03

# Команды и их описание

root@srv1:~# apt list --installed | grep zfs

root@srv1:~# zpool list

-- смотрим, что нужный пакет у нас стоит и zfs есть

root@srv1:~# lsblk

-- смотрим блочные устройства, выбираем диски под  четыре пула, raid1

root@srv1:~# zpool create test_pool1 mirror /dev/sdb /dev/sdc

root@srv1:~# zpool create test_pool2 mirror /dev/sdd /dev/sde

root@srv1:~# zpool create test_pool3 mirror /dev/sdf /dev/sdg

root@srv1:~# zpool create test_pool4 mirror /dev/sdh /dev/sdi

-- создали четыре пула

root@srv1:~# zpool list

root@srv1:~# zfs set compression=lzjb test_pool1

-- установили локальный параметр, алгоритм сжатия lzjb для фс test_pool1

root@srv1:~# zfs set compression=lz4 test_pool2

-- установили локальный параметр, алгоритм сжатия lz4 для фс test_pool2

root@srv1:~# zfs set compression=gzip-9 test_pool3

-- установили локальный параметр, алгоритм сжатия gzip-9 для фс test_pool3

root@srv1:~# zfs set compression=zle test_pool4

-- установили локальный параметр, алгоритм сжатия zle для фс test_pool4

root@srv1:~# zfs get all | grep compression

-- проверили, что установили то что нужно

root@srv1:~# for i in {1..4}; do wget -P /test_pool$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done

-- в корень каждой фс копируем с сайта файл pg2600.converter.log для проверки степени сжатия разных алгоритмов

root@srv1:~# ls -l /test_pool*

-- смотрим что получилось, замечаем что итоговый размер в папке каждой фс разный

root@srv1:~# zfs list

-- здесь явно видно что наименьшее используемое пространство занимает фс test_pool3 с алгоритмом сжатия gzip-9, то есть он сжимает опытные данные лучше всех остальных

root@srv1:~# zfs get all | grep compressratio | grep -v ref

-- смотрим % сжания для наших фс

-- для определения настроек zfs скачаем скачаем с сайта конфигурацию собранную на файлах - "дисках"

root@srv1:~# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-
WgAQe57aDEzxSRalPAwbNN1Bb&export=download'

root@srv1:~# tar -xzvf archive.tar.gz

-- развернем скачанный архив в текущую папку

root@srv1:~# ls -l ./zpoolexport/

root@srv1:~# zpool import -d zpoolexport/

-- проверим что файлы корректны и готовы к импорту

root@srv1:~# zpool import -d zpoolexport/ otus

-- импортируем конфигурацию в zpool otus

root@srv1:~# zfs get available otus

-- смотрим размер хранилища, 350Мб

root@srv1:~# zfs get readonly otus

-- смотрим его тип, чтение/запись

root@srv1:~# zfs get recordsize otus

-- смотрим размер логического блока, 128Кб

root@srv1:~# zfs get compression otus

-- смотрим алгоритм сжатия, zle

root@srv1:~# zfs get checksum otus

-- смотрим какая используется контрольная сумма, sha256

root@srv1:~# wget -O otus_task2.file --no-check-certificate 'https://drive.usercontent.google.com/download?id=1wgxjih8YZ-
cqLqaZVa0lA3h3Y029c3oI&export=download'

-- для тестирования снапшота с сайта скопируем снапшот для фс otus/test выгруженный в файл

root@srv1:~# ls

root@srv1:~# zfs receive otus/test@today < otus_task2.file

-- восстанавливаем снапшот

root@srv1:~# ls /otus

root@srv1:~# ls /otus/test

root@srv1:~# find /otus/test -name "secret_message"

-- проверяем восстановление, ищем файл secret_message

root@srv1:~# cat /otus/test/task1/file_mess/secret_message

-- выводим его содержимоем, должны получить веб-адрес
-- получили https://otus.ru/lessons/linux-hl/


# Протокол выполнения

root@srv1:~# apt list --installed | grep zfs

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

libzfs4linux/noble-updates,now 2.2.2-0ubuntu9.4 amd64 [installed,automatic]

zfs-zed/noble-updates,now 2.2.2-0ubuntu9.4 amd64 [installed,automatic]

zfsutils-linux/noble-updates,now 2.2.2-0ubuntu9.4 amd64 [installed]

root@srv1:~# zpool list

no pools available

root@srv1:~# lsblk

NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS

sda                         8:0    0   30G  0 disk

├─sda1                      8:1    0    1M  0 part

├─sda2                      8:2    0    2G  0 part /boot

└─sda3                      8:3    0   28G  0 part

  ├─ubuntu--vg-ubuntu--lv 252:8    0    8G  0 lvm  /
  
  └─ubuntu--vg-home--lv   252:9    0    2G  0 lvm  /home
  
sdb                         8:16   0    1G  0 disk

sdc                         8:32   0    1G  0 disk

sdd                         8:48   0    1G  0 disk

sde                         8:64   0    1G  0 disk

sdf                         8:80   0    1G  0 disk

sdg                         8:96   0    1G  0 disk

sdh                         8:112  0    1G  0 disk

sdi                         8:128  0    1G  0 disk

sdj                         8:144  0    1G  0 disk

sdk                         8:160  0   10G  0 disk

sdl                         8:176  0    2G  0 disk

├─vg_var-lv_var_rmeta_0   252:0    0    4M  0 lvm

│ └─vg_var-lv_var         252:4    0  1.1G  0 lvm  /var

└─vg_var-lv_var_rimage_0  252:1    0  1.1G  0 lvm

  └─vg_var-lv_var         252:4    0  1.1G  0 lvm  /var

sdm                         8:192  0    2G  0 disk

├─vg_var-lv_var_rmeta_1   252:2    0    4M  0 lvm

│ └─vg_var-lv_var         252:4    0  1.1G  0 lvm  /var

└─vg_var-lv_var_rimage_1  252:3    0  1.1G  0 lvm

  └─vg_var-lv_var         252:4    0  1.1G  0 lvm  /var

sr0                        11:0    1 1024M  0 rom

root@srv1:~# zpool create test_pool1 mirror /dev/sdb /dev/sdc

root@srv1:~# zpool create test_pool2 mirror /dev/sdd /dev/sde

root@srv1:~# zpool create test_pool3 mirror /dev/sdf /dev/sdg

root@srv1:~# zpool create test_pool4 mirror /dev/sdh /dev/sdi

root@srv1:~# zpool list

NAME         SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT

test_pool1   960M   108K   960M        -         -     0%     0%  1.00x    ONLINE  -

test_pool2   960M   108K   960M        -         -     0%     0%  1.00x    ONLINE  -

test_pool3   960M   111K   960M        -         -     0%     0%  1.00x    ONLINE  -

test_pool4   960M   111K   960M        -         -     0%     0%  1.00x    ONLINE  -

root@srv1:~#

root@srv1:~# zfs set compression=lzjb test_pool1

root@srv1:~# zfs set compression=lz4 test_pool2

root@srv1:~# zfs set compression=gzip-9 test_pool3

root@srv1:~# zfs set compression=zle test_pool4

root@srv1:~# zfs get all | grep compression

test_pool1  compression           lzjb                   local

test_pool2  compression           lz4                    local

test_pool3  compression           gzip-9                 local

test_pool4  compression           zle                    local

root@srv1:~#

root@srv1:~# for i in {1..4}; do wget -P /test_pool$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done

--2026-07-17 21:56:43--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log

Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47

Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 41250559 (39M) [text/plain]

Saving to: ‘/test_pool1/pg2600.converter.log’

pg2600.converter.log        100%[===========================================>]  39.34M  2.20MB/s    in 19s

2026-07-17 21:57:03 (2.04 MB/s) - ‘/test_pool1/pg2600.converter.log’ saved [41250559/41250559]

--2026-07-17 21:57:03--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log

Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47

Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 41250559 (39M) [text/plain]

Saving to: ‘/test_pool2/pg2600.converter.log’

pg2600.converter.log        100%[===========================================>]  39.34M  2.22MB/s    in 31s

2026-07-17 21:57:35 (1.26 MB/s) - ‘/test_pool2/pg2600.converter.log’ saved [41250559/41250559]

--2026-07-17 21:57:35--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log

Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47

Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 41250559 (39M) [text/plain]

Saving to: ‘/test_pool3/pg2600.converter.log’

pg2600.converter.log        100%[===========================================>]  39.34M  3.14MB/s    in 11s

2026-07-17 21:57:47 (3.54 MB/s) - ‘/test_pool3/pg2600.converter.log’ saved [41250559/41250559]

--2026-07-17 21:57:47--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log

Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47

Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 41250559 (39M) [text/plain]

Saving to: ‘/test_pool4/pg2600.converter.log’

pg2600.converter.log        100%[===========================================>]  39.34M  5.69MB/s    in 7.5s

2026-07-17 21:57:56 (5.26 MB/s) - ‘/test_pool4/pg2600.converter.log’ saved [41250559/41250559]

root@srv1:~# ls -l /test_pool*

/test_pool1:

total 22128

-rw-r--r-- 1 root root 41250559 Jul  2 07:31 pg2600.converter.log

/test_pool2:

total 18021

-rw-r--r-- 1 root root 41250559 Jul  2 07:31 pg2600.converter.log

/test_pool3:

total 10973

-rw-r--r-- 1 root root 41250559 Jul  2 07:31 pg2600.converter.log

/test_pool4:

total 40312

-rw-r--r-- 1 root root 41250559 Jul  2 07:31 pg2600.converter.log

root@srv1:~#

root@srv1:~# zfs list

NAME         USED  AVAIL  REFER  MOUNTPOINT

test_pool1  21.8M   810M  21.6M  /test_pool1

test_pool2  17.7M   814M  17.6M  /test_pool2

test_pool3  10.9M   821M  10.7M  /test_pool3

test_pool4  39.5M   792M  39.4M  /test_pool4

root@srv1:~# zfs get all | grep compressratio | grep -v ref

test_pool1  compressratio         1.82x                  -

test_pool2  compressratio         2.23x                  -

test_pool3  compressratio         3.66x                  -

test_pool4  compressratio         1.00x                  -

root@srv1:~#

root@srv1:~# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-

WgAQe57aDEzxSRalPAwbNN1Bb&export=download'

--2026-07-18 19:44:04--  https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download

Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 142.251.38.97, 2a00:1450:400f:806::2001

Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|142.251.38.97|:443... connected.

HTTP request sent, awaiting response... 200 OK

Length: 7275140 (6.9M) [application/octet-stream]

Saving to: ‘archive.tar.gz’

archive.tar.gz      100%[===================>]   6.94M  7.98MB/s    in 0.9s

2026-07-18 19:44:11 (7.98 MB/s) - ‘archive.tar.gz’ saved [7275140/7275140]

root@srv1:~# tar -xzvf archive.tar.gz

zpoolexport/

zpoolexport/filea

zpoolexport/fileb

root@srv1:~# ls -l

total 7112

-rw-r--r-- 1 root root 7275140 Dec  6  2023 archive.tar.gz

drwxr-xr-x 2 root root    4096 May 15  2020 zpoolexport

root@srv1:~# ls -l ./zpoolexport/

total 1024008

-rw-r--r-- 1 root root 524288000 May 15  2020 filea

-rw-r--r-- 1 root root 524288000 May 15  2020 fileb

root@srv1:~# zpool import -d zpoolexport/

   pool: otus
   
     id: 6554193320433390805
  
  state: ONLINE

status: Some supported features are not enabled on the pool.
        
        (Note that they may be intentionally disabled if the
        
        'compatibility' property is set.)
 
 action: The pool can be imported using its name or numeric identifier, though
        
        some features will not be available without an explicit 'zpool upgrade'.
 
 config:

        otus                         ONLINE

          mirror-0                   ONLINE
          
            /root/zpoolexport/filea  ONLINE
            
            /root/zpoolexport/fileb  ONLINE

root@srv1:~# zpool import -d zpoolexport/ otus

root@srv1:~# zfs get available otus

NAME  PROPERTY   VALUE  SOURCE

otus  available  350M   -

root@srv1:~# zfs get readonly otus

NAME  PROPERTY  VALUE   SOURCE

otus  readonly  off     default

root@srv1:~# zfs get recordsize otus

NAME  PROPERTY    VALUE    SOURCE

otus  recordsize  128K     local

root@srv1:~# zfs get compression otus

NAME  PROPERTY     VALUE           SOURCE

otus  compression  zle             local

root@srv1:~# zfs get checksum otus

NAME  PROPERTY  VALUE      SOURCE

otus  checksum  sha256     local

root@srv1:~# wget -O otus_task2.file --no-check-certificate 'https://drive.usercontent.google.com/download?id=1wgxjih8YZ-
cqLqaZVa0lA3h3Y029c3oI&export=download'
--2026-07-18 20:02:52--  https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 142.251.38.97, 2a00:1450:400f:806::2001
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|142.251.38.97|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5432736 (5.2M) [application/octet-stream]
Saving to: ‘otus_task2.file’

otus_task2.file     100%[===================>]   5.18M  7.63MB/s    in 0.7s

2026-07-18 20:02:55 (7.63 MB/s) - ‘otus_task2.file’ saved [5432736/5432736]

root@srv1:~# ls

archive.tar.gz  otus_task2.file  wget-log  zpoolexport

root@srv1:~# zfs receive otus/test@today < otus_task2.file

root@srv1:~# ls /otus

hometask2  test

root@srv1:~# ls /otus/test

10M.file        for_examaple.txt  Limbo.txt      task1              world.sql

cinderella.tar  homework4.txt     Moby_Dick.txt  War_and_Peace.txt

root@srv1:~# find /otus/test -name "secret_message"

/otus/test/task1/file_mess/secret_message

/otus/test/task1/file_mess/secret_message

-bash: /otus/test/task1/file_mess/secret_message: Permission denied

root@srv1:~# cat /otus/test/task1/file_mess/secret_message

https://otus.ru/lessons/linux-hl/

root@srv1:~#

