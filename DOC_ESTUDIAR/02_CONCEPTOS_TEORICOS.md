# 📚 CONCEPTOS TEÓRICOS DE REDES - GUÍA DE ESTUDIO

Esta guía explica en detalle todos los conceptos teóricos implementados en la red empresarial.

---

## 📖 Índice
1. [OSPF - Open Shortest Path First](#ospf)
2. [VLANs - Virtual LANs](#vlans)
3. [NAT - Network Address Translation](#nat)
4. [ACLs - Access Control Lists](#acls)
5. [Spanning Tree Protocol](#stp)
6. [DHCP y DHCP Relay](#dhcp)
7. [Enrutamiento Estático vs Dinámico](#enrutamiento)
8. [Subnetting y VLSM](#subnetting)

---

## 🔷 1. OSPF - Open Shortest Path First {#ospf}

### **¿Qué es OSPF?**
OSPF (Open Shortest Path First) es un **protocolo de enrutamiento dinámico de estado de enlace** que permite a los routers compartir información sobre la topología de red y calcular automáticamente las mejores rutas.

### **Características Principales**

#### **Protocolo de Estado de Enlace**
- Cada router mantiene un **mapa completo de la topología** (LSDB - Link State Database)
- Los routers intercambian **LSAs (Link State Advertisements)** para compartir información
- Usa el **algoritmo de Dijkstra (SPF)** para calcular el árbol de rutas más cortas

#### **Áreas OSPF**
- **Área 0 (Backbone):** Área central a la que todas las demás deben conectarse
- En nuestra red: Todos los routers están en **Área 0**

#### **Métrica: Costo**
```
Costo = Ancho de Banda de Referencia / Ancho de Banda de la Interfaz
```
- **Ancho de banda de referencia:** 100 Mbps (por defecto)
- **Menor costo = Mejor ruta**

**En nuestra red:**
- Enlaces P2P directos: **Costo 10** (rutas preferidas)
- Enlace de backup (VLAN 1000): **Costo 50** (ruta secundaria)

#### **Router ID**
Identificador único de cada router OSPF:
- Buenos Aires: `1.1.1.1`
- Córdoba: `2.2.2.2`
- Mendoza: `3.3.3.3`

#### **Tipos de Redes OSPF**

**Point-to-Point (P2P):**
```
ip ospf network point-to-point
```
- No requiere elección de DR/BDR
- Convergencia más rápida
- Usado en nuestros enlaces P2P entre routers

**Broadcast (por defecto en Ethernet):**
- Requiere elección de DR (Designated Router) y BDR (Backup DR)
- Usado en VLAN 1000 (enlace de backup)

#### **DR y BDR (Designated Router)**
- **DR:** Router principal que centraliza la distribución de LSAs
- **BDR:** Backup del DR
- **Elección basada en:**
  1. Prioridad OSPF (mayor gana)
  2. Router ID más alto (desempate)

**En nuestra red (VLAN 1000):**
- Buenos Aires: Prioridad **100** → DR
- Córdoba: Prioridad **50** → BDR
- Mendoza: Prioridad **50**

#### **Passive Interface**
```
passive-interface GigabitEthernet0/1.30
```
- **Propósito:** Evitar envío de paquetes OSPF Hello en interfaces LAN
- **Beneficios:**
  - Seguridad (no exponer OSPF a usuarios)
  - Reducción de tráfico innecesario
- **Efecto:** La red sigue siendo anunciada, pero no se forman adyacencias

**En nuestra red:**
- Todas las interfaces LAN (VLANs de usuarios) son passive

#### **Default Information Originate**
```
default-information originate
```
- **Propósito:** Inyectar una ruta por defecto (0.0.0.0/0) en OSPF
- **Configurado en:** Router Buenos Aires
- **Efecto:** Todos los routers aprenden la ruta por defecto hacia Internet

### **Proceso de Convergencia OSPF**

1. **Descubrimiento de Vecinos:**
   - Paquetes **Hello** cada 10 segundos
   - Dead interval: 40 segundos

2. **Intercambio de LSDB:**
   - Paquetes DBD (Database Description)
   - LSR (Link State Request)
   - LSU (Link State Update)

3. **Cálculo de Rutas:**
   - Algoritmo SPF (Dijkstra)
   - Instalación en tabla de enrutamiento

4. **Mantenimiento:**
   - LSAs se refrescan cada 30 minutos
   - Detección de cambios y reconvergencia

### **Ventajas de OSPF**
✅ Convergencia rápida ante fallas  
✅ Soporte para redes grandes (jerárquico)  
✅ Sin límite de saltos (vs RIP: 15 saltos)  
✅ Uso eficiente del ancho de banda  
✅ Soporte para VLSM y CIDR  
✅ Autenticación de vecinos  

---

## 🔷 2. VLANs - Virtual LANs {#vlans}

### **¿Qué es una VLAN?**
Una **VLAN (Virtual LAN)** es una **red lógica** que agrupa dispositivos en el mismo dominio de broadcast, independientemente de su ubicación física.

### **Beneficios de las VLANs**

1. **Segmentación de Red:**
   - Separar usuarios de servidores
   - Aislar WiFi de invitados
   - Crear redes de gestión

2. **Seguridad:**
   - Limitar el alcance de broadcasts
   - Controlar el tráfico entre VLANs

3. **Rendimiento:**
   - Reducir dominios de broadcast
   - Optimizar el uso del ancho de banda

4. **Flexibilidad:**
   - Reorganizar la red sin cambios físicos
   - Facilitar la gestión

### **Tipos de Puertos**

#### **Puerto de Acceso (Access Port)**
```
switchport mode access
switchport access vlan 10
```
- **Propósito:** Conectar dispositivos finales (PCs, servidores)
- **Característica:** Pertenece a **una sola VLAN**
- **Tráfico:** Sin etiqueta (untagged)

#### **Puerto Troncal (Trunk Port)**
```
switchport mode trunk
switchport trunk allowed vlan 10,20
```
- **Propósito:** Conectar switches o routers
- **Característica:** Transporta **múltiples VLANs**
- **Protocolo:** IEEE 802.1Q (etiquetado de frames)
- **Tráfico:** Con etiqueta (tagged), excepto VLAN nativa

#### **VLAN Nativa**
```
switchport trunk native vlan 70
```
- **Propósito:** VLAN que viaja **sin etiqueta** en un trunk
- **Uso común:** Management, CDP, VTP
- **En nuestra red:** VLAN 70 (Management en Mendoza)

### **Inter-VLAN Routing**

**Problema:** Las VLANs están aisladas en capa 2, no pueden comunicarse directamente.

**Solución: Router-on-a-Stick**
```
interface GigabitEthernet0/2
 no ip address
!
interface GigabitEthernet0/2.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/2.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

**Funcionamiento:**
1. Switch envía tráfico etiquetado al router
2. Router recibe frame en subinterfaz correspondiente
3. Router enruta entre VLANs
4. Router envía respuesta por la subinterfaz destino

**En nuestra red:**
- **Córdoba:** Router-on-a-Stick para VLANs 10 y 20
- **Mendoza:** Router-on-a-Stick para VLANs 44, 55 y 70
- **Buenos Aires:** Router-on-a-Stick para VLANs 30, 100 y 200

### **VLANs en Nuestra Red**

| VLAN | Nombre | Red | Propósito |
|------|--------|-----|-----------|
| 10 | Usuarios Córdoba | 192.168.10.0/24 | PCs de usuarios |
| 20 | Servidores Córdoba | 192.168.20.0/24 | Servidor de archivos |
| 30 | LAN Buenos Aires | 192.168.30.0/24 | LAN principal |
| 44 | WiFi Internos | 192.168.44.0/24 | WiFi empleados |
| 55 | WiFi Invitados | 192.168.55.0/24 | WiFi visitantes |
| 70 | Management | 192.168.70.0/24 | Gestión de red |
| 100 | WAN1 | 42.25.25.0/29 | Enlace ISP primario |
| 200 | WAN2 | 43.26.26.0/29 | Enlace ISP secundario |
| 1000 | OSPF Backup | 172.20.10.0/29 | Ruta de respaldo |
| 100 | DNS | 192.168.100.0/29 | Servidor DNS |
| 101 | Web | 192.168.100.8/29 | Servidor Web |

---

## 🔷 3. NAT - Network Address Translation {#nat}

### **¿Qué es NAT?**
**NAT (Network Address Translation)** es una técnica que traduce direcciones IP privadas a direcciones IP públicas, permitiendo que múltiples dispositivos compartan una o pocas IPs públicas.

### **¿Por qué usar NAT?**

1. **Conservación de IPs Públicas:**
   - IPv4 tiene ~4.3 mil millones de direcciones
   - NAT permite reutilizar rangos privados (RFC 1918)

2. **Seguridad:**
   - Oculta la topología interna
   - Los dispositivos internos no son directamente accesibles

3. **Flexibilidad:**
   - Cambiar ISP sin renumerar la red interna

### **Rangos de IPs Privadas (RFC 1918)**
```
10.0.0.0 - 10.255.255.255     (10.0.0.0/8)
172.16.0.0 - 172.31.255.255   (172.16.0.0/12)
192.168.0.0 - 192.168.255.255 (192.168.0.0/16)
```

### **Tipos de NAT**

#### **1. NAT Estático (Static NAT)**
**Mapeo 1:1 entre IP privada e IP pública**

```
ip nat inside source static 192.168.100.2 1.1.1.1
```

**Ejemplo en nuestra red:**
- DNS Server: `192.168.100.2` → `1.1.1.1`
- Web Server: `192.168.100.9` → `45.162.20.10`

**Uso:** Servidores que deben ser accesibles desde Internet

#### **2. NAT Dinámico (Dynamic NAT)**
**Mapeo de un pool de IPs privadas a un pool de IPs públicas**

```
ip nat pool PUBLIC_POOL 200.1.1.1 200.1.1.10 netmask 255.255.255.0
ip nat inside source list 1 pool PUBLIC_POOL
```

**Limitación:** Requiere tantas IPs públicas como conexiones simultáneas

#### **3. PAT - Port Address Translation (NAT Overload)**
**Múltiples IPs privadas comparten UNA IP pública usando puertos diferentes**

```
ip nat inside source list 10 interface GigabitEthernet0/1.100 overload
```

**Funcionamiento:**
```
Interno: 192.168.30.5:50000 → Externo: 42.25.25.1:1024
Interno: 192.168.10.8:50001 → Externo: 42.25.25.1:1025
Interno: 192.168.20.3:50002 → Externo: 42.25.25.1:1026
```

**En nuestra red:**
- **ACL 10:** Define redes internas que usan NAT por WAN1
- **ACL 11:** Define redes internas que usan NAT por WAN2
- **Overload:** Permite miles de conexiones simultáneas

### **Terminología NAT**

- **Inside Local:** IP privada del dispositivo interno (ej: 192.168.30.5)
- **Inside Global:** IP pública asignada (ej: 42.25.25.1)
- **Outside Local:** IP del destino externo vista desde dentro
- **Outside Global:** IP real del destino externo

### **Configuración NAT en Nuestra Red**

**Router Buenos Aires:**
```
interface GigabitEthernet0/1.30
 ip nat inside
!
interface GigabitEthernet0/1.100
 ip nat outside
!
interface GigabitEthernet0/1.200
 ip nat outside
!
ip nat inside source list 10 interface GigabitEthernet0/1.100 overload
ip nat inside source list 11 interface GigabitEthernet0/1.200 overload
!
access-list 10 permit 192.168.30.0 0.0.0.255
access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 permit 192.168.20.0 0.0.0.255
access-list 10 permit 192.168.44.0 0.0.0.255
access-list 10 permit 192.168.55.0 0.0.0.255
access-list 10 permit 192.168.70.0 0.0.0.255
```

**Redundancia NAT:**
- **ACL 10:** NAT por WAN1 (primario)
- **ACL 11:** NAT por WAN2 (backup)
- Si WAN1 falla, el tráfico automáticamente usa WAN2

---

## 🔷 4. ACLs - Access Control Lists {#acls}

### **¿Qué es una ACL?**
Una **ACL (Access Control List)** es una lista de reglas que **permiten o deniegan tráfico** basándose en criterios como IP origen, IP destino, protocolo y puerto.

### **Tipos de ACLs**

#### **ACL Estándar (Standard ACL)**
```
access-list 10 permit 192.168.30.0 0.0.0.255
```
- **Número:** 1-99, 1300-1999
- **Filtrado:** Solo por **IP origen**
- **Ubicación:** Lo más cerca posible del **destino**
- **Uso:** NAT, filtrado simple

#### **ACL Extendida (Extended ACL)**
```
ip access-list extended FTP_BLOCK
 deny tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq ftp
 permit ip any any
```
- **Número:** 100-199, 2000-2699
- **Filtrado:** IP origen, IP destino, protocolo, puerto
- **Ubicación:** Lo más cerca posible del **origen**
- **Uso:** Control de acceso granular

### **Wildcard Mask**

**Concepto:** Inverso de la máscara de subred
```
Máscara de subred: 255.255.255.0
Wildcard mask:     0.0.0.255
```

**Regla:**
- **0:** Debe coincidir exactamente
- **255:** No importa (cualquier valor)

**Ejemplos:**
```
host 192.168.20.10          → 192.168.20.10 0.0.0.0
192.168.10.0 0.0.0.255      → Toda la red 192.168.10.0/24
any                         → 0.0.0.0 255.255.255.255
```

### **Orden de Procesamiento**

1. Las ACLs se procesan **de arriba hacia abajo**
2. Primera coincidencia gana (**first match**)
3. **Deny implícito** al final (deny any any)
4. Siempre terminar con `permit ip any any` si se requiere

### **ACLs en Nuestra Red**

#### **Objetivo:** Solo PC de Buenos Aires (192.168.30.10) puede acceder al servidor de archivos FTP (192.168.20.10)

**En Router Córdoba (Interfaz VLAN 20):**
```
ip access-list extended FTP_BLOCK
 permit ip host 192.168.30.10 host 192.168.20.10
 deny ip any host 192.168.20.10
 permit ip any any
!
interface GigabitEthernet0/2.20
 ip access-group FTP_BLOCK out
```

**Lógica:**
1. **Línea 1:** Permitir todo el tráfico desde 192.168.30.10 al servidor
2. **Línea 2:** Denegar todo lo demás hacia el servidor
3. **Línea 3:** Permitir el resto del tráfico

**En Router Córdoba (Interfaz P2P):**
```
ip access-list extended FTP_BLOCK_P2P
 deny tcp 192.168.44.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.55.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.70.0 0.0.0.255 host 192.168.20.10 eq ftp
 permit ip any any
!
interface GigabitEthernet0/0/0
 ip access-group FTP_BLOCK_P2P in
```

**Lógica:** Bloquear FTP desde Mendoza (VLANs 44, 55, 70) al servidor

**En Switch ACC-CORD (Puerto del Servidor):**
```
ip access-list extended FTP_BLOCK
 deny tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.20.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.44.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.55.0 0.0.0.255 host 192.168.20.10 eq ftp
 deny tcp 192.168.70.0 0.0.0.255 host 192.168.20.10 eq ftp
 permit ip any any
!
interface FastEthernet0/2
 ip access-group FTP_BLOCK in
```

**Lógica:** Bloquear FTP desde todas las redes excepto Buenos Aires

### **Buenas Prácticas ACLs**

✅ Usar ACLs nombradas (más legibles)  
✅ Documentar con `description`  
✅ Colocar reglas más específicas primero  
✅ Terminar con `permit ip any any` si se requiere  
✅ Aplicar ACLs estándar cerca del destino  
✅ Aplicar ACLs extendidas cerca del origen  
✅ Revisar con `show access-lists`  

---

## 🔷 5. Spanning Tree Protocol (STP) {#stp}

### **¿Qué es STP?**
**Spanning Tree Protocol (STP)** es un protocolo de capa 2 que **previene loops** en redes con enlaces redundantes, bloqueando puertos estratégicamente.

### **Problema: Loops en Capa 2**

**Sin STP:**
```
Switch A ←→ Switch B
    ↓          ↓
    └─ Switch C ─┘
```

**Consecuencias:**
- **Broadcast Storm:** Frames de broadcast circulan infinitamente
- **Inestabilidad de tabla MAC:** Switches aprenden MACs por múltiples puertos
- **Duplicación de frames:** Mismo frame llega múltiples veces

### **Solución: STP**

STP crea un **árbol lógico sin loops** bloqueando puertos redundantes.

### **Proceso de STP**

#### **1. Elección del Root Bridge**
- Switch con **menor Bridge ID** (Prioridad + MAC)
- **Prioridad por defecto:** 32768
- **Prioridad configurable:** Múltiplos de 4096

**En nuestra red:**
```
spanning-tree vlan 10,20 priority 4096
```
- SW-CORE-DIS-CORD: Prioridad **4096** → Root Bridge para VLANs 10, 20
- SW-CORE-DIS-MEND: Prioridad **24576** → Root Bridge para VLANs 44, 55, 70
- SW_OSPF_BACKUP: Prioridad **4096** → Root Bridge para VLAN 1000

#### **2. Selección de Puertos**

**Root Port (RP):**
- Puerto con **menor costo** hacia el Root Bridge
- **Un RP por switch** (excepto Root Bridge)

**Designated Port (DP):**
- Puerto que **reenvía tráfico** en cada segmento
- Todos los puertos del Root Bridge son DP

**Blocked Port:**
- Puerto **bloqueado** para prevenir loops
- Escucha BPDUs pero no reenvía tráfico

#### **3. Cálculo de Costo**

| Velocidad | Costo STP |
|-----------|-----------|
| 10 Mbps | 100 |
| 100 Mbps | 19 |
| 1 Gbps | 4 |
| 10 Gbps | 2 |

### **Estados de Puerto STP**

1. **Blocking:** Escucha BPDUs, no reenvía tráfico (20 seg)
2. **Listening:** Procesa BPDUs, no aprende MACs (15 seg)
3. **Learning:** Aprende MACs, no reenvía tráfico (15 seg)
4. **Forwarding:** Reenvía tráfico normalmente
5. **Disabled:** Puerto administrativamente apagado

**Tiempo de convergencia:** ~50 segundos (Blocking → Forwarding)

### **PVST (Per-VLAN Spanning Tree)**

**Concepto:** Instancia STP **independiente por VLAN**

**Ventajas:**
- Balanceo de carga entre VLANs
- Diferentes Root Bridges por VLAN

**En nuestra red:**
```
spanning-tree mode pvst
```

**Ejemplo:**
- VLAN 10: Root Bridge = SW-CORE-DIS-CORD
- VLAN 44: Root Bridge = SW-CORE-DIS-MEND

### **PortFast**

```
spanning-tree portfast
```

**Propósito:** Transición inmediata a **Forwarding** en puertos de acceso

**Uso:** Puertos conectados a PCs, servidores (no switches)

**Beneficio:** Evitar espera de 50 segundos al conectar dispositivos

**En nuestra red:** Habilitado en todos los puertos de acceso

### **STP en Nuestra Red**

| Switch | VLANs | Prioridad | Rol |
|--------|-------|-----------|-----|
| SW-CORE-DIS-CORD | 10, 20 | 4096 | Root Bridge |
| SW-CORE-DIS-MEND | 44, 55, 70 | 24576 | Root Bridge |
| SW_OSPF_BACKUP | 1000 | 4096 | Root Bridge |
| SW-ACC-MEND | 44, 55, 70 | 49152 | Non-root |
| SW-ACC-CORD | 10, 20 | 32768 (default) | Non-root |

---

## 🔷 6. DHCP y DHCP Relay {#dhcp}

### **¿Qué es DHCP?**
**DHCP (Dynamic Host Configuration Protocol)** asigna automáticamente configuración IP a dispositivos:
- Dirección IP
- Máscara de subred
- Gateway por defecto
- Servidores DNS

### **Proceso DHCP (DORA)**

1. **Discover:** Cliente busca servidor DHCP (broadcast)
2. **Offer:** Servidor ofrece una IP
3. **Request:** Cliente solicita la IP ofrecida
4. **Acknowledge:** Servidor confirma la asignación

### **Problema: Broadcasts no cruzan routers**

**Escenario:**
```
DHCP Server (VLAN 70) ← Router → Cliente WiFi (VLAN 44)
```

**Problema:** DHCP Discover (broadcast) no llega al servidor

### **Solución: DHCP Relay (IP Helper)**

```
interface GigabitEthernet0/2.44
 ip helper-address 192.168.70.10
```

**Funcionamiento:**
1. Cliente envía DHCP Discover (broadcast)
2. Router recibe el broadcast
3. Router convierte a **unicast** hacia 192.168.70.10
4. Servidor DHCP responde
5. Router reenvía la respuesta al cliente

### **En Nuestra Red**

**DHCP Server:** `192.168.70.10` (Mendoza)

**DHCP Relay configurado en Router Mendoza:**
```
interface GigabitEthernet0/2.44
 ip helper-address 192.168.70.10
!
interface GigabitEthernet0/2.55
 ip helper-address 192.168.70.10
```

**Beneficio:** Centralizar DHCP para WiFi Internos (VLAN 44) e Invitados (VLAN 55)

---

## 🔷 7. Enrutamiento Estático vs Dinámico {#enrutamiento}

### **Enrutamiento Estático**

**Definición:** Rutas configuradas **manualmente** por el administrador

```
ip route 0.0.0.0 0.0.0.0 42.25.25.2
```

**Ventajas:**
✅ Control total sobre rutas  
✅ Sin overhead de protocolos  
✅ Predecible  
✅ Seguro (no anuncia rutas)  

**Desventajas:**
❌ No se adapta a cambios  
❌ Difícil de escalar  
❌ Requiere configuración manual  

**Uso en nuestra red:**
- Rutas por defecto hacia ISPs
- Rutas específicas a servidores públicos
- Rutas de backup con métricas administrativas

### **Enrutamiento Dinámico (OSPF)**

**Definición:** Routers **aprenden rutas automáticamente**

**Ventajas:**
✅ Convergencia automática ante fallas  
✅ Escalable  
✅ Balanceo de carga  
✅ Adaptativo  

**Desventajas:**
❌ Consumo de CPU y memoria  
❌ Ancho de banda para actualizaciones  
❌ Complejidad de configuración  

**Uso en nuestra red:**
- Enrutamiento entre sucursales (BS.AS, Córdoba, Mendoza)
- Distribución de rutas LAN
- Failover automático

### **Distancia Administrativa**

**Concepto:** Confiabilidad de una fuente de ruta (menor = más confiable)

| Fuente | AD |
|--------|-----|
| Directamente conectada | 0 |
| Ruta estática | 1 |
| OSPF | 110 |
| RIP | 120 |

**En nuestra red:**
```
ip route 0.0.0.0 0.0.0.0 42.25.25.2      ! AD = 1 (primaria)
ip route 0.0.0.0 0.0.0.0 43.26.26.2 5    ! AD = 5 (backup)
```

**Efecto:** WAN1 se usa primero; si falla, WAN2 toma el control

---

## 🔷 8. Subnetting y VLSM {#subnetting}

### **¿Qué es Subnetting?**
**Subnetting** es dividir una red grande en **subredes más pequeñas** para:
- Optimizar el uso de IPs
- Mejorar la seguridad
- Reducir broadcasts

### **Máscara de Subred**

**Notación CIDR:**
```
192.168.10.0/24
```
- **/24:** 24 bits para red, 8 bits para hosts
- **Hosts disponibles:** 2^8 - 2 = 254

**Tabla de Máscaras:**
| CIDR | Máscara | Hosts |
|------|---------|-------|
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /29 | 255.255.255.248 | 6 |
| /30 | 255.255.255.252 | 2 |

### **VLSM (Variable Length Subnet Mask)**

**Concepto:** Usar diferentes máscaras de subred según las necesidades

**En nuestra red:**
- **LANs:** /24 (254 hosts) → Usuarios
- **WAN:** /29 (6 hosts) → Enlaces ISP
- **P2P:** /30 (2 hosts) → Enlaces punto a punto

**Ejemplo P2P:**
```
Red: 10.10.1.0/30
Máscara: 255.255.255.252
Hosts: .1 (Mendoza), .2 (BS.AS)
Broadcast: .3
```

**Beneficio:** Uso eficiente de IPs (no desperdiciar 252 IPs en un enlace P2P)

---

## 📝 Resumen de Conceptos

| Concepto | Capa OSI | Propósito |
|----------|----------|-----------|
| OSPF | 3 (Red) | Enrutamiento dinámico |
| VLANs | 2 (Enlace) | Segmentación lógica |
| NAT | 3 (Red) | Traducción de IPs |
| ACLs | 3-4 (Red/Transporte) | Control de acceso |
| STP | 2 (Enlace) | Prevención de loops |
| DHCP | 7 (Aplicación) | Asignación automática de IPs |

---

**Esta guía cubre todos los conceptos teóricos implementados en la red. Úsala como referencia para entender el "por qué" detrás de cada configuración.**
