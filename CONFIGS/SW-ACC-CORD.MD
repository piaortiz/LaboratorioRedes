SW-ACC-CORDO#show running-config 
Building configuration...

Current configuration : 2139 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname SW-ACC-CORDO
!
!
!
no ip domain-lookup
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
 description PC2-VLAN10
 switchport access vlan 10
 switchport mode access
 spanning-tree portfast
!
interface FastEthernet0/2
 description FILE_SERVER_INTERNAL_VLAN20
 switchport access vlan 20
 ip access-group FTP_BLOCK in
 switchport mode access
 spanning-tree portfast
!
interface FastEthernet0/3
!
interface FastEthernet0/4
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
!
interface FastEthernet0/8
!
interface FastEthernet0/9
!
interface FastEthernet0/10
!
interface FastEthernet0/11
!
interface FastEthernet0/12
!
interface FastEthernet0/13
!
interface FastEthernet0/14
!
interface FastEthernet0/15
!
interface FastEthernet0/16
!
interface FastEthernet0/17
!
interface FastEthernet0/18
!
interface FastEthernet0/19
!
interface FastEthernet0/20
!
interface FastEthernet0/21
!
interface FastEthernet0/22
!
interface FastEthernet0/23
!
interface FastEthernet0/24
!
interface GigabitEthernet0/1
!
interface GigabitEthernet0/2
 description Trunk_to_DIS-CORD
 switchport trunk allowed vlan 10,20
 switchport mode trunk
!
interface Vlan1
 no ip address
 shutdown
!
banner motd ^C
*************************************************
*     ACCESO NO AUTORIZADO PROHIBIDO           *
*     Switch ACC-CORDO - Sitio Cordoba         *
*     Toda actividad es monitoreada            *
*************************************************
^C
!
!
!
ip access-list extended FTP_BLOCK
 deny tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.20.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.44.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.55.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.70.0 0.0.0.255 host 192.168.20.10 eq ftp
 permit ip any any
line con 0
 logging synchronous
 exec-timeout 30 0
!
line vty 0 4
 exec-timeout 30 0
 logging synchronous
 no login
line vty 5 15
 no login
!
!
!
!
end


SW-ACC-CORDO# 