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

 Монтирование и размонтирование файловых систем не повлияет на остальную часть системы, за исключением 
 для файловых систем, которые явно помечены как общие (с помощью mount  --make-shared;
 для получения общих флагов смотрите /proc/self/mountinfo  findmnt-o +PROPAGATION)
 
 # Пространство имен UTS
 
 Установка имени хоста или доменного имени не повлияет на остальную часть системы.
 
 # Пространство имен IPC
 
 У процесса будет независимое пространство имен для очередей сообщений POSIX, а также 
 Очереди сообщений System V, наборы семафоров и сегменты общей памяти
 
 # Пользовательское пространство имен
 
 Процесс будет иметь отдельный набор идентификаторов UID, GID и возможностей.
