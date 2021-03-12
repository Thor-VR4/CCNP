# PIM
Лабараторное задание №5.

## Задание
1. [Настроите PIM на устройствах](#chapter-0)
2. [Проверить роботоспособность](#chapter-1)
![alt-текст](https://github.com/Thor-VR4/CCNP/blob/main/HomeWork/%231%20IP/Net.png "Стенд №1")

<a id="chapter-0"></a>
## Настроите PIM на устройствах
Для настройки выбран протокол PIM-SM c Anycast-RP. В качестве RP используються Spine.  
Пример настройки на NXOS1:
```
feature pim
ip pim rp-address 10.10.10.10
interface loopback 1
 description RP
 ip address 10.10.10.10/32
 ip pim sparse-mode
interface loopback 2
 ip address 10.10.10.1/32
 ip pim sparse-mode
ip pim anycast-rp 10.10.10.10 10.10.10.1
ip pim anycast-rp 10.10.10.10 10.10.10.2
```

<a id="chapter-1"></a>
## Проверка роботоспособности

На R9 подключаемся к группе 234.5.2.1:
```
inter e0/0
 ip igmp join-group 234.5.2.1
```
Проверяем роботоспособность с R12 при помощи icmp:
```
R12#ping 234.5.2.1 repeat 10 
Type escape sequence to abort.
Sending 10, 100-byte ICMP Echos to 234.5.2.1, timeout is 2 seconds:
.
Reply to request 1 from 192.168.6.13, 41 ms
Reply to request 1 from 192.168.6.13, 47 ms
Reply to request 2 from 192.168.6.13, 38 ms
Reply to request 2 from 192.168.6.13, 51 ms
Reply to request 3 from 192.168.6.13, 139 ms
Reply to request 3 from 192.168.6.13, 140 ms
Reply to request 4 from 192.168.6.13, 214 ms
Reply to request 4 from 192.168.6.13, 214 ms
Reply to request 5 from 192.168.6.13, 241 ms
Reply to request 5 from 192.168.6.13, 254 ms
Reply to request 6 from 192.168.6.13, 476 ms
Reply to request 6 from 192.168.6.13, 509 ms
R12#
```