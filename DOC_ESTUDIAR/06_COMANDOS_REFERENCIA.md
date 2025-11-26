# ⚡ COMANDOS DE REFERENCIA RÁPIDA

Esta guía contiene todos los comandos útiles para verificar, troubleshoot y demostrar tu red.

---

## 📖 Índice
1. [Comandos OSPF](#ospf)
2. [Comandos VLANs](#vlans)
3. [Comandos NAT](#nat)
4. [Comandos ACLs](#acls)
5. [Comandos STP](#stp)
6. [Comandos de Enrutamiento](#enrutamiento)
7. [Comandos de Interfaces](#interfaces)
8. [Comandos de Conectividad](#conectividad)
9. [Comandos de Troubleshooting](#troubleshooting)
10. [Comandos de Demostración](#demostracion)

---

## 🔷 1. Comandos OSPF {#ospf}

### **Verificación Básica**

```cisco
! Ver tabla de enrutamiento OSPF
show ip route ospf

! Ver vecinos OSPF
show ip ospf neighbor

! Ver información detallada de OSPF
show ip ospf

! Ver interfaces OSPF
show ip ospf interface

! Ver base de datos de estado de enlace
show ip ospf database
```

### **Información Detallada**

```cisco
! Ver información de una interfaz específica
show ip ospf interface GigabitEthernet0/0/0

! Ver detalles de un vecino específico
show ip ospf neighbor detail

! Ver estadísticas de OSPF
show ip ospf statistics

! Ver configuración de OSPF
show running-config | section router ospf
```

### **Troubleshooting OSPF**

```cisco
! Verificar que OSPF esté habilitado
show ip protocols

! Ver eventos de OSPF
show ip ospf events

! Limpiar proceso OSPF (reconvergencia forzada)
clear ip ospf process

! Debug OSPF (usar con precaución)
debug ip ospf adj
debug ip ospf events
undebug all  ! Desactivar debug
```

### **Qué Buscar**

**En `show ip ospf neighbor`:**
- Estado debe ser **FULL** (o 2WAY en DR/BDR)
- Dead Time debe estar contando hacia abajo
- Verificar Router ID del vecino

**En `show ip route ospf`:**
- Rutas aprendidas vía OSPF (código "O")
- Distancia administrativa [110/costo]
- Next hop correcto

---

## 🔷 2. Comandos VLANs {#vlans}

### **En Switches**

```cisco
! Ver VLANs configuradas
show vlan brief

! Ver detalles de una VLAN específica
show vlan id 10

! Ver interfaces trunk
show interfaces trunk

! Ver estado de una interfaz
show interfaces FastEthernet0/1 switchport

! Ver configuración de VLANs
show running-config | section vlan
```

### **En Routers (Subinterfaces)**

```cisco
! Ver subinterfaces configuradas
show ip interface brief

! Ver detalles de una subinterfaz
show interfaces GigabitEthernet0/1.30

! Ver encapsulación 802.1Q
show running-config interface GigabitEthernet0/1.30
```

### **Verificación de Trunking**

```cisco
! Ver estado de trunk
show interfaces GigabitEthernet0/1 trunk

! Ver VLANs permitidas en trunk
show interfaces trunk | include Allowed

! Ver VLAN nativa
show interfaces trunk | include Native
```

### **Qué Buscar**

**En `show vlan brief`:**
- VLAN existe y está activa
- Puertos asignados correctamente

**En `show interfaces trunk`:**
- Modo: trunk
- VLANs permitidas correctas
- VLAN nativa configurada

---

## 🔷 3. Comandos NAT {#nat}

### **Verificación de NAT**

```cisco
! Ver traducciones NAT activas
show ip nat translations

! Ver estadísticas de NAT
show ip nat statistics

! Ver configuración de NAT
show running-config | include nat
```

### **Troubleshooting NAT**

```cisco
! Limpiar traducciones NAT
clear ip nat translation *

! Ver interfaces NAT inside/outside
show ip interface brief | include NAT

! Debug NAT (usar con precaución)
debug ip nat
debug ip nat detailed
undebug all
```

### **Verificación de ACLs para NAT**

```cisco
! Ver ACLs usadas para NAT
show access-lists 10
show access-lists 11

! Ver coincidencias de ACL
show access-lists | include matches
```

### **Qué Buscar**

**En `show ip nat translations`:**
- Inside local: IP privada del dispositivo
- Inside global: IP pública asignada
- Outside local/global: Destino externo
- Puertos correctos (para PAT)

**En `show ip nat statistics`:**
- Total active translations
- Interfaces inside/outside correctas
- ACLs aplicadas

---

## 🔷 4. Comandos ACLs {#acls}

### **Verificación de ACLs**

```cisco
! Ver todas las ACLs
show access-lists

! Ver una ACL específica
show access-lists FTP_BLOCK

! Ver ACLs aplicadas a interfaces
show ip interface GigabitEthernet0/2.20 | include access list

! Ver configuración de ACLs
show running-config | section access-list
```

### **Estadísticas de ACLs**

```cisco
! Ver coincidencias (matches) de ACL
show access-lists FTP_BLOCK

! Ver ACLs con estadísticas detalladas
show ip access-lists
```

### **Troubleshooting ACLs**

```cisco
! Verificar ACL en interfaz
show running-config interface GigabitEthernet0/2.20

! Limpiar contadores de ACL
clear access-list counters FTP_BLOCK

! Ver orden de procesamiento
show access-lists FTP_BLOCK
```

### **Qué Buscar**

**En `show access-lists`:**
- Orden de las reglas (de arriba hacia abajo)
- Coincidencias (matches) en cada regla
- Regla que está bloqueando/permitiendo tráfico

**En `show ip interface`:**
- ACL aplicada en dirección correcta (in/out)
- Nombre de ACL correcto

---

## 🔷 5. Comandos STP {#stp}

### **Verificación de STP**

```cisco
! Ver resumen de STP
show spanning-tree

! Ver STP para una VLAN específica
show spanning-tree vlan 10

! Ver STP en una interfaz
show spanning-tree interface FastEthernet0/1

! Ver Root Bridge
show spanning-tree root

! Ver configuración de STP
show spanning-tree summary
```

### **Información Detallada**

```cisco
! Ver detalles de STP por VLAN
show spanning-tree vlan 10 detail

! Ver prioridades de puertos
show spanning-tree interface FastEthernet0/1 detail

! Ver Bridge ID
show spanning-tree bridge
```

### **Qué Buscar**

**En `show spanning-tree`:**
- Root Bridge ID (menor es Root)
- Root Port (puerto hacia Root)
- Designated Ports (reenvían tráfico)
- Blocked Ports (bloqueados para prevenir loops)

**Estados de Puerto:**
- **FWD** (Forwarding): Reenvía tráfico ✅
- **BLK** (Blocking): Bloqueado ⚠️
- **LRN** (Learning): Aprendiendo MACs
- **LIS** (Listening): Escuchando BPDUs

---

## 🔷 6. Comandos de Enrutamiento {#enrutamiento}

### **Tabla de Enrutamiento**

```cisco
! Ver tabla de enrutamiento completa
show ip route

! Ver solo rutas OSPF
show ip route ospf

! Ver solo rutas estáticas
show ip route static

! Ver solo rutas conectadas
show ip route connected

! Ver ruta a una red específica
show ip route 192.168.10.0
```

### **Protocolos de Enrutamiento**

```cisco
! Ver protocolos de enrutamiento activos
show ip protocols

! Ver métricas y distancias administrativas
show ip route | include \[
```

### **Rutas Específicas**

```cisco
! Ver ruta por defecto
show ip route 0.0.0.0

! Ver todas las rutas por defecto
show ip route | include 0.0.0.0
```

### **Qué Buscar**

**Códigos de Ruta:**
- **C**: Conectada directamente
- **S**: Estática
- **O**: OSPF
- **O IA**: OSPF inter-área
- **O E1/E2**: OSPF externa

**Formato:**
```
O    192.168.10.0/24 [110/20] via 10.10.1.10, 00:05:23, GigabitEthernet0/0/0
     ^                ^  ^      ^              ^          ^
     Código          AD Costo  Next hop       Tiempo     Interfaz
```

---

## 🔷 7. Comandos de Interfaces {#interfaces}

### **Estado de Interfaces**

```cisco
! Ver resumen de interfaces
show ip interface brief

! Ver detalles de una interfaz
show interfaces GigabitEthernet0/0/0

! Ver estadísticas de interfaz
show interfaces GigabitEthernet0/0/0 | include packets

! Ver errores en interfaces
show interfaces | include error
```

### **Configuración de Interfaces**

```cisco
! Ver configuración de una interfaz
show running-config interface GigabitEthernet0/0/0

! Ver todas las interfaces
show ip interface
```

### **Troubleshooting de Interfaces**

```cisco
! Ver estado de línea y protocolo
show interfaces status

! Ver contadores de interfaz
show interfaces counters

! Limpiar contadores
clear counters GigabitEthernet0/0/0
```

### **Qué Buscar**

**En `show ip interface brief`:**
- Status: **up** (físicamente activa)
- Protocol: **up** (protocolo de capa 2 activo)
- IP Address: Correcta

**Estados Comunes:**
- **up/up**: Interfaz funcionando ✅
- **up/down**: Problema de capa 2 ⚠️
- **down/down**: Interfaz apagada o cable desconectado ❌
- **administratively down**: Interfaz con `shutdown` ⚠️

---

## 🔷 8. Comandos de Conectividad {#conectividad}

### **Pruebas Básicas**

```cisco
! Ping básico
ping 192.168.10.5

! Ping extendido (más opciones)
ping
  Protocol [ip]: 
  Target IP address: 192.168.10.5
  Repeat count [5]: 10
  Datagram size [100]: 
  Timeout in seconds [2]: 
  Extended commands [n]: n
  Sweep range of sizes [n]: n

! Traceroute
traceroute 192.168.10.5

! Traceroute desde una IP específica
traceroute 192.168.10.5 source 192.168.30.1
```

### **Pruebas de Conectividad Avanzadas**

```cisco
! Telnet a un puerto específico (ej: FTP)
telnet 192.168.20.10 21

! Verificar resolución DNS
ping www.google.com

! Verificar conectividad con tamaño de paquete específico
ping 192.168.10.5 size 1500
```

### **Qué Buscar**

**En Ping:**
- **!** = Éxito ✅
- **.** = Timeout ⚠️
- **U** = Unreachable ❌

**En Traceroute:**
- Cada salto debe responder
- Verificar ruta tomada
- Identificar dónde falla la conectividad

---

## 🔷 9. Comandos de Troubleshooting {#troubleshooting}

### **Diagnóstico General**

```cisco
! Ver logs del sistema
show logging

! Ver versión de IOS
show version

! Ver uso de CPU
show processes cpu

! Ver uso de memoria
show memory

! Ver configuración actual
show running-config

! Ver configuración guardada
show startup-config
```

### **Verificación de Configuración**

```cisco
! Comparar running vs startup
show archive config differences

! Ver cambios recientes
show archive

! Ver sesiones activas
show users
```

### **Troubleshooting por Capa OSI**

**Capa 1 (Física):**
```cisco
show interfaces status
show controllers
```

**Capa 2 (Enlace):**
```cisco
show interfaces
show spanning-tree
show vlan
```

**Capa 3 (Red):**
```cisco
show ip route
show ip interface brief
show ip protocols
```

**Capa 4-7 (Transporte-Aplicación):**
```cisco
show ip nat translations
show access-lists
telnet [ip] [puerto]
```

---

## 🔷 10. Comandos de Demostración {#demostracion}

### **Demo 1: Conectividad entre Sucursales**

```cisco
! Desde Router Córdoba
ping 192.168.44.1
traceroute 192.168.44.1
show ip route 192.168.44.0
```

**Qué mostrar:**
- Ping exitoso
- Ruta tomada (directo o vía BS.AS)
- Entrada en tabla de enrutamiento OSPF

---

### **Demo 2: Failover de WAN**

```cisco
! En Router Buenos Aires

! 1. Mostrar ruta actual
show ip route 0.0.0.0

! 2. Apagar WAN1
interface GigabitEthernet0/1.100
shutdown

! 3. Esperar 2 segundos y verificar
show ip route 0.0.0.0

! 4. Reactivar WAN1
interface GigabitEthernet0/1.100
no shutdown

! 5. Verificar que vuelve a WAN1
show ip route 0.0.0.0
```

**Qué mostrar:**
- Cambio automático de WAN1 a WAN2
- Tiempo de convergencia (~2 segundos)
- Retorno automático a WAN1

---

### **Demo 3: ACL Bloqueando FTP**

```cisco
! Desde PC Córdoba (192.168.10.5)
telnet 192.168.20.10 21
! Resultado: Connection refused o timeout

! Desde PC Buenos Aires (192.168.30.10)
telnet 192.168.20.10 21
! Resultado: Connected

! En Router Córdoba, verificar ACL
show access-lists FTP_BLOCK
```

**Qué mostrar:**
- Bloqueo de FTP desde Córdoba
- Permiso de FTP desde Buenos Aires
- Contadores de ACL incrementándose

---

### **Demo 4: OSPF Convergencia**

```cisco
! En Router Buenos Aires

! 1. Ver vecinos OSPF
show ip ospf neighbor

! 2. Apagar enlace P2P a Córdoba
interface GigabitEthernet0/0/0
shutdown

! 3. Esperar y verificar tabla de enrutamiento
show ip route ospf

! 4. Ver que la ruta cambió
show ip route 192.168.10.0

! 5. Reactivar enlace
interface GigabitEthernet0/0/0
no shutdown
```

**Qué mostrar:**
- Pérdida de adyacencia OSPF
- Reconvergencia automática
- Nueva ruta calculada

---

### **Demo 5: NAT en Acción**

```cisco
! En Router Buenos Aires

! 1. Limpiar traducciones
clear ip nat translation *

! 2. Desde PC interno, hacer ping a Internet
! (Desde PC: ping 8.8.8.8)

! 3. Ver traducciones NAT
show ip nat translations

! 4. Ver estadísticas
show ip nat statistics
```

**Qué mostrar:**
- Traducción de IP privada a pública
- Uso de PAT (diferentes puertos)
- Múltiples conexiones simultáneas

---

### **Demo 6: DHCP Relay**

```cisco
! En Router Mendoza

! 1. Ver configuración de DHCP Relay
show running-config interface GigabitEthernet0/2.44

! 2. Desde cliente WiFi, solicitar IP
! (Desde PC: ipconfig /release, ipconfig /renew)

! 3. En Router, ver estadísticas
show ip interface GigabitEthernet0/2.44 | include Helper

! 4. Verificar que cliente recibió IP
! (Desde PC: ipconfig)
```

**Qué mostrar:**
- Cliente sin IP solicita DHCP
- Router reenvía solicitud a servidor
- Cliente recibe IP del rango correcto

---

## 📊 Comandos por Escenario de Troubleshooting

### **Problema: No hay conectividad entre PCs**

```cisco
! 1. Verificar interfaces
show ip interface brief

! 2. Verificar VLANs
show vlan brief

! 3. Verificar enrutamiento
show ip route

! 4. Verificar OSPF
show ip ospf neighbor

! 5. Hacer ping y traceroute
ping [destino]
traceroute [destino]
```

---

### **Problema: No hay salida a Internet**

```cisco
! 1. Verificar ruta por defecto
show ip route 0.0.0.0

! 2. Verificar NAT
show ip nat translations
show ip nat statistics

! 3. Verificar interfaces WAN
show ip interface brief | include Gig0/1

! 4. Hacer ping a gateway ISP
ping 42.25.25.2

! 5. Hacer ping a Internet
ping 8.8.8.8
```

---

### **Problema: ACL no funciona como esperado**

```cisco
! 1. Ver ACL
show access-lists [nombre]

! 2. Ver dónde está aplicada
show ip interface [interfaz] | include access list

! 3. Ver contadores
show access-lists [nombre] | include matches

! 4. Verificar orden de reglas
show running-config | section access-list

! 5. Limpiar contadores y probar
clear access-list counters [nombre]
```

---

### **Problema: OSPF no forma adyacencias**

```cisco
! 1. Ver vecinos
show ip ospf neighbor

! 2. Ver interfaces OSPF
show ip ospf interface

! 3. Verificar configuración
show running-config | section router ospf

! 4. Verificar que las redes estén anunciadas
show ip protocols

! 5. Debug (con precaución)
debug ip ospf adj
```

---

## 🎯 Comandos Esenciales para la Presentación

### **Top 10 Comandos a Memorizar**

1. `show ip route` - Tabla de enrutamiento
2. `show ip ospf neighbor` - Vecinos OSPF
3. `show ip interface brief` - Estado de interfaces
4. `show vlan brief` - VLANs configuradas
5. `show ip nat translations` - Traducciones NAT
6. `show access-lists` - ACLs y estadísticas
7. `show spanning-tree` - Estado de STP
8. `ping [destino]` - Prueba de conectividad
9. `traceroute [destino]` - Ruta de paquetes
10. `show running-config` - Configuración actual

---

## 💡 Tips de Comandos

### **Filtrado de Salida**

```cisco
! Buscar líneas que contengan una palabra
show running-config | include ospf

! Buscar secciones
show running-config | section router ospf

! Excluir líneas
show running-config | exclude !

! Comenzar desde una línea
show running-config | begin interface
```

### **Paginación**

```cisco
! Desactivar paginación (mostrar todo)
terminal length 0

! Reactivar paginación
terminal length 24
```

### **Guardar Configuración**

```cisco
! Guardar configuración actual
write memory
! o
copy running-config startup-config

! Verificar que se guardó
show startup-config
```

---

## 🔧 Comandos de Configuración Rápida

### **Entrar a Modo de Configuración**

```cisco
! Modo privilegiado
enable

! Modo de configuración global
configure terminal

! Salir de configuración
end
! o
Ctrl+Z
```

### **Configurar Interfaz**

```cisco
configure terminal
interface GigabitEthernet0/0/0
 ip address 10.10.1.9 255.255.255.252
 no shutdown
 description P2P_to_CORDOBA
end
```

### **Guardar y Salir**

```cisco
end
write memory
exit
```

---

## ⚠️ Comandos de Precaución

**NO usar en producción sin supervisión:**

```cisco
! Debug (genera mucho tráfico)
debug ip ospf adj
debug ip nat
debug ip packet

! SIEMPRE desactivar debug después
undebug all

! Reload (reinicia el dispositivo)
reload

! Limpiar configuración (borra todo)
write erase
```

---

## 📝 Plantilla de Troubleshooting

```cisco
! ========================================
! PLANTILLA DE TROUBLESHOOTING
! ========================================

! 1. VERIFICAR CONECTIVIDAD BÁSICA
show ip interface brief
ping [destino]
traceroute [destino]

! 2. VERIFICAR ENRUTAMIENTO
show ip route
show ip protocols
show ip ospf neighbor

! 3. VERIFICAR VLANS (en switches)
show vlan brief
show interfaces trunk

! 4. VERIFICAR NAT (si aplica)
show ip nat translations
show ip nat statistics

! 5. VERIFICAR ACLS (si aplica)
show access-lists
show ip interface [interfaz] | include access list

! 6. VERIFICAR LOGS
show logging

! 7. GUARDAR EVIDENCIA
show tech-support
```

---

## ✅ Checklist de Comandos para Demostración

**Antes de la presentación, verificar:**

- [ ] `show ip route` - Rutas correctas
- [ ] `show ip ospf neighbor` - Todos los vecinos FULL
- [ ] `show ip interface brief` - Todas las interfaces up/up
- [ ] `show vlan brief` - VLANs configuradas
- [ ] `show ip nat translations` - NAT funcionando
- [ ] `show access-lists` - ACLs con matches
- [ ] `ping` entre todos los sitios - Conectividad OK
- [ ] `show spanning-tree` - No hay puertos bloqueados inesperadamente

---

**Esta guía de comandos es tu referencia rápida para verificar, troubleshoot y demostrar tu red. ¡Úsala con confianza!** 🚀
