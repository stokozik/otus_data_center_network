## **Underlay. IS-IS**
## **Цель: Настроить IS-IS для Underlay сети**
## **Описание/Пошаговая инструкция:**
1. Настроить IP адресацию на физических и Loopback интерфейсах всех Spine и Leaf
2. Настроить на всех устройствах IS-IS процессы, запустить IS-IS на всех линках между Spine и Leaf, а также на Loopback интерфейсах
3. Настроить IS-IS-аутентификацию
4. Настроить BFD
5. Убедиться в наличии одинаковой базы LSDB на всех устройствах, проверить IP связность между Loopback интерфейсами

### **Схема сети**
![alt text](image.png)

## **Выполнение работы:**
1. Настраиваем адресацию интерфейсов устройств согласно таблицы:
![alt text](image-1.png)
2. Создаем IS-IS процесс на каждом коммутаторе, указываем router-id равный ip адресу loopback интерфейса, указываем на каждом маршрутизаторе NET вида 49.xxxx.nnnn.nnnn.nnnn.00, где xxxx - номер Area (0001) nnnn.nnnn.nnnn - router id (адрес loopback0 дополненный нулями), глобально указываем тип маршрутизатора L1, настраиваем address family ipv4 unicast, включаем bfd:
```
Spine 1:

router isis Underlay
   net 49.0001.0100.0000.1001.00
   router-id ipv4 10.0.1.1
   is-type level-1
   address-family ipv4 unicast
      bfd all-interfaces
```
3. Производим настройку IS-IS на интерфейсах, указываем тип сети point-to-point, настраиваем аутентификацию. Включаем loopback0 интерфейсы в процесс IS-IS:
```
Spine 1:

interface Ethernet1
   description -=p2p_Leaf1=-
   no switchport
   ip address 10.2.1.0/31
   isis enable Underlay
   isis network point-to-point
   isis authentication mode md5
   isis authentication key 7 MQPLKfteGAtNwmAC5q9/qA==

interface Loopback0
   ip address 10.0.1.1/32
   isis enable Underlay
   isis passive
```
4. Проверяем что IS-IS соседство между Spine и Leaf установлено, базы данных LSDB одинаковы на всех устройствах:

```
Spine1#show isis database 

IS-IS Instance: Underlay VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Spine1.00-00                 64  62746  1129    146 L1 <>
    Spine2.00-00                 24  53325  1125    146 L1 <>
    Leaf3.00-00                  22  13813  1125    121 L1 <>
    Leaf1.00-00                  48  42657  1124    121 L1 <>
    Leaf2.00-00                  51  15596  1122    121 L1 <>

Leaf3#show isis database 

IS-IS Instance: Underlay VRF: default
  IS-IS Level 1 Link State Database
    LSPID                   Seq Num  Cksum  Life Length IS Flags
    Spine1.00-00                 64  62746  1071    146 L1 <>
    Spine2.00-00                 25  52814  1148    146 L1 <>
    Leaf3.00-00                  22  13813  1068    121 L1 <>
    Leaf1.00-00                  48  42657  1067    121 L1 <>
    Leaf2.00-00                  51  15596  1067    121 L1 <>

Spine1#show isis neighbors 
 
Instance  VRF      System Id        Type Interface          SNPA              State Hold time 
  Circuit Id          
Underlay  default  Leaf3            L1   Ethernet3          P2P               UP    27        
  0D                  
Underlay  default  Leaf1            L1   Ethernet1          P2P               UP    22        
  0F                  
Underlay  default  Leaf2            L1   Ethernet2          P2P               UP    26        
  0F                  

Spine2#show isis neighbors 
 
Instance  VRF      System Id        Type Interface          SNPA              State Hold time 
  Circuit Id          
Underlay  default  Leaf3            L1   Ethernet3          P2P               UP    24        
  0E                  
Underlay  default  Leaf1            L1   Ethernet1          P2P               UP    24        
  10                  
Underlay  default  Leaf2            L1   Ethernet2          P2P               UP    22        
  10   
```
5. Проверяем связность между Loopback интерфейсами:
```
Leaf1#ping 10.0.2.2
PING 10.0.2.2 (10.0.2.2) 72(100) bytes of data.
80 bytes from 10.0.2.2: icmp_seq=1 ttl=63 time=42.5 ms
80 bytes from 10.0.2.2: icmp_seq=2 ttl=63 time=40.2 ms
80 bytes from 10.0.2.2: icmp_seq=3 ttl=63 time=60.9 ms
80 bytes from 10.0.2.2: icmp_seq=4 ttl=63 time=60.7 ms
80 bytes from 10.0.2.2: icmp_seq=5 ttl=63 time=22.3 ms

--- 10.0.2.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 84ms
rtt min/avg/max/mdev = 22.331/45.378/60.958/14.460 ms, pipe 4, ipg/ewma 21.119/43.600 ms

Leaf1#traceroute 10.0.3.2
traceroute to 10.0.3.2 (10.0.3.2), 30 hops max, 60 byte packets
 1  10.2.1.0 (10.2.1.0)  58.699 ms  61.044 ms  76.965 ms
 2  10.0.3.2 (10.0.3.2)  104.477 ms  111.767 ms  118.919 ms
```
6. Проверяем что bfd пиринг установлен:

```
Spine2#show bfd peers 
VRF name: default
-----------------
DstAddr       MyDisc    YourDisc  Interface/Transport    Type           LastUp 
--------- ----------- ----------- -------------------- ------- ----------------
10.2.2.1   615022180  2510497962        Ethernet1(13)  normal   12/08/24 00:21 
10.2.2.3  1349843520  2557947939        Ethernet2(14)  normal   12/08/24 00:21 
10.2.2.5  3604147996  3748466851        Ethernet3(15)  normal   12/08/24 00:21 

   LastDown            LastDiag    State
-------------- ------------------- -----
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up
         NA       No Diagnostic       Up

```
