# Análisis del Proyecto - Trabajo Práctico de Redes

## 📋 Descripción General del Proyecto

Este proyecto implementa una red empresarial multi-sitio con **tres ubicaciones principales**: Buenos Aires (BS.AS), Córdoba y Mendoza. La topología incluye conexión a Internet a través de ISPs, routing dinámico OSPF, VLANs segmentadas, redundancia STP, WiFi dual-SSID y servicios NAT.

---

## 🏗️ Arquitectura de la Red

### Ubicaciones Principales

#### 1. **Buenos Aires (BS.AS)** - Sitio Central
- **Router Principal**: Conexión dual a ISP_LOCAL
- **VLANs**: 
  - VLAN 20 (Red Local BS.AS)
- **Dispositivos**:
  - PC-BS-AS (192.168.0.0/24)
- **Función**: Gateway principal a Internet con NAT

#### 2. **Córdoba** - Sitio Secundario
- **Switches**: 
  - SW11_CORDOBA (Switch de Acceso)
  - SW_CORE_DS_CORO (Switch Core/Distribución)
- **VLANs**:
  - VLAN 10 (192.168.0.0/24) - Red Corporativa
  - VLAN 20 (192.168.20.0/24) - Red de Servidores
- **Dispositivos**:
  - **PC2-VLAN10**: 192.168.10.10 (Usuario)
  - **FILESERVER-INTERNO-VLAN20**: 192.168.20.10 (Servidor de Archivos)
- **Función**: Sitio corporativo con servicios de archivos

#### 3. **Mendoza** - Sitio Terciario
- **Switches**: 
  - SW11_MENDOZA (Switch de Acceso)
  - SW_CORE_DS_MEND (Switch Core/Distribución)
  - SW_CORE_DS_HEND (Switch de Servicios)
  - SW_MEST_DHCP_SERVER (Switch Servidor DHCP)
- **VLANs**:
  - VLAN 44 (192.168.44.0/24) - WiFi Internos/Empleados
  - VLAN 55 (192.168.55.0/24) - WiFi Invitados/Externos
  - VLAN 70 (192.168.70.0/24) - Red de Management (VLAN Nativa)
- **Dispositivos**:
  - **Access Point Mendoza**: 192.168.70.3
  - **LAPTOP-NOTEBOOK-INTERNOS**: WiFi (SSID-INTERNOS - VLAN 44)
  - **LAPTOP-NOTEBOOK-EXTERNOS**: WiFi (SSID-INVITADOS - VLAN 55)
  - **PC-DA-MENDOZA**: 192.168.2.0/4
  - **Servidor DHCP**: Para distribución de IPs
- **Función**: Sitio con servicios WiFi segmentados y gestión de red inalámbrica

---

## 🌐 Topología de Conectividad

### Enlaces entre Routers (Fibra Óptica - P2P)

```
BS.AS ←──────────────→ Córdoba
  │                        │
  │                        │
  └──────────→ Mendoza ←───┘
```

**Enlaces Point-to-Point (Fibra):**
- BS.AS ↔ Córdoba: Conexión directa en interfaz física
- BS.AS ↔ Mendoza: Conexión directa en interfaz física
- Córdoba ↔ Mendoza: Conexión directa en interfaz física

**Red de Respaldo (Broadcast):**
- Switch SW_OSPF_BACKUP conecta los 3 routers
- VLAN 1000 para routing
- Subinterfaz .1000 en cada router (Routing On A Stick)

### Conexión a Internet

```
Internet
   │
ISP_INTERNACIONAL
   │
ISP_LOCAL (SP_LOCAL)
   │
   ├─── WAN1 VLAN 100 (81.26.24.0/29)
   └─── WAN2 VLAN 102 (68.69.85.0/24)
        │
   Router BS.AS (Dual-WAN)
```

**Direccionamiento WAN:**
- VLAN 100: 81.26.24.0/29
- VLAN 102: 68.69.85.0/24
- LAN LOCAL: 192.168.20.0/24

---

## 🔧 Configuraciones Requeridas

### 1️⃣ INTERFACES

#### Enlaces P2P entre Routers (FIBRA)
- ✅ **ACTUALIZACIÓN DEL PROFESOR**: Configurar IPs directamente en interfaces físicas
- ❌ ~~NO usar subinterfaz .500~~ (limitación de Packet Tracer)
- ✅ Permitirá configurar OSPF tipo Point-to-Point correctamente

#### Redes Locales (VLANs)
- **BS.AS**: VLAN 20
- **Córdoba**: VLAN 10, VLAN 20
- **Mendoza**: VLAN 44, VLAN 55, VLAN 70

#### Conexión ISP
- Configurar IPs directamente en interfaces físicas entre ISP_LOCAL e ISP_INTERNACIONAL

---

### 2️⃣ RUTEO OSPF

#### Vecindades Point-to-Point (sobre enlaces de fibra)
1. **BS.AS ↔ Córdoba**: Vecindad OSPF tipo Point-to-Point
2. **BS.AS ↔ Mendoza**: Vecindad OSPF tipo Point-to-Point
3. **Córdoba ↔ Mendoza**: Vecindad OSPF tipo Point-to-Point

#### Vecindad Broadcast (red de respaldo)
- **SW_OSPF_BACKUP**: 
  - VLAN 1000 a nivel de switch
  - Subinterfaz .1000 en cada router (Routing On A Stick)
  - NO configurar IP en interfaces físicas
  - Vecindad OSPF tipo Broadcast entre los 3 sitios

#### Propagación de Rutas
- ✅ Todas las vecindades OSPF deben propagar las redes locales
- ✅ Configurar interfaces LAN de cada router como **Interfaces Pasivas**

---

### 3️⃣ RUTEO ESTÁTICO

#### Salida a Internet (desde BS.AS)
- Configurar **2 rutas estáticas predeterminadas** hacia ISP_LOCAL
- Tráfico hacia Internet

#### Comunicación ISP
- Ruteo estático entre ISP_LOCAL e ISP_INTERNACIONAL
- Permitir comunicación entre servicios

---

### 4️⃣ SPANNING TREE PROTOCOL (STP)

#### Configuración General
- ✅ Configurar STP en **todos los switches** de LAN interna
- ✅ El switch de **Core/Distribución** debe ser el **Root Bridge** de cada LAN

**Switches afectados:**
- **SW_DIST_IB** (Distribución)
- **SW_CORP_BA_COLP** (Corporativo Buenos Aires)
- **SW11_CORDOBA** (Acceso Córdoba)
- **SW_CORE_DS_CORO** (Core/Distribución Córdoba) - **ROOT BRIDGE**
- **SW11_MENDOZA** (Acceso Mendoza)
- **SW_CORE_DS_MEND** (Core/Distribución Mendoza) - **ROOT BRIDGE**
- **SW_CORE_DS_HEND** (Servicios Mendoza)
- **SW_MEST_DHCP_SERVER** (Servidor DHCP)

---

### 5️⃣ WiFi (MENDOZA)

#### Configuración Dual-SSID
- **SSID-INTERNOS**: VLAN 44 (192.168.44.0/24)
  - Para empleados y dispositivos corporativos
  - Laptop-Notebook-Internos
- **SSID-INVITADOS**: VLAN 55 (192.168.55.0/24)
  - Para visitantes y dispositivos externos
  - Laptop-Notebook-Externos

**Access Point:**
- **IP Management**: 192.168.70.3
- **VLAN Management**: VLAN 70 (Nativa)
- **Ubicación**: Mendoza, conectado a SW_CORE_DS_MEND

**Segmentación:**
- Tráfico interno aislado del tráfico de invitados
- Ambas redes propagadas por el mismo Access Point
- VLAN 70 como red de gestión (management) del AP

---

### 6️⃣ NAT (NETWORK ADDRESS TRANSLATION)

#### NAT en Buenos Aires (Salida a Internet)
- **Tipo**: NAT por traducción de puertos (PAT/NAT por desborde)
- **Interfaces**: Dual-WAN hacia ISP_LOCAL
  - Interface 1 → NAT con IP de interfaz 1
  - Interface 2 → NAT con IP de interfaz 2
- **Tráfico**: Todo el tráfico saliente de la red corporativa

#### NAT de Servicios (Servidores Públicos)

##### Web Server
- **Tipo**: NAT Estático
- **IP Interna**: (IP privada del servidor web)
- **IP Externa**: 45.162.20.10
- **Propósito**: Acceso público al servidor web

##### DNS Server
- **Tipo**: NAT Estático
- **IP Interna**: (IP privada del servidor DNS)
- **IP Externa**: 1.1.1.1
- **Propósito**: Servicio DNS público

---

### 7️⃣ LISTAS DE CONTROL DE ACCESO (ACL)

#### Regla de Seguridad FTP
- **Política**: Solo PC-BS-AS puede acceder al servidor FTP
- **Restricción**: Bloquear acceso desde todos los demás dispositivos
- **Tipo de ACL**: Extended ACL (filtrado por IP origen/destino y servicio)

**Configuración requerida:**
```
PERMITIR: PC-BS-AS (192.168.0.0/24) → FTP Server (puerto 21)
DENEGAR: Todos los demás → FTP Server
```

---

## �️ Inventario Completo de Dispositivos

### Routers
| Dispositivo | Ubicación | Función | Interfaces Principales |
|-------------|-----------|---------|------------------------|
| Router BS.AS | Buenos Aires | Gateway Principal, NAT, Dual-WAN | 2x WAN (ISP), LAN VLAN 20, P2P a Córdoba/Mendoza, Subint .1000 |
| Router Córdoba | Córdoba | Routing OSPF, Inter-VLAN | LAN VLAN 10/20, P2P a BS.AS/Mendoza, Subint .1000 |
| Router Mendoza | Mendoza | Routing OSPF, WiFi Gateway | LAN VLAN 44/55/70, P2P a BS.AS/Córdoba, Subint .1000 |
| ISP_LOCAL (SP_LOCAL) | Proveedor | Conexión Internet | 2x hacia Router BS.AS, 1x hacia ISP_INTERNACIONAL |
| ISP_INTERNACIONAL | Proveedor | Salida Internet Global | 1x hacia ISP_LOCAL |

### Switches - Buenos Aires
| Dispositivo | Tipo | VLANs | Función |
|-------------|------|-------|---------|
| SW_DIST_IB | Distribución | 10, 100, 102, 200 | Switch de distribución principal |
| SW_CORP_BA_COLP | Core BA | 1000 | Switch corporativo, OSPF Backup |

### Switches - Córdoba
| Dispositivo | Tipo | VLANs | Función |
|-------------|------|-------|---------|
| SW_CORE_DS_CORO | Core/Distribución | 10, 20 | **Root Bridge** de Córdoba, Switch principal |
| SW11_CORDOBA | Acceso | 10, 20 | Switch de acceso a usuarios y servidores |

### Switches - Mendoza
| Dispositivo | Tipo | VLANs | Función |
|-------------|------|-------|---------|
| SW_CORE_DS_MEND | Core/Distribución | 44, 55, 70 | **Root Bridge** de Mendoza, gestión WiFi |
| SW11_MENDOZA | Acceso | - | Switch de acceso principal |
| SW_CORE_DS_HEND | Servicios | 44, 45, 70 | Switch de servicios especiales |
| SW_MEST_DHCP_SERVER | Servidor | - | Switch para servidor DHCP |

### Switch Compartido
| Dispositivo | Tipo | VLANs | Función |
|-------------|------|-------|---------|
| SW_OSPF_BACKUP | Backup/Redundancia | 1000 | Red de respaldo OSPF entre los 3 sitios |

### Dispositivos Finales - Buenos Aires
| Dispositivo | IP | VLAN | Tipo |
|-------------|-----|------|------|
| PC-BS-AS | 192.168.0.x | 20 | Computadora Usuario |

### Dispositivos Finales - Córdoba
| Dispositivo | IP | VLAN | Tipo |
|-------------|-----|------|------|
| PC2-VLAN10 | 192.168.10.10 | 10 | Computadora Usuario |
| FILESERVER-INTERNO-VLAN20 | 192.168.20.10 | 20 | Servidor de Archivos |

### Dispositivos Finales - Mendoza
| Dispositivo | IP | VLAN | Tipo |
|-------------|-----|------|------|
| Access Point Mendoza | 192.168.70.3 | 70 (Mgmt) | Punto de Acceso WiFi Dual-SSID |
| Laptop-Notebook-Internos | DHCP (44.x) | 44 | Dispositivo WiFi Empleados |
| Laptop-Notebook-Externos | DHCP (55.x) | 55 | Dispositivo WiFi Invitados |
| PC-DA-MENDOZA | 192.168.2.x | - | Computadora Usuario |
| Servidor DHCP | Por definir | - | Servidor DHCP para WiFi |

### Servidores Públicos (Internet)
| Servidor | IP Privada | IP Pública (NAT) | Servicio |
|----------|------------|------------------|----------|
| Web Server | Por definir | 45.162.20.10 | HTTP/HTTPS |
| DNS Server | Por definir | 1.1.1.1 | DNS Público |
| FTP Server | Por definir | - | FTP (solo PC-BS-AS) |

---

## �📊 Esquema de VLANs

| VLAN | Nombre | Ubicación | Subred | Propósito |
|------|--------|-----------|--------|-----------|
| 10 | Red Corporativa | Córdoba | 192.168.0.0/24 ó 192.168.10.0/24 | Red Local Usuarios |
| 20 | Red Servidores | BS.AS + Córdoba | 192.168.20.0/24 | Red Local / File Server |
| 44 | WiFi Internos | Mendoza | 192.168.44.0/24 | WiFi Empleados (SSID-INTERNOS) |
| 55 | WiFi Invitados | Mendoza | 192.168.55.0/24 | WiFi Público (SSID-INVITADOS) |
| 70 | Management | Mendoza | 192.168.70.0/24 | Red de Gestión (VLAN Nativa) |
| 100 | WAN1 | ISP | 81.26.24.0/29 | Conexión Internet 1 |
| 102 | WAN2 | ISP | 68.69.85.0/24 | Conexión Internet 2 |
| 1000 | OSPF Backup | SW_OSPF_BACKUP | 172.20.10.0/29 | Red de Respaldo OSPF |

---

## 🔐 Resumen de Seguridad

### Segmentación
- ✅ VLANs separadas para diferentes departamentos
- ✅ WiFi segregado (Internos vs Invitados)
- ✅ ACL restrictiva en servidor FTP

### Alta Disponibilidad
- ✅ Dual-WAN a Internet (redundancia ISP)
- ✅ OSPF con múltiples rutas (mesh entre sitios)
- ✅ Red de respaldo OSPF (VLAN 1000)
- ✅ STP para prevenir loops en LAN

### NAT Multi-Capa
- ✅ PAT para tráfico saliente corporativo
- ✅ NAT estático para servicios públicos (Web, DNS)

---

## 📝 Notas Importantes del Profesor

> **CORRECCIÓN**: En las conexiones directas entre routers (enlaces de fibra):
> - ❌ **NO configurar subinterfaz .500**
> - ✅ **Configurar IPs directamente en interfaces físicas**
> - **Razón**: Limitación de Packet Tracer con OSPF en subinterfaces
> - **Beneficio**: Permite configurar OSPF tipo Point-to-Point correctamente

---

## 🎯 Checklist de Implementación

### Fase 1: Configuración Básica
- [ ] Configurar hostnames en todos los routers y switches
- [ ] Configurar IPs en interfaces físicas (enlaces P2P)
- [ ] Crear VLANs en switches correspondientes
- [ ] Configurar trunk/access ports

### Fase 2: Routing
- [ ] Configurar OSPF en enlaces P2P (BS.AS-Córdoba, BS.AS-Mendoza, Córdoba-Mendoza)
- [ ] Configurar subinterfaz .1000 para red de respaldo
- [ ] Configurar OSPF tipo Broadcast en VLAN 1000
- [ ] Configurar interfaces LAN como pasivas
- [ ] Configurar rutas estáticas hacia Internet
- [ ] Configurar ruteo estático entre ISPs

### Fase 3: Redundancia y Optimización
- [ ] Configurar STP en todos los switches
- [ ] Configurar Root Bridge en switches Core/Distribución
- [ ] Verificar convergencia STP

### Fase 4: Servicios WiFi
- [ ] Configurar Access Point en Mendoza (IP: 192.168.70.3)
- [ ] Configurar VLAN 70 como VLAN de Management (Nativa)
- [ ] Crear SSID-INTERNOS (VLAN 44 - 192.168.44.0/24)
- [ ] Crear SSID-INVITADOS (VLAN 55 - 192.168.55.0/24)
- [ ] Configurar DHCP para ambas redes WiFi
- [ ] Verificar aislamiento entre SSIDs

### Fase 5: NAT
- [ ] Configurar PAT en Router BS.AS (dual-WAN)
- [ ] Configurar NAT estático para Web Server (45.162.20.10)
- [ ] Configurar NAT estático para DNS Server (1.1.1.1)

### Fase 6: Seguridad
- [ ] Crear ACL para restricción FTP
- [ ] Aplicar ACL en interfaz correspondiente
- [ ] Verificar acceso desde PC-BS-AS
- [ ] Verificar bloqueo desde otros dispositivos

### Fase 7: Pruebas
- [ ] Verificar conectividad entre todos los sitios
- [ ] Verificar salida a Internet desde todos los sitios
- [ ] Verificar failover de rutas OSPF
- [ ] Verificar acceso a servicios públicos (Web/DNS)
- [ ] Verificar segregación de tráfico WiFi
- [ ] Verificar políticas de ACL

---

## 📌 Direccionamiento IP Resumido

### Redes WAN
- **Enlaces P2P**: Configurar según topología
- **VLAN 1000 (Backup)**: 172.20.10.0/29
- **ISP WAN1 (VLAN 100)**: 81.26.24.0/29
- **ISP WAN2 (VLAN 102)**: 68.69.85.0/24

### Redes LAN por Sitio

#### Buenos Aires
- **VLAN 20**: 192.168.20.0/24
- **Red General**: 192.168.0.0/24

#### Córdoba
- **VLAN 10** (Usuarios): 192.168.0.0/24 o 192.168.10.0/24
  - PC2-VLAN10: 192.168.10.10
- **VLAN 20** (Servidores): 192.168.20.0/24
  - FILESERVER-INTERNO-VLAN20: 192.168.20.10
- **Enlaces P2P Internos**: 10.10.1.0/30, 10.10.1.8/30

#### Mendoza
- **VLAN 44** (WiFi Internos): 192.168.44.0/24
  - Laptop-Notebook-Internos (DHCP)
- **VLAN 55** (WiFi Invitados): 192.168.55.0/24
  - Laptop-Notebook-Externos (DHCP)
- **VLAN 70** (Management): 192.168.70.0/24
  - Access Point: 192.168.70.3
- **Red General**: 192.168.70.0/20
- **PC Mendoza**: 192.168.2.0/4

### IPs Públicas (NAT)
- **Web Server**: 45.162.20.10
- **DNS Server**: 1.1.1.1

### Dispositivos Específicos

| Dispositivo | IP | VLAN | Ubicación |
|-------------|-----|------|-----------|
| PC-BS-AS | 192.168.0.x | 20 | Buenos Aires |
| PC2-VLAN10 | 192.168.10.10 | 10 | Córdoba |
| FILESERVER-INTERNO | 192.168.20.10 | 20 | Córdoba |
| Access Point Mendoza | 192.168.70.3 | 70 | Mendoza |
| Laptop-Notebook-Internos | DHCP | 44 | Mendoza WiFi |
| Laptop-Notebook-Externos | DHCP | 55 | Mendoza WiFi |
| PC-DA-MENDOZA | 192.168.2.x | - | Mendoza |

---

## 🚀 Tecnologías Implementadas

- **Routing Dinámico**: OSPF (Point-to-Point + Broadcast)
- **Routing Estático**: Rutas predeterminadas a Internet
- **Switching**: VLANs, Trunking, Access Ports
- **Redundancia**: STP, OSPF multi-path, Dual-WAN
- **NAT**: PAT (Port Address Translation) + NAT Estático
- **Seguridad**: ACLs, Segmentación VLAN
- **WiFi**: Dual-SSID con VLANs separadas
- **Inter-VLAN Routing**: Routing On A Stick (subinterfaz .1000)

---

**Última actualización**: Noviembre 17, 2025
**Herramienta**: Cisco Packet Tracer
