# Реализация DHCPv4/6

## Цель:

* Настройка и проверка двух серверов DHCPv4.
* Настройка и проверка DHCP-ретрансляции.
* Проверка назначения адреса SLAAC.
* Настройка и проверка DHCPv6.

## Описание/Пошаговая инструкция выполнения домашнего задания:

* Ваша задача заключается в настройке маршрутизатора R2 для назначения IPv4-адресов в двух разных подсетях.
* Настроить маршрутизатор R2 для назначения адресов IPv6 в двух разных подсетях, подключенных к маршрутизатору R1.
* Подробное описание задания в методичках в материалах к занятию.
* Готовые конфигурации необходимо оформить на github с описанием проделанной работы, используя markdown.

---




#### Часть 1. Создание сети и настройка основных параметров устройства
<details>
<summary>Шаг 1. Создайте сеть согласно топологии.</summary>

![Topology](img/topology.png)

##### Таблица адресации.
|Устройство|Интерфейс|IPv6-адрес|
|   :---:|:---|:---|
|R1|G0/0/0|2001:db8:acad:2::1/64|
|   |   |fe80::1|
|  |G0/0/1|2001:db8:acad:1::1/64|
|   |   |fe80::1|
|R2|G0/0/0|2001:db8:acad:2::2/64|
|   |   |fe80::2|
|  |G0/0/1|2001:db8:acad:3::1/64|
|   |   |fe80::1|
|PC-A|NIC|DHCP|
|PC-B|NIC|DHCP|  

</details>

<details>
<summary>Шаг 2. Настройте базовые параметры каждого коммутатора. </summary>

 **S1 S2**
```Console
enable
conf t
hostname S1 // S2
no ip domain-lookup
enable secret class
line console 0
password cisco
login
logging synchronous
exit
line vty 0 4
password cisco
login
exit
service password-encryption

banner motd #  S1 - Switch #
// S2 banner motd #  S2 - Switch #

interface range fa0/1-4, fa0/7-24, gi0/1-2
// S2  interface range fa0/1-4, fa0/6-17, fa0/19-24, gi0/1-2  
shutdown
exit
exit

copy running-config startup-config
```
</details>

<details>
<summary>Шаг 3. Произведите базовую настройку маршрутизаторов.</summary>

 **R1 R2**

```Console
enable  
conf t
hostname R1  // R2
no ip domain-lookup  
enable secret class  
line console 0  
password cisco  
login  
logging synchronous  
exit  
line vty 0 4  
password cisco  
login  
exit  
service password-encryption  
banner motd #  R1 - Router # // R2 banner motd #  R2 - Router #   
ipv6 unicast-routing  
exit  
copy running-config startup-config  

```

</details>

<details>
<summary>Шаг 4. Настройка интерфейсов и маршрутизации для обоих маршрутизаторов.</summary>

**R1**
```Console
en 
conf t
int gigabitEthernet 0/0/0  
ipv6 address 2001:db8:acad:2::1/64  
ipv6 address fe80::1 link-local
no sh 
ex
int gigabitEthernet 0/0/1  
ipv6 address 2001:db8:acad:1::1/64  
ipv6 address fe80::1 link-local  
no sh 
ex
ipv6 route ::/0 2001:db8:acad:2::2
ex
copy running-config startup-config  
```

**R2**
```Console
en
conf t
int gigabitEthernet 0/0/0  
ipv6 address 2001:db8:acad:2::2/64  
ipv6 address fe80::2 link-local  
no sh
ex
int gigabitEthernet 0/0/1  
ipv6 address 2001:db8:acad:3::1/64  
ipv6 address fe80::1 link-local  
no sh 
ex
ipv6 route ::/0 2001:db8:acad:2::1
ex
copy running-config startup-config  
```

Убедитесь, что маршрутизация работает с помощью пинга адреса G0/0/1 R2 из R1
```Console
R2#ping 2001:db8:acad:1::1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:1::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

```Console
R1#ping 2001:db8:acad:3::1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

</details>



#### Часть 2. Проверка назначения адреса SLAAC от R1
<details>
<summary>Часть 2. Проверка назначения адреса SLAAC от R1</summary>

**PS-A1**
```Console
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::209:7CFF:FE54:9403
   IPv6 Address....................: 2001:DB8:ACAD:1:209:7CFF:FE54:9403
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

`Откуда взялась часть адреса с идентификатором хоста?`  
Часть адреса с идентификатором хоста сгенерировалась методом SLAAC

</details>

#### Часть 3. Настройка и проверка сервера DHCPv6 без гражданства на R1

<details>
<summary>Шаг 1. Более подробно изучите конфигурацию PC-A.</summary>


**PC-A**
```Console
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0009.7C54.9403
   Link-local IPv6 Address.........: FE80::209:7CFF:FE54:9403
   IPv6 Address....................: 2001:DB8:ACAD:1:209:7CFF:FE54:9403
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-A2-6A-3D-21-00-09-7C-54-94-03
   DNS Servers.....................: ::
                                     0.0.0.0

```


</details>

<details>
<summary>Шаг 2. Настройте R1 для предоставления DHCPv6 без состояния для PC-A.</summary>

**R1**

```Console
en
conf t
ipv6 dhcp pool R1-STATELESS
dns-server 2001:db8:acad::254
domain-name STATELESS.com

ex

interface g0/0/1
ipv6 nd other-config-flag
ipv6 dhcp server R1-STATELESS

ex
ex


copy running-config startup-config  
```

Перезапустите PC-A.e.  
Проверьте выводipconfig /allи обратите внимание на изменения.

**PC-A**

```Console
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 0009.7C54.9403
   Link-local IPv6 Address.........: FE80::209:7CFF:FE54:9403
   IPv6 Address....................: 2001:DB8:ACAD:1:209:7CFF:FE54:9403
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 1113926435
   DHCPv6 Client DUID..............: 00-01-00-01-A2-6A-3D-21-00-09-7C-54-94-03
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0
```

Тестирование подключения с помощью пинга IP-адреса интерфейса G0/1 R2.
**PC-A**
```Console
C:\>ping 2001:db8:acad:3::1

Pinging 2001:db8:acad:3::1 with 32 bytes of data:

Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254

Ping statistics for 2001:DB8:ACAD:3::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

</details>

#### Часть 4. Настройка и проверка состояния DHCPv6 сервера на R1

<details>
<summary>В части 4 настраивается R1 для ответа на запросы DHCPv6 от локальной сети на R2</summary>

**R1**

```Console
en
conf t

ipv6 dhcp pool R2-STATEFUL
address prefix 2001:db8:acad:3:aaa::/80
dns-server 2001:db8:acad::254
domain-name STATEFUL.com

int g0/0/0
ipv6 dhcp server R2-STATEFUL

```

</details>

#### Часть 5. Настройка и проверка DHCPv6 Relay на R2

<details>
<summary>Шаг 1. Включите PC-B и проверьте адрес SLAAC, который он генерирует.</summary>

```Console
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0002.4A67.7897
   Link-local IPv6 Address.........: FE80::202:4AFF:FE67:7897
   IPv6 Address....................: 2001:DB8:ACAD:3:202:4AFF:FE67:7897
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-70-93-CA-56-00-02-4A-67-78-97
   DNS Servers.....................: ::
                                     0.0.0.0
```

</details>

<details>
<summary>Шаг 2. Настройте R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1.</summary>

**R2**

```Console
en
conf t

int g0/0/1
ipv6 nd managed-config-flag
ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0
                        ^
% Invalid input detected at '^' marker.

```
в PCT не работает

</details>

<details>
<summary>Шаг 3. Попытка получить адрес IPv6 из DHCPv6 на PC-B.</summary>

```Console
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0002.4A67.7897
   Link-local IPv6 Address.........: FE80::202:4AFF:FE67:7897
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 455044735
   DHCPv6 Client DUID..............: 00-01-00-01-70-93-CA-56-00-02-4A-67-78-97
   DNS Servers.....................: ::
                                     0.0.0.0

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Physical Address................: 0007.EC80.663D
   Link-local IPv6 Address.........: ::
```

Чуда конечно не произошло, ну это и ожидалось. Ретрансляции DHCPv6 не работает - так как команда не применилась

```Console

C:\>ping 2001:db8:acad:1::1

Pinging 2001:db8:acad:1::1 with 32 bytes of data:

Reply from FE80::1: Destination host unreachable.
Reply from FE80::1: Destination host unreachable.
Reply from FE80::1: Destination host unreachable.
Reply from FE80::1: Destination host unreachable.

Ping statistics for 2001:DB8:ACAD:1::1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```
Ну вот и все.

</details>

---