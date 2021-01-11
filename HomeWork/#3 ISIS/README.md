# ISIS
Лабараторное задание №3.

## Задание
1. [Настроить ISIS в Underlay сети, для IP связанности между всеми устройствами NXOS](#chapter-0)
2. [Проверить роботоспособность](#chapter-1)

<a id="chapter-0"></a>
## Настроить ISIS в Underlay сети, для IP связанности между всеми устройствами NXOS
Стенд разбит на три области:
1. Включает в себя SuperSpine
2. Первый POD 
3. Второй POD
![alt-текст](https://github.com/Thor-VR4/CCNP/blob/main/HomeWork/%233%20ISIS/ISIS_underlay.jpg "Стенд №2")

Для области 1 выбрано взаимодействвие L2. В остальных областях L1.  
Пример настройки области 1 на R8:
```
router isis 1
 net 49.0001.0000.0000.0008.00
 is-type level-2-only
 metric-style wide
!
interface Ethernet0/0
 description N1
 ip address 10.1.0.7 255.255.255.254
 ip router isis 1
 ipv6 address FE80::8 link-local
 ipv6 router isis 1
 isis network point-to-point 
!
```
Пример настройки области 2 и суммаризации на NXOS1:
```
router isis 1
  net 49.0002.0000.0000.0001.00
  distribute level-1 into level-2 all
  summary-address 10.0.1.0/24 level-2
  summary-address 10.0.2.0/24 level-2
  summary-address 192.168.4.0/24 level-2
  summary-address 192.168.5.0/24 level-2
  summary-address 192.168.6.0/24 level-2
  address-family ipv6 unicast
    distribute level-1 into level-2 all

interface Ethernet1/1
  description R8
  no switchport
  medium p2p
  ip address 10.1.0.6/31
  ipv6 link-local fe80::1
  isis circuit-type level-2
  ip router isis 1
  ipv6 router isis 1
  no shutdown
  
interface Ethernet1/2
  description NXOS6
  no switchport
  medium p2p
  ip address 10.0.1.0/31
  ipv6 link-local fe80::1
  isis circuit-type level-1
  ip router isis 1
  ipv6 router isis 1
  no shutdown
```

Пример настройки области 2 на NXOS4:
```
router isis 1
  net 49.0002.0000.0000.0004.00
  is-type level-1
  

interface Ethernet1/1
  description N1
  no switchport
  medium p2p
  ip address 10.0.1.3/31
  ipv6 link-local fe80::4
  ip router isis 1
  ipv6 router isis 1
  no shutdown

interface Ethernet1/5
  description R12
  no switchport
  medium p2p
  ip address 192.168.4.18/31
  ipv6 link-local fe80::4
  ip router isis 1
  ipv6 router isis 1
  isis passive-interface level-1-2
  no shutdown
```
<a id="chapter-1"></a>
## Проверка роботоспособности

Маршрутная информация на NXOS4:
```
NXOS4# show ip route isis-1 

0.0.0.0/0, ubest/mbest: 2/0
    *via 10.0.1.2, Eth1/1, [115/40], 00:33:23, isis-1, L1
    *via 10.0.2.14, Eth1/2, [115/40], 00:33:22, isis-1, L1
10.0.1.0/31, ubest/mbest: 1/0
    *via 10.0.1.2, Eth1/1, [115/80], 00:46:52, isis-1, L1
10.0.1.4/31, ubest/mbest: 1/0
    *via 10.0.1.2, Eth1/1, [115/80], 00:46:52, isis-1, L1
10.0.2.10/31, ubest/mbest: 1/0
    *via 10.0.2.14, Eth1/2, [115/80], 01:32:26, isis-1, L1
10.0.2.22/31, ubest/mbest: 1/0
    *via 10.0.2.14, Eth1/2, [115/80], 01:32:26, isis-1, L1
10.1.0.6/31, ubest/mbest: 1/0
    *via 10.0.1.2, Eth1/1, [115/80], 00:46:52, isis-1, L1
10.1.0.8/31, ubest/mbest: 1/0
    *via 10.0.2.14, Eth1/2, [115/80], 01:32:26, isis-1, L1
172.16.0.1/32, ubest/mbest: 1/0
    *via 10.0.1.2, Eth1/1, [115/41], 00:46:52, isis-1, L1
172.16.0.2/32, ubest/mbest: 1/0
    *via 10.0.2.14, Eth1/2, [115/41], 01:32:26, isis-1, L1
172.16.0.5/32, ubest/mbest: 2/0
    *via 10.0.1.2, Eth1/1, [115/81], 00:46:52, isis-1, L1
    *via 10.0.2.14, Eth1/2, [115/81], 01:30:09, isis-1, L1
172.16.0.6/32, ubest/mbest: 2/0
    *via 10.0.1.2, Eth1/1, [115/81], 00:46:52, isis-1, L1
    *via 10.0.2.14, Eth1/2, [115/81], 01:32:26, isis-1, L1
192.168.5.20/31, ubest/mbest: 2/0
    *via 10.0.1.2, Eth1/1, [115/120], 00:46:52, isis-1, L1
    *via 10.0.2.14, Eth1/2, [115/120], 01:30:09, isis-1, L1
192.168.6.12/31, ubest/mbest: 2/0
    *via 10.0.1.2, Eth1/1, [115/120], 00:46:52, isis-1, L1
    *via 10.0.2.14, Eth1/2, [115/120], 01:32:26, isis-1, L1


NXOS4# show ipv6 route isis-1 

0::/0, ubest/mbest: 2/0
    *via fe80::1, Eth1/1, [115/40], 00:33:43, isis-1, L1
    *via fe80::2, Eth1/2, [115/40], 00:33:42, isis-1, L1
2a03::1/128, ubest/mbest: 1/0
    *via fe80::1, Eth1/1, [115/41], 00:47:12, isis-1, L1
2a03::2/128, ubest/mbest: 1/0
    *via fe80::2, Eth1/2, [115/41], 01:32:46, isis-1, L1
2a03::5/128, ubest/mbest: 2/0
    *via fe80::1, Eth1/1, [115/81], 00:47:12, isis-1, L1
    *via fe80::2, Eth1/2, [115/81], 01:30:29, isis-1, L1
2a03::6/128, ubest/mbest: 2/0
    *via fe80::1, Eth1/1, [115/81], 00:47:12, isis-1, L1
    *via fe80::2, Eth1/2, [115/81], 01:32:46, isis-1, L1
```

Маршрутная информация на R8:
```
R8#show ip route isis 

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 9 subnets, 3 masks
i L2     10.0.1.0/24 [115/50] via 10.1.0.6, 00:27:52, Ethernet0/0
i L2     10.0.2.0/24 [115/50] via 10.1.0.8, 00:26:00, Ethernet0/1
i L2     10.0.3.0/24 [115/50] via 10.1.0.25, 00:23:52, Ethernet0/2
      172.16.0.0/32 is subnetted, 8 subnets
i L2     172.16.0.1 [115/11] via 10.1.0.6, 00:34:09, Ethernet0/0
i L2     172.16.0.2 [115/11] via 10.1.0.8, 00:34:09, Ethernet0/1
i L2     172.16.0.3 [115/11] via 10.1.0.25, 00:34:19, Ethernet0/2
i L2     172.16.0.4 [115/51] via 10.1.0.6, 00:34:09, Ethernet0/0
i L2     172.16.0.5 [115/51] via 10.1.0.6, 00:34:09, Ethernet0/0
i L2     172.16.0.6 [115/51] via 10.1.0.6, 00:34:09, Ethernet0/0
i L2     172.16.0.7 [115/51] via 10.1.0.25, 00:34:19, Ethernet0/2
i L2  192.168.4.0/24 [115/90] via 10.1.0.6, 00:26:58, Ethernet0/0
i L2  192.168.5.0/24 [115/90] via 10.1.0.6, 00:26:52, Ethernet0/0
i L2  192.168.6.0/24 [115/90] via 10.1.0.6, 00:26:47, Ethernet0/0
i L2  192.168.7.0/24 [115/90] via 10.1.0.25, 00:23:10, Ethernet0/2


R8#show ipv6 route isis 

I2  2A03::1/128 [115/11]
     via FE80::1, Ethernet0/0
I2  2A03::2/128 [115/11]
     via FE80::2, Ethernet0/1
I2  2A03::3/128 [115/11]
     via FE80::3, Ethernet0/2
I2  2A03::4/128 [115/51]
     via FE80::1, Ethernet0/0
     via FE80::2, Ethernet0/1
I2  2A03::5/128 [115/51]
     via FE80::1, Ethernet0/0
     via FE80::2, Ethernet0/1
I2  2A03::6/128 [115/51]
     via FE80::1, Ethernet0/0
     via FE80::2, Ethernet0/1
I2  2A03::7/128 [115/51]
     via FE80::3, Ethernet0/2
```

Проверка сввязанности между NXOS6 и NXOS7:
```
NXOS6# ping 172.16.0.7
PING 172.16.0.7 (172.16.0.7): 56 data bytes
64 bytes from 172.16.0.7: icmp_seq=0 ttl=251 time=170.118 ms
64 bytes from 172.16.0.7: icmp_seq=1 ttl=251 time=53.311 ms
64 bytes from 172.16.0.7: icmp_seq=2 ttl=251 time=59.591 ms
64 bytes from 172.16.0.7: icmp_seq=3 ttl=251 time=20.361 ms
64 bytes from 172.16.0.7: icmp_seq=4 ttl=251 time=75.202 ms

--- 172.16.0.7 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 20.361/75.716/170.118 ms



NXOS7# traceroute6  2A03::6
traceroute to 2a03::6 (2a03::6), 30 hops max, 16 byte packets
 1  2a03::3  64.458 ms  6.401 ms  8.169 ms
 2  2a03::8  22.377 ms  6.959 ms  15.289 ms
 3  2a03::1  39.05 ms 2a03::2  53.299 ms 2a03::1  11.477 ms
 4  2a03::6  60.078 ms  29.781 ms  32.237 ms
```

