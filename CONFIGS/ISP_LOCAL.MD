ISP_LOCAL#show running-config 
Building configuration...

Current configuration : 1738 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname ISP_LOCAL
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX15249X1X-
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
 description Internet_uplink
 ip address 164.25.0.2 255.255.255.248
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 description WAN1_from_BSAS
 ip address 42.25.25.2 255.255.255.248
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 description WAN2_from_BSAS
 ip address 43.26.26.2 255.255.255.248
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
ip route 192.168.30.0 255.255.255.0 42.25.25.1 
ip route 0.0.0.0 0.0.0.0 164.25.0.1 
ip route 192.168.10.0 255.255.255.0 42.25.25.1 
ip route 192.168.20.0 255.255.255.0 42.25.25.1 
ip route 172.20.10.0 255.255.255.248 42.25.25.1 
ip route 192.168.100.0 255.255.255.248 164.25.0.1 
ip route 192.168.100.8 255.255.255.248 164.25.0.1 
ip route 192.168.30.0 255.255.255.0 43.26.26.1 5
ip route 192.168.10.0 255.255.255.0 43.26.26.1 5
ip route 192.168.20.0 255.255.255.0 43.26.26.1 5
ip route 172.20.10.0 255.255.255.248 43.26.26.1 5
ip route 192.168.44.0 255.255.255.0 42.25.25.1 
ip route 192.168.55.0 255.255.255.0 42.25.25.1 
ip route 192.168.70.0 255.255.255.0 42.25.25.1 
ip route 192.168.44.0 255.255.255.0 43.26.26.1 5
ip route 192.168.55.0 255.255.255.0 43.26.26.1 5
ip route 192.168.70.0 255.255.255.0 43.26.26.1 5
ip route 45.162.20.10 255.255.255.255 164.25.0.1 
ip route 1.1.1.1 255.255.255.255 164.25.0.1 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end


ISP_LOCAL#  