# OSPF
Лабараторное задание №2.

## Задание
1. [Настроить OSPF в Underlay сети, для IP связанности между всеми устройствами NXOS](#chapter-0)
2. [Проверить роботоспособность](#chapter-1)

<a id="chapter-0"></a>
## Настроить OSPF в Underlay сети, для IP связанности между всеми устройствами NXOS
Стенд разбит на три области:
0. Включает в себя SuperSpine и Spine
1. Первый POD 
2. Второй POD
![alt-текст](https://github.com/Thor-VR4/CCNP/blob/main/HomeWork/%232%20OSPF/OSPF_underlay.jpg "Стенд №1")

Т.к. в доступной версии NXOS нет поддержки IPv4 address family for OSPFv3, то реализована схема OSPFv2 + OSPFv3.  
Пример настройки на R8:
```
!
router ospfv3 1
!
 address-family ipv6 unicast
  router-id 0.0.0.8
 exit-address-family
!
router ospf 2
 router-id 0.0.0.8
!
```
Зоны, включающие в себя POD, созданы как Totally stubby.
Пример настройки области и суммаризации на NXOS1:
```
router ospf 2
  router-id 0.0.0.1
  area 0.0.0.1 stub no-summary
  area 0.0.0.1 range 10.0.1.0/24
  area 0.0.0.1 range 10.0.2.0/24
  area 0.0.0.1 range 192.168.4.0/24
  area 0.0.0.1 range 192.168.5.0/24
  area 0.0.0.1 range 192.168.6.0/24
router ospfv3 1
  router-id 0.0.0.1
  area 0.0.0.1 stub no-summary
  address-family ipv6 unicast
```

<a id="chapter-1"></a>
## Проверка роботоспособности

Маршрутная информация на NXOS2:
```
NXOS2# show ip route  ospf-2 

10.0.1.0/24, ubest/mbest: 1/0
    *via Null0, [220/80], 22:54:25, ospf-2, discard
10.0.1.0/31, ubest/mbest: 1/0
    *via 10.0.2.11, Eth1/2, [110/80], 23:35:30, ospf-2, intra
10.0.1.2/31, ubest/mbest: 1/0
    *via 10.0.2.15, Eth1/3, [110/80], 23:27:46, ospf-2, intra
10.0.1.4/31, ubest/mbest: 1/0
    *via 10.0.2.22, Eth1/4, [110/80], 23:27:46, ospf-2, intra
10.0.2.0/24, ubest/mbest: 1/0
    *via Null0, [220/40], 22:54:20, ospf-2, discard
10.0.3.26/31, ubest/mbest: 1/0
    *via 10.1.0.9, Eth1/1, [110/90], 23:23:24, ospf-2, inter
10.0.4.16/31, ubest/mbest: 2/0
    *via 10.0.2.15, Eth1/3, [110/80], 23:28:18, ospf-2, intra
    *via 10.0.2.22, Eth1/4, [110/80], 23:28:18, ospf-2, intra
10.1.0.6/31, ubest/mbest: 1/0
    *via 10.1.0.9, Eth1/1, [110/50], 23:57:51, ospf-2, intra
10.1.0.24/31, ubest/mbest: 1/0
    *via 10.1.0.9, Eth1/1, [110/50], 23:57:51, ospf-2, intra
172.16.0.1/32, ubest/mbest: 3/0
    *via 10.0.2.11, Eth1/2, [110/81], 23:28:18, ospf-2, intra
    *via 10.0.2.15, Eth1/3, [110/81], 23:28:18, ospf-2, intra
    *via 10.0.2.22, Eth1/4, [110/81], 23:28:18, ospf-2, intra
172.16.0.3/32, ubest/mbest: 1/0
    *via 10.1.0.9, Eth1/1, [110/51], 23:23:26, ospf-2, inter
172.16.0.4/32, ubest/mbest: 1/0
    *via 10.0.2.15, Eth1/3, [110/41], 23:27:46, ospf-2, intra
172.16.0.5/32, ubest/mbest: 1/0
    *via 10.0.2.22, Eth1/4, [110/41], 23:27:46, ospf-2, intra
172.16.0.6/32, ubest/mbest: 1/0
    *via 10.0.2.11, Eth1/2, [110/41], 23:35:30, ospf-2, intra
172.16.0.7/32, ubest/mbest: 1/0
    *via 10.1.0.9, Eth1/1, [110/91], 23:14:35, ospf-2, inter
172.16.0.8/32, ubest/mbest: 1/0
    *via 10.1.0.9, Eth1/1, [110/41], 23:57:51, ospf-2, intra
192.168.4.0/24, ubest/mbest: 1/0
    *via Null0, [220/80], 22:55:25, ospf-2, discard
192.168.4.18/31, ubest/mbest: 1/0
    *via 10.0.2.15, Eth1/3, [110/80], 23:17:34, ospf-2, intra
192.168.5.0/24, ubest/mbest: 1/0
    *via Null0, [220/80], 22:55:18, ospf-2, discard
192.168.5.20/31, ubest/mbest: 1/0
    *via 10.0.2.22, Eth1/4, [110/80], 23:17:15, ospf-2, intra
192.168.6.0/24, ubest/mbest: 1/0
    *via Null0, [220/80], 22:55:31, ospf-2, discard
192.168.6.12/31, ubest/mbest: 1/0
    *via 10.0.2.11, Eth1/2, [110/80], 23:35:30, ospf-2, intra
192.168.7.28/31, ubest/mbest: 1/0
    *via 10.1.0.9, Eth1/1, [110/130], 23:14:24, ospf-2, inter
	
	

NXOS2# show ipv6 route  ospf

2a03::1/128, ubest/mbest: 3/0
    *via fe80::4, Eth1/3, [110/80], 23:28:59, ospfv3-1, intra
    *via fe80::5, Eth1/4, [110/80], 23:28:59, ospfv3-1, intra
    *via fe80::6, Eth1/2, [110/80], 23:28:59, ospfv3-1, intra
2a03::3/128, ubest/mbest: 1/0
    *via fe80::8, Eth1/1, [110/50], 23:24:43, ospfv3-1, inter
2a03::4/128, ubest/mbest: 1/0
    *via fe80::4, Eth1/3, [110/40], 23:29:28, ospfv3-1, intra
2a03::5/128, ubest/mbest: 1/0
    *via fe80::5, Eth1/4, [110/40], 23:29:02, ospfv3-1, intra
2a03::6/128, ubest/mbest: 1/0
    *via fe80::6, Eth1/2, [110/40], 23:36:20, ospfv3-1, intra
2a03::7/128, ubest/mbest: 1/0
    *via fe80::8, Eth1/1, [110/90], 23:15:47, ospfv3-1, inter
2a03::8/128, ubest/mbest: 1/0
    *via fe80::8, Eth1/1, [110/40], 23:59:08, ospfv3-1, intra	
```

Маршрутная информация на R8:
```
R8#show ip route ospf
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 10 subnets, 3 masks
O IA     10.0.1.0/24 [110/50] via 10.1.0.6, 22:57:34, Ethernet0/0
O IA     10.0.2.0/24 [110/50] via 10.1.0.8, 22:57:13, Ethernet0/1
O IA     10.0.3.26/31 [110/50] via 10.1.0.25, 23:26:18, Ethernet0/2
O IA     10.0.4.16/31 [110/90] via 10.1.0.8, 23:32:03, Ethernet0/1
                      [110/90] via 10.1.0.6, 23:32:06, Ethernet0/0
      172.16.0.0/32 is subnetted, 8 subnets
O IA     172.16.0.1 [110/11] via 10.1.0.6, 1d00h, Ethernet0/0
O IA     172.16.0.2 [110/11] via 10.1.0.8, 1d00h, Ethernet0/1
O IA     172.16.0.3 [110/11] via 10.1.0.25, 23:26:20, Ethernet0/2
O IA     172.16.0.4 [110/51] via 10.1.0.8, 23:31:18, Ethernet0/1
                    [110/51] via 10.1.0.6, 23:31:24, Ethernet0/0
O IA     172.16.0.5 [110/51] via 10.1.0.8, 23:30:39, Ethernet0/1
                    [110/51] via 10.1.0.6, 23:30:41, Ethernet0/0
O IA     172.16.0.6 [110/51] via 10.1.0.8, 23:38:24, Ethernet0/1
                    [110/51] via 10.1.0.6, 23:38:29, Ethernet0/0
O IA     172.16.0.7 [110/51] via 10.1.0.25, 23:17:29, Ethernet0/2
O IA  192.168.4.0/24 [110/90] via 10.1.0.8, 22:58:18, Ethernet0/1
                     [110/90] via 10.1.0.6, 22:58:52, Ethernet0/0
O IA  192.168.5.0/24 [110/90] via 10.1.0.8, 22:58:11, Ethernet0/1
                     [110/90] via 10.1.0.6, 22:58:46, Ethernet0/0
O IA  192.168.6.0/24 [110/90] via 10.1.0.8, 22:58:24, Ethernet0/1
                     [110/90] via 10.1.0.6, 22:59:00, Ethernet0/0
      192.168.7.0/31 is subnetted, 1 subnets
O IA     192.168.7.28 [110/90] via 10.1.0.25, 23:17:17, Ethernet0/2


R8#show ipv6 route ospf

OI  2A03::1/128 [110/10]
     via FE80::1, Ethernet0/0
OI  2A03::2/128 [110/10]
     via FE80::2, Ethernet0/1
OI  2A03::3/128 [110/10]
     via FE80::3, Ethernet0/2
OI  2A03::4/128 [110/50]
     via FE80::1, Ethernet0/0
     via FE80::2, Ethernet0/1
OI  2A03::5/128 [110/50]
     via FE80::2, Ethernet0/1
     via FE80::1, Ethernet0/0
OI  2A03::6/128 [110/50]
     via FE80::2, Ethernet0/1
     via FE80::1, Ethernet0/0
OI  2A03::7/128 [110/50]
     via FE80::3, Ethernet0/2
```

Проверка сввязанности между NXOS6 и NXOS7:
```
NXOS6# ping 172.16.0.7
PING 172.16.0.7 (172.16.0.7): 56 data bytes
64 bytes from 172.16.0.7: icmp_seq=0 ttl=251 time=54.227 ms
64 bytes from 172.16.0.7: icmp_seq=1 ttl=251 time=26.639 ms
64 bytes from 172.16.0.7: icmp_seq=2 ttl=251 time=31.452 ms
64 bytes from 172.16.0.7: icmp_seq=3 ttl=251 time=38.935 ms
64 bytes from 172.16.0.7: icmp_seq=4 ttl=251 time=36.805 ms

--- 172.16.0.7 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 26.639/37.611/54.227 ms



NXOS7# traceroute6  2A03::6
traceroute to 2a03::6 (2a03::6), 30 hops max, 16 byte packets
 1  2a03::3  77.629 ms  12.94 ms  17.747 ms
 2  2a03::8  21.434 ms  22.204 ms  32.752 ms
 3  2a03::1  67.977 ms  24.032 ms  21.12 ms
 4  2a03::6  88.429 ms  38.549 ms  31.941 ms
```

