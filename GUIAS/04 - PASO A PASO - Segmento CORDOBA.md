# Guía 04 · Segmento CÓRDOBA - PASO A PASO
**Fecha:** 19/11/2025  
**Prerrequisito:** Ninguno (se puede realizar en paralelo con otras guías)

---

## 🎯 OBJETIVO

Configurar el segmento local de Córdoba con:
- 2 VLANs locales (VLAN 10 y VLAN 20)
- Switch de acceso (ACC-CORDO) conectado a switch de distribución (DIS-CORD)
- Router CÓRDOBA con router-on-a-stick para las VLANs
- Conectividad local verificada antes de integrar con OSPF

> ⚠️ **NOTA SOBRE GUÍA 03**: Esta guía (04) configura las VLANs locales en **Gig0/2** del router CÓRDOBA. Si ya completaste la Guía 03, es posible que las VLANs estén configuradas en **Gig0/1**. En ese caso, sigue la configuración de la Guía 03 o ajusta según tu cableado físico. Lo importante es que las subinterfaces .10 y .20 estén en la interfaz correcta según tu topología.

---

## 📋 INFORMACIÓN DE LA TOPOLOGÍA

### Dispositivos
| Dispositivo | Tipo | Función |
|-------------|------|---------|
| **CÓRDOBA** | Router 2911 | Gateway para VLANs 10 y 20, routing inter-VLAN |
| **DIS-CORD** | Switch 2960 | Switch de distribución/core local |
| **ACC-CORDO** | Switch 2960 | Switch de acceso para usuarios y servidores |
| **PC2 - VLAN 10** | PC | Estación de trabajo en VLAN 10 |
| **FILE SERVER INTERNAL VLAN 20** | Server | Servidor de archivos en VLAN 20 |

### VLANs y Redes
| VLAN | Nombre | Red | Gateway | Rango Útil |
|------|--------|-----|---------|------------|
| 10 | USUARIOS_CORDOBA | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.2 - 192.168.10.254 |
| 20 | SERVIDORES_CORDOBA | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.2 - 192.168.20.254 |

### Cableado Físico
| Desde | Puerto | Hacia | Puerto | Tipo | VLAN/Modo |
|-------|--------|-------|--------|------|-----------|
| Router CÓRDOBA | **Gig0/2** | DIS-CORD | Gig0/1 | Trunk | 802.1Q (VLANs 10, 20) |
| DIS-CORD | Gig0/2 | ACC-CORDO | **Gig0/2** | Trunk | 802.1Q (VLANs 10, 20) |
| ACC-CORDO | **Fa0/1** | PC2 - VLAN 10 | NIC | Access | VLAN 10 |
| ACC-CORDO | **Fa0/2** | FILE SERVER | NIC | Access | VLAN 20 |

> ⚠️ **CRÍTICO - Router CÓRDOBA**: Las VLANs locales (10 y 20) se configuran en **Gig0/2** con subinterfaces. NO usar Gig0/1 para las VLANs locales. El router tiene otras interfaces (Gig0/0/0, etc.) para enlaces P2P a otros routers.

### IPs de Dispositivos Finales
| Dispositivo | IP | Máscara | Gateway | DNS |
|-------------|----|---------|---------|----|
| PC2 - VLAN 10 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 1.1.1.1 |
| FILE SERVER INTERNAL | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | 1.1.1.1 |

---

## 📝 PASO 1: SWITCH DE ACCESO (ACC-CORDO)

### 1A. Configuración Básica
```cisco
enable
configure terminal

hostname SW-ACC-CORDO
no ip domain-lookup

! Banner de seguridad
banner motd #
*************************************************
*     ACCESO NO AUTORIZADO PROHIBIDO           *
*     Switch ACC-CORDO - Sitio Cordoba         *
*     Toda actividad es monitoreada            *
*************************************************
#

! Configurar contraseña enable
enable secret cisco123
line console 0
 password cisco
 login
 logging synchronous
 exec-timeout 30 0
exit

line vty 0 4
 password cisco
 login
 logging synchronous
 exec-timeout 30 0
exit
exit
```

### 1B. Crear VLANs
```cisco
configure terminal

! Crear VLAN 10
vlan 10
 name USUARIOS_CORDOBA
exit

! Crear VLAN 20
vlan 20
 name SERVIDORES_CORDOBA
exit
exit
```

### 1C. Configurar Puertos de Acceso
```cisco
configure terminal

! Puerto hacia PC2 - VLAN 10
interface fa0/1
 description PC2-VLAN10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
exit

! Puerto hacia FILE SERVER - VLAN 20
interface fa0/2
 description FILE_SERVER_INTERNAL_VLAN20
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
exit
exit
```

### 1D. Configurar Puerto Trunk hacia DIS-CORD
```cisco
configure terminal

interface gig0/2
 description Trunk_to_DIS-CORD
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit
exit
write memory
```

### Verificación ACC-CORDO
```cisco
show vlan brief
show interfaces trunk
show interfaces status
```

**✅ Esperado:**
- VLANs 10 y 20 creadas
- Fa0/1 (PC): access VLAN 10, estado **connected**
- Fa0/2 (Server): access VLAN 20, estado **connected**
- Gig0/2: trunk con VLANs 10,20 permitidas

---

## 📝 PASO 2: SWITCH DE DISTRIBUCIÓN (DIS-CORD)

### 2A. Configuración Básica
```cisco
enable
configure terminal

hostname DIS-CORD
no ip domain-lookup

banner motd #
*************************************************
*     ACCESO NO AUTORIZADO PROHIBIDO           *
*     Switch DIS-CORD - Sitio Cordoba Core     *
*     Toda actividad es monitoreada            *
*************************************************
#

enable secret cisco123
line console 0
 password cisco
 login
 logging synchronous
 exec-timeout 30 0
exit

line vty 0 4
 password cisco
 login
 logging synchronous
 exec-timeout 30 0
exit
exit
```

### 2B. Crear VLANs
```cisco
configure terminal

vlan 10
 name USUARIOS_CORDOBA
exit

vlan 20
 name SERVIDORES_CORDOBA
exit
exit
```

### 2C. Configurar Trunk hacia Router
```cisco
configure terminal

interface gig0/1
 description Trunk_to_Router_CORDOBA_Gig0/2
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit
exit
```

### 2D. Configurar Trunk hacia ACC-CORDO
```cisco
configure terminal

interface gig0/2
 description Trunk_to_ACC-CORDO
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit
exit
```

### 2E. Configurar STP Root Bridge
```cisco
configure terminal

! Establecer este switch como root bridge para VLANs 10 y 20
spanning-tree vlan 10 priority 4096
spanning-tree vlan 20 priority 4096
exit
write memory
```

### Verificación DIS-CORD
```cisco
show vlan brief
show interfaces trunk
show spanning-tree vlan 10
show spanning-tree vlan 20
```

**✅ Esperado:**
- VLANs 10 y 20 creadas
- Gig0/1 y Gig0/2 en modo **trunk**
- VLANs 10,20 permitidas en ambos trunks
- Este switch es **root** para VLANs 10 y 20 (priority 4096)

---

## 📝 PASO 3: ROUTER CÓRDOBA

> ⚠️ **NOTA CRÍTICA**: En este paso configuraremos las VLANs locales en la interfaz **Gig0/2** del router. NO confundir con enlaces P2P entre routers que se configuran en otras interfaces (Gig0/0/0, etc.).

### 3A. Configuración Básica
```cisco
enable
configure terminal

hostname CORDOBA
no ip domain-lookup

banner motd #
*************************************************
*     ACCESO NO AUTORIZADO PROHIBIDO           *
*     Router CORDOBA - Sitio Cordoba           *
*     Toda actividad es monitoreada            *
*************************************************
#

enable secret cisco123
line console 0
 password cisco
 login
 logging synchronous
 exec-timeout 30 0
exit

line vty 0 4
 password cisco
 login
 logging synchronous
 exec-timeout 30 0
exit
exit
```

### 3B. Configurar Interfaz Física (sin IP)
```cisco
configure terminal

! Interfaz física hacia DIS-CORD (NO lleva IP, NO configurar OSPF aquí)
interface gig0/2
 description Trunk_to_DIS-CORD
 no ip address
 no shutdown
exit
exit
```

### 3C. Configurar Subinterfaces para VLANs (Router-on-a-Stick)
```cisco
configure terminal

! Subinterfaz para VLAN 10
interface gig0/2.10
 description Gateway_VLAN10_USUARIOS
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

! Subinterfaz para VLAN 20
interface gig0/2.20
 description Gateway_VLAN20_SERVIDORES
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit
exit
write memory
```

> ⚠️ **IMPORTANTE**: NO configurar comandos OSPF (`ip ospf network point-to-point`, `ip ospf cost`, etc.) en la interfaz física Gig0/2, ya que esta interfaz es un TRUNK para VLANs locales, NO un enlace P2P entre routers.

### Verificación Router CÓRDOBA
```cisco
show ip interface brief
show interfaces gig0/2
show interfaces gig0/2.10
show interfaces gig0/2.20
show running-config interface gig0/2
```

**✅ Esperado:**
- Gig0/2: **up/up** (sin IP configurada)
- Gig0/2.10: **up/up** con IP 192.168.10.1
- Gig0/2.20: **up/up** con IP 192.168.20.1
- Encapsulación dot1Q visible en cada subinterfaz

---

## 📝 PASO 4: CONFIGURAR DISPOSITIVOS FINALES

### 4A. PC2 - VLAN 10
En Packet Tracer, hacer clic en **PC2 - VLAN 10** → **Desktop** → **IP Configuration**:

```
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
DNS Server: 1.1.1.1
```

### 4B. FILE SERVER INTERNAL - VLAN 20
En Packet Tracer, hacer clic en **FILE SERVER INTERNAL** → **Desktop** → **IP Configuration**:

```
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
DNS Server: 1.1.1.1
```

Opcional - Activar servicios:
- **Services** → **HTTP**: ON
- **Services** → **FTP**: ON (usuario: `cisco`, password: `cisco`)

---

## 🔍 PASO 5: VERIFICACIONES CRÍTICAS

### 5A. Verificar Conectividad Local

**Desde PC2 - VLAN 10:**
```
ping 192.168.10.1     (Gateway VLAN 10)
ping 192.168.20.1     (Gateway VLAN 20)
ping 192.168.20.10    (FILE SERVER)
```

**Desde FILE SERVER:**
```
ping 192.168.20.1     (Gateway VLAN 20)
ping 192.168.10.1     (Gateway VLAN 10)
ping 192.168.10.10    (PC2)
```

**✅ Todos los pings deben responder exitosamente**

### 5B. Verificar Tablas MAC en Switches

**En ACC-CORDO:**
```cisco
show mac address-table
```
- Debe mostrar MACs de PC2 en VLAN 10
- Debe mostrar MAC de FILE SERVER en VLAN 20

**En DIS-CORD:**
```cisco
show mac address-table
```
- Debe mostrar MACs aprendidas en VLANs 10 y 20

### 5C. Verificar Tabla ARP en Router

**En Router CÓRDOBA:**
```cisco
show ip arp
```

**✅ Esperado:**
```
Internet  192.168.10.10   -   <MAC_PC2>       ARPA   GigabitEthernet0/2.10
Internet  192.168.20.10   -   <MAC_SERVER>    ARPA   GigabitEthernet0/2.20
```

> 📝 **Nota**: Si seguiste la Guía 03, las interfaces podrían aparecer como Gig0/1.10 y Gig0/1.20. Ambas configuraciones son válidas, depende de tu cableado físico.

### 5D. Verificar STP

**En DIS-CORD:**
```cisco
show spanning-tree vlan 10
show spanning-tree vlan 20
```

**✅ Esperado:**
- DIS-CORD debe aparecer como **root** (This bridge is the root)
- Priority: 4096

---

## 🚨 TROUBLESHOOTING

### Problema: PC2 no hace ping al gateway
**Posibles causas:**
1. Puerto Fa0/1 en ACC-CORDO no está en VLAN 10
2. Trunk entre ACC-CORDO y DIS-CORD no permite VLAN 10
3. Trunk entre DIS-CORD y Router no permite VLAN 10
4. Subinterfaz Gig0/2.10 en router está down

**Diagnóstico:**
```cisco
! En ACC-CORDO
show vlan brief
show interfaces fa0/1 switchport

! En DIS-CORD
show interfaces trunk

! En Router CÓRDOBA
show ip interface brief
show interfaces gig0/2
```

### Problema: PC2 hace ping a gateway pero no a FILE SERVER
**Causa:** Routing inter-VLAN no funciona

**Verificación:**
```cisco
! En Router CÓRDOBA
show ip route
```

Debe mostrar:
```
C       192.168.10.0/24 is directly connected, GigabitEthernet0/2.10
C       192.168.20.0/24 is directly connected, GigabitEthernet0/2.20
```

### Problema: Trunk no funciona
**Verificación de encapsulación:**
```cisco
! En Router
show interfaces gig0/2.10
```

Debe mostrar: `Encapsulation 802.1Q Virtual LAN, Vlan ID 10`

Si no aparece, reconfigurar:
```cisco
configure terminal
interface gig0/2.10
 no encapsulation dot1Q 10
 encapsulation dot1Q 10
exit
```

### Problema: Configuración duplicada en Gig0/1 y Gig0/2
**Síntoma:** Las VLANs están configuradas en Gig0/1 pero el cable físico está en Gig0/2

**Solución:**
```cisco
configure terminal

! Limpiar Gig0/2 si tiene comandos OSPF P2P incorrectos
interface gig0/2
 description Trunk_to_DIS-CORD
 no ip ospf network point-to-point
 no ip ospf cost
 no ip ospf priority
 no shutdown
exit

! Mover IPs de Gig0/1 a Gig0/2 si es necesario
! Primero eliminar subinterfaces de Gig0/1
no interface gig0/1.10
no interface gig0/1.20

interface gig0/1
 no ip address
 shutdown
exit

! Configurar correctamente en Gig0/2
interface gig0/2.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

interface gig0/2.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

write memory
```

---

## ✅ CHECKLIST FINAL

- [ ] Switch ACC-CORDO: VLANs 10/20 creadas, puertos access configurados
- [ ] Switch DIS-CORD: VLANs 10/20 creadas, trunks operativos, root bridge configurado
- [ ] Router CÓRDOBA: Subinterfaces .10 y .20 up/up con IPs correctas
- [ ] PC2: IP configurada, ping a gateway exitoso
- [ ] FILE SERVER: IP configurada, servicios HTTP/FTP activos
- [ ] Ping PC2 → FILE SERVER: exitoso (routing inter-VLAN funciona)
- [ ] Ping FILE SERVER → PC2: exitoso
- [ ] Tabla ARP del router muestra ambos dispositivos
- [ ] STP: DIS-CORD es root para VLANs 10 y 20
- [ ] Configuraciones guardadas: `write memory` en todos los dispositivos

---

## 📄 COMANDOS DE DOCUMENTACIÓN

Guardar las siguientes salidas para el informe:

**En ACC-CORDO:**
```cisco
show running-config
show vlan brief
show interfaces trunk
show mac address-table
```

**En DIS-CORD:**
```cisco
show running-config
show vlan brief
show interfaces trunk
show spanning-tree vlan 10
show spanning-tree vlan 20
```

**En Router CÓRDOBA:**
```cisco
show running-config
show ip interface brief
show ip route
show ip arp
```

---

## 🔗 PRÓXIMOS PASOS

Una vez completada esta guía, el segmento de Córdoba estará operativo localmente. Los siguientes pasos son:

1. **Configurar enlaces P2P** hacia Buenos Aires y Mendoza (Guía 03)
2. **Activar OSPF** en el router CÓRDOBA para propagar rutas (Guía 03)
3. **Configurar red de backup** VLAN 1000 (Guía 03)

---

**¡Guía completada! 🎉**  
El segmento de Córdoba está listo para integrarse con el resto de la red corporativa.
