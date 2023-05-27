# Пространство имен NET
```
root@linux-gb:/home/rust# unshare --net bash
root@linux-gb:/home/rust# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
root@linux-gb:/home/rust# exit
exit
root@linux-gb:/home/rust# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether ------------ brd ff:ff:ff:ff:ff:ff
    inet ------------ brd ------------ scope global dynamic noprefixroute enp0s3
       valid_lft 85936sec preferred_lft 85936sec
    inet6 ------------ scope link noprefixroute
       valid_lft forever preferred_lft forever
3: lxcbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether ------------ brd ff:ff:ff:ff:ff:ff
    inet ------------ brd ------------ scope global lxcbr0
       valid_lft forever preferred_lft forever
    inet6 ------------ scope link
       valid_lft forever preferred_lft forever
5: lxdbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether ------------ brd ff:ff:ff:ff:ff:ff
    inet ------------ scope global lxdbr0
       valid_lft forever preferred_lft forever
    inet6 ------------ scope global
       valid_lft forever preferred_lft forever
root@linux-gb:/home/rust#
```
Процессы внутри определенного сетевого пространства имен получают собственный закрытый сетевой стек, включая сетевые интерфейсы, таблицы маршрутов, правила iptables, сокеты (ss, netstat);
Соединение между сетевыми пространствами имен можно реализовать с помощью двух виртуальных интерфейсов;
Взаимодействие между изолированными сетевыми стеками в одном пространстве имен реализуется с помощью моста.
  
# Пространство имен PID
```  
rust@linux-gb:~$ sudo unshare --fork --pid --mount-proc  /bin/bash
root@linux-gb:/home/rust# ps
    PID TTY          TIME CMD
      1 pts/1    00:00:00 bash
      7 pts/1    00:00:00 ps
root@linux-gb:/home/rust# exit
exit
rust@linux-gb:~$ sudo unshare --pid --fork --mount /bin/bash
root@linux-gb:/home/rust# ps
    PID TTY          TIME CMD
   3911 pts/1    00:00:00 sudo
   3912 pts/1    00:00:00 unshare
   3913 pts/1    00:00:00 bash
   3919 pts/1    00:00:00 ps
root@linux-gb:/home/rust# exit
exit
rust@linux-gb:~$ ps
    PID TTY          TIME CMD
   2309 pts/0    00:00:00 bash
   3940 pts/0    00:00:00 ps
rust@linux-gb:~$
```
процессы внутри пространства имен видят и могут взаимодействовать только с процессами в том же пространстве имен PID (изоляция);
каждое пространство имен PID имеет собственную нумерацию, начинающуюся с 1;
эта нумерация уникальна для каждого пространства имен – если PID 1 исчезает, тогда все пространство имен удаляется;
пространства имен могут вкладываться друг в друга;
в случае вложения у процесса получается несколько PID;
Все ps-подобные команды для реализации своей функциональности используют монтирование виртуальной файловой системы procfs.

#  Смонтировать пространство имен
```
Шаг 1: Создайте новый каталог в /tmp

[root@linuxnix ~]# mkdir /tmp/mount_namespace
[root@linuxnix ~]#
Шаг 2: Затем переместите текущий процесс bash в его собственное пространство имен mount, передав флаг mount команде unshare, как показано ниже

[root@linuxnix ~]# unshare -m /bin/bash
[root@linuxnix ~]#
Шаг 3: Процесс bash теперь находится в отдельном пространстве имен. Давайте проверим связанный номер индексного индекса пространства имен

[root@linuxnix ~]# readlink /proc/$/ns/mnt
mnt:[4026532169]
Шаг 4: Создайте временную точку монтирования и убедитесь, что она доступна

[root@linuxnix ~]# монтирование -n -t tmpfs tmpfs /tmp/mount_namespace
[root@linuxnix ~]# df -hTP /tmp/mount_namespace
Тип используемой файловой системы, используемый размер, % смонтировано на
tmpfs 484M 0 484M 0% /tmp/mount_namespace
[root@linuxnix ~]# cat / proc /монтирует | grep /tmp/mount_namespace
tmpfs /tmp/mount_namespace tmpfs rw, seclabel, relatime 0 0
[root@linuxnix ~]#
Шаг 5: Теперь запустите новый сеанс терминала и выведите из него идентификатор индекса пространства имен:

[root@linuxnix ~]# readlink /proc/$/ns/mnt
mnt:[4026531840]
[root@linuxnix ~]#
Обратите внимание, что это значение отличается от значения в другом терминале.

Шаг 6: Давайте проверим, видна ли точка монтирования в новом сеансе терминала

[root@linuxnix ~]# df -hTP /tmp/mount_namespace
Тип используемой файловой системы, используемый размер, % Смонтировано на
/dev/nvme0n1p1 xfs 20G 7.1G 13G 36% /
[root@linuxnix ~]# cat /proc/mounts | grep /tmp/mount_namespace
[root@linuxnix ~]#
Как и ожидалось, точка монтирования не видна в пространстве имен по умолчанию. В контексте LXC пространства имен монтирования полезны, поскольку они обеспечивают возможность существования внутри контейнера другого макета файловой системы.
```
 Монтирование и размонтирование файловых систем не повлияет на остальную часть системы, за исключением 
 для файловых систем, которые явно помечены как общие (с помощью mount  --make-shared;
 для получения общих флагов смотрите /proc/self/mountinfo  findmnt-o +PROPAGATION)
 
 # Пространство имен UTS
 ```
 root@linuxnix ~]# имя хоста
linuxnix
[root@linuxnix ~]# unshare -u /bin/bash
[root@linuxnix ~]# имя хоста uts
[root@linuxnix ~]# имя хоста
uts
[root@linuxnix ~]# имя хоста
uts[root@linuxnix ~]# cat /proc/sys/kernel/имя хоста в ядре - это то, что нужно для создания пространства имен в Linux
 ```
 Установка имени хоста или доменного имени не повлияет на остальную часть системы.
 
 # Пространство имен IPC
 
 У процесса будет независимое пространство имен для очередей сообщений POSIX, а также 
 Очереди сообщений System V, наборы семафоров и сегменты общей памяти
 
 # Пользовательское пространство имен
 
 Процесс будет иметь отдельный набор идентификаторов UID, GID и возможностей.
