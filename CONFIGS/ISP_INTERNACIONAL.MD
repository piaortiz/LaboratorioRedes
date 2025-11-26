ISP_INTERNACIONAL#show running-config 
Building configuration...

Current configuration : 1617 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname ISP_INTERNACIONAL
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
license udi pid CISCO2911/K9 sn FTX1524US4O-
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
 description Hacia_ISP_LOCAL_G0/0
 ip address 164.25.0.1 255.255.255.248
 ip nat outside
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/1.100
 description Hacia_SW_MS_CORE_VLAN100_DNS
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.248
 ip nat inside
!
interface GigabitEthernet0/1.101
 description Hacia_SW_MS_CORE_VLAN101_WEB
 encapsulation dot1Q 101
 ip address 192.168.100.14 255.255.255.248
 ip nat inside
!
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
ip nat inside source static 192.168.100.2 1.1.1.1 
ip nat inside source static 192.168.100.9 45.162.20.10 
ip classless
ip route 192.168.30.0 255.255.255.0 164.25.0.2 
ip route 42.25.25.0 255.255.255.248 164.25.0.2 
ip route 43.26.26.0 255.255.255.248 164.25.0.2 
ip route 192.168.10.0 255.255.255.0 164.25.0.2 
ip route 192.168.20.0 255.255.255.0 164.25.0.2 
ip route 172.20.10.0 255.255.255.248 164.25.0.2 
ip route 192.168.44.0 255.255.255.0 164.25.0.2 
ip route 192.168.55.0 255.255.255.0 164.25.0.2 
ip route 192.168.70.0 255.255.255.0 164.25.0.2 
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


ISP_INTERNACIONAL# 