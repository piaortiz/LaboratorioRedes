CORDOBA#
CORDOBA#
CORDOBA#
CORDOBA#
CORDOBA#
CORDOBA#
CORDOBA#
CORDOBA#
CORDOBA#
CORDOBA#show ru
CORDOBA#show running-config 
Building configuration...

Current configuration : 2443 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname CORDOBA
!
!
!
!
!
!
!
!
no ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX1524GZK6-
!
!
!
!
!
!
!
!
!
!
!
no ip domain-lookup
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0
 description Trunk_to_SW_OSPF_BACKUP
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/0.1000
 description BACKUP_OSPF_VLAN1000
 encapsulation dot1Q 1000
 ip address 172.20.10.2 255.255.255.248
 ip ospf cost 50
 ip ospf priority 50
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface GigabitEthernet0/2
 description Trunk_to_DIS-CORD
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/2.10
 description Gateway_VLAN10_USUARIOS
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/2.20
 description Gateway_VLAN20_SERVIDORES
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip access-group FTP_BLOCK out
!
interface GigabitEthernet0/0/0
 description P2P_to_BSAS
 ip address 10.10.1.10 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
 ip access-group FTP_BLOCK_P2P in
!
interface GigabitEthernet0/1/0
 description P2P_to_MENDOZA
 ip address 10.10.1.17 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 10
 ip ospf priority 1
!
interface Vlan1
 no ip address
 shutdown
!
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 passive-interface GigabitEthernet0/2.10
 passive-interface GigabitEthernet0/2.20
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 10.10.1.8 0.0.0.3 area 0
 network 10.10.1.16 0.0.0.3 area 0
 network 172.20.10.0 0.0.0.7 area 0
!
ip classless
ip route 0.0.0.0 0.0.0.0 10.10.1.9 
!
ip flow-export version 9
!
!
ip access-list extended FTP_BLOCK_P2P
 deny tcp 192.168.44.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.55.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.70.0 0.0.0.255 host 192.168.20.10 eq ftp
 permit ip any any
ip access-list extended FTP_BLOCK
 permit ip host 192.168.30.10 host 192.168.20.10
 deny ip any host 192.168.20.10
 permit ip any any
!
!
!
!
!
line con 0
 exec-timeout 30 0
 logging synchronous
!
line aux 0
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
end


CORDOBA#  