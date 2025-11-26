# 📊 RESUMEN EJECUTIVO - INFRAESTRUCTURA DE RED EMPRESARIAL

## 🎯 Descripción General del Proyecto

Esta documentación describe una **infraestructura de red empresarial multi-sitio** que conecta tres ubicaciones principales (Buenos Aires, Córdoba y Mendoza) con conectividad a Internet a través de ISPs redundantes.

---

## 🏢 Arquitectura de la Red

### **Topología General**
La red implementa una arquitectura **jerárquica de tres capas** con los siguientes componentes:

```
INTERNET (ISP Internacional)
         ↓
    ISP Local (Redundancia)
         ↓
    Buenos Aires (Sede Central)
       /   |   \
      /    |    \
 Córdoba  OSPF  Mendoza
          Backup
```

### **Sitios y Funciones**

| Sitio | Función Principal | VLANs | Dispositivos |
|-------|------------------|-------|--------------|
| **Buenos Aires** | Sede central, gateway a Internet | VLAN 30 (LAN), 100/200 (WAN) | Router BS.AS, SW-BUENOSAIRES |
| **Córdoba** | Sucursal con usuarios y servidores | VLAN 10 (Usuarios), 20 (Servidores) | Router CORDOBA, SW-CORE-DIS-CORD, SW-ACC-CORD |
| **Mendoza** | Sucursal con WiFi y gestión | VLAN 44 (WiFi Internos), 55 (WiFi Invitados), 70 (Management) | Router MENDOZA, SW-CORE-DIS-MEND, SW-ACC-MEND |

---

## 🔧 Tecnologías Implementadas

### **1. Enrutamiento Dinámico - OSPF**
- **Protocolo:** OSPF (Open Shortest Path First) - Área 0 (Backbone)
- **Router IDs:**
  - Buenos Aires: `1.1.1.1`
  - Córdoba: `2.2.2.2`
  - Mendoza: `3.3.3.3`
- **Enlaces P2P (Point-to-Point):**
  - BS.AS ↔ Córdoba: `10.10.1.8/30`
  - BS.AS ↔ Mendoza: `10.10.1.0/30`
  - Córdoba ↔ Mendoza: `10.10.1.16/30`
- **Ruta de Backup:** VLAN 1000 (`172.20.10.0/29`) con costo OSPF elevado (50)

### **2. Segmentación de Red - VLANs**

#### **Buenos Aires**
- **VLAN 30:** LAN Buenos Aires (`192.168.30.0/24`)
- **VLAN 100:** WAN1 a ISP Local (`42.25.25.0/29`)
- **VLAN 200:** WAN2 a ISP Local (`43.26.26.0/29`)

#### **Córdoba**
- **VLAN 10:** Usuarios (`192.168.10.0/24`)
- **VLAN 20:** Servidores (`192.168.20.0/24`)

#### **Mendoza**
- **VLAN 44:** WiFi Internos (`192.168.44.0/24`)
- **VLAN 55:** WiFi Invitados (`192.168.55.0/24`)
- **VLAN 70:** Management (VLAN Nativa) (`192.168.70.0/24`)

#### **Servidores Externos**
- **VLAN 100:** DNS Server (`192.168.100.0/29`)
- **VLAN 101:** Web Server (`192.168.100.8/29`)

### **3. NAT (Network Address Translation)**
- **NAT Overload (PAT)** configurado en Router Buenos Aires
- **Dos interfaces de salida redundantes:**
  - Primaria: `GigabitEthernet0/1.100` (WAN1)
  - Secundaria: `GigabitEthernet0/1.200` (WAN2)
- **ACLs para NAT:**
  - ACL 10: NAT por WAN1
  - ACL 11: NAT por WAN2
- **Redes traducidas:** Todas las LANs internas (30, 10, 20, 44, 55, 70)

### **4. NAT Estático (Servidores Públicos)**
Configurado en ISP Internacional:
- **DNS Server:** `192.168.100.2` → `1.1.1.1` (IP pública)
- **Web Server:** `192.168.100.9` → `45.162.20.10` (IP pública)

### **5. Seguridad - ACLs (Access Control Lists)**

#### **Bloqueo de FTP al Servidor de Archivos (192.168.20.10)**

**En Router Córdoba:**
```
ip access-list extended FTP_BLOCK
  permit ip host 192.168.30.10 host 192.168.20.10  ! Solo PC BS.AS puede acceder
  deny ip any host 192.168.20.10                    ! Bloquear todo lo demás
  permit ip any any                                  ! Permitir resto del tráfico
```

**En Router Córdoba (P2P):**
```
ip access-list extended FTP_BLOCK_P2P
  deny tcp 192.168.44.0 0.0.0.255 host 192.168.20.10 eq ftp
  deny tcp 192.168.55.0 0.0.0.255 host 192.168.20.10 eq ftp
  deny tcp 192.168.70.0 0.0.0.255 host 192.168.20.10 eq ftp
  permit ip any any
```

**En Switch ACC-CORD:**
```
ip access-list extended FTP_BLOCK
  deny tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq ftp
  deny tcp 192.168.20.0 0.0.0.255 host 192.168.20.10 eq ftp
  deny tcp 192.168.44.0 0.0.0.255 host 192.168.20.10 eq ftp
  deny tcp 192.168.55.0 0.0.0.255 host 192.168.20.10 eq ftp
  deny tcp 192.168.70.0 0.0.0.255 host 192.168.20.10 eq ftp
  permit ip any any
```

### **6. Redundancia y Alta Disponibilidad**

#### **Redundancia de Enlaces WAN**
- Dos enlaces independientes a ISP Local
- Rutas estáticas con métricas administrativas diferentes:
  - WAN1: Métrica por defecto (1)
  - WAN2: Métrica 5 (backup)

#### **Redundancia OSPF**
- Enlaces P2P directos entre routers (costo 10)
- Enlace de backup por VLAN 1000 (costo 50)
- Prioridades OSPF configuradas:
  - Buenos Aires: 100 (DR - Designated Router)
  - Córdoba: 50
  - Mendoza: 50

#### **Spanning Tree Protocol (STP)**
- **Modo:** PVST (Per-VLAN Spanning Tree)
- **Prioridades configuradas:**
  - SW-CORE-DIS-CORD: 4096 (Root Bridge para VLANs 10, 20)
  - SW-CORE-DIS-MEND: 24576 (Root Bridge para VLANs 44, 55, 70)
  - SW_OSPF_BACKUP: 4096 (Root Bridge para VLAN 1000)
  - SW-ACC-MEND: 49152 (Non-root)
- **PortFast:** Habilitado en puertos de acceso para usuarios

### **7. DHCP (Dynamic Host Configuration Protocol)**
- **DHCP Server:** Ubicado en Mendoza (`192.168.70.10`)
- **DHCP Relay (IP Helper):** Configurado en Router Mendoza
  - `ip helper-address 192.168.70.10` en VLAN 44 y 55
- **Propósito:** Asignación dinámica de IPs para WiFi Internos e Invitados

---

## 📡 Direccionamiento IP

### **Redes LAN (Usuarios)**
| Red | VLAN | Gateway | Ubicación | Propósito |
|-----|------|---------|-----------|-----------|
| 192.168.30.0/24 | 30 | .1 | Buenos Aires | LAN principal |
| 192.168.10.0/24 | 10 | .1 | Córdoba | Usuarios |
| 192.168.20.0/24 | 20 | .1 | Córdoba | Servidores |
| 192.168.44.0/24 | 44 | .1 | Mendoza | WiFi Internos |
| 192.168.55.0/24 | 55 | .1 | Mendoza | WiFi Invitados |
| 192.168.70.0/24 | 70 | .1 | Mendoza | Management |

### **Redes WAN**
| Red | Propósito | Dispositivos |
|-----|-----------|--------------|
| 42.25.25.0/29 | WAN1 BS.AS - ISP Local | .1 (BS.AS), .2 (ISP) |
| 43.26.26.0/29 | WAN2 BS.AS - ISP Local | .1 (BS.AS), .2 (ISP) |
| 164.25.0.0/29 | ISP Local - ISP Internacional | .1 (ISP Int), .2 (ISP Local) |

### **Redes P2P (OSPF)**
| Red | Enlace | IPs |
|-----|--------|-----|
| 10.10.1.0/30 | BS.AS - Mendoza | .1 (Mendoza), .2 (BS.AS) |
| 10.10.1.8/30 | BS.AS - Córdoba | .9 (BS.AS), .10 (Córdoba) |
| 10.10.1.16/30 | Córdoba - Mendoza | .17 (Córdoba), .18 (Mendoza) |
| 172.20.10.0/29 | OSPF Backup (VLAN 1000) | .1 (BS.AS), .2 (Córdoba), .3 (Mendoza) |

### **Servidores Públicos**
| Servidor | IP Privada | IP Pública | VLAN |
|----------|------------|------------|------|
| DNS | 192.168.100.2 | 1.1.1.1 | 100 |
| Web | 192.168.100.9 | 45.162.20.10 | 101 |

---

## 🔐 Características de Seguridad

### **1. Control de Acceso**
- ✅ ACLs extendidas para bloquear FTP a servidor de archivos
- ✅ Solo PC de Buenos Aires (192.168.30.10) puede acceder al servidor de archivos
- ✅ WiFi Invitados (VLAN 55) segregado de recursos internos

### **2. Gestión**
- ✅ VLAN de Management dedicada (VLAN 70)
- ✅ Banners de advertencia en switches críticos
- ✅ Timeouts de sesión configurados (30 minutos)
- ✅ Logging sincronizado en consola

### **3. Redundancia**
- ✅ Doble enlace WAN con failover automático
- ✅ Múltiples rutas OSPF con costos diferenciados
- ✅ STP para prevenir loops en capa 2

---

## 📈 Flujo de Tráfico

### **Tráfico Saliente a Internet**
```
PC Interno → Gateway VLAN → OSPF → Router BS.AS → NAT → ISP Local → ISP Internacional → Internet
```

### **Tráfico entre Sucursales**
```
PC Córdoba → OSPF (Ruta directa o vía BS.AS) → PC Mendoza
```

### **Acceso a Servidores Públicos desde Internet**
```
Internet → ISP Internacional → NAT Estático → Servidor (DNS/Web)
```

---

## 🎓 Conceptos Clave Implementados

1. **Enrutamiento Dinámico (OSPF):** Convergencia automática ante fallas
2. **VLANs y Trunking:** Segmentación lógica de la red
3. **NAT/PAT:** Conservación de direcciones IP públicas
4. **ACLs:** Control de acceso granular
5. **Redundancia:** Alta disponibilidad mediante múltiples rutas
6. **STP:** Prevención de loops en capa 2
7. **DHCP Relay:** Centralización de servicios DHCP
8. **Subneteo:** Uso eficiente del espacio de direcciones

---

## 📊 Inventario de Dispositivos

| Dispositivo | Modelo | Función | Ubicación |
|-------------|--------|---------|-----------|
| Router BS.AS | Cisco 2911 | Gateway principal, NAT | Buenos Aires |
| Router CORDOBA | Cisco 2911 | Router sucursal, ACLs | Córdoba |
| Router MENDOZA | Cisco 2911 | Router sucursal, DHCP Relay | Mendoza |
| ISP_LOCAL | Cisco 2911 | Proveedor de Internet local | Externo |
| ISP_INTERNACIONAL | Cisco 2911 | Proveedor de Internet, NAT estático | Externo |
| SW-BUENOSAIRES | Catalyst 2960 | Switch de acceso | Buenos Aires |
| SW-CORE-DIS-CORD | Catalyst 2960 | Switch de distribución | Córdoba |
| SW-ACC-CORD | Catalyst 2960 | Switch de acceso | Córdoba |
| SW-CORE-DIS-MEND | Catalyst 2960 | Switch de distribución | Mendoza |
| SW-ACC-MEND | Catalyst 2960 | Switch de acceso | Mendoza |
| SW_OSPF_BACKUP | Catalyst 2960 | Switch para ruta de backup | Compartido |
| SW-MS-CORE | Catalyst 2960 | Switch para servidores públicos | Externo |

---

## ✅ Cumplimiento de Requerimientos

### **Implementado Completamente:**
- ✅ Topología multi-sitio con OSPF
- ✅ Redundancia de enlaces WAN
- ✅ Segmentación por VLANs
- ✅ NAT/PAT para salida a Internet
- ✅ NAT estático para servidores públicos
- ✅ ACLs para control de acceso FTP
- ✅ DHCP Relay para WiFi
- ✅ STP con prioridades configuradas
- ✅ Ruta de backup OSPF
- ✅ Passive interfaces en OSPF
- ✅ Default route injection en OSPF

---

## 🎯 Puntos Destacados para Presentación

1. **Escalabilidad:** La red puede crecer agregando nuevos sitios a OSPF
2. **Resiliencia:** Múltiples niveles de redundancia (WAN, OSPF, STP)
3. **Seguridad:** Control granular de acceso mediante ACLs
4. **Eficiencia:** Uso de OSPF para enrutamiento dinámico y convergencia rápida
5. **Gestión:** VLAN dedicada para administración de red
6. **Servicios:** DHCP centralizado con relay agents

---

## 📚 Documentos Relacionados

- `02_CONCEPTOS_TEORICOS.md` - Explicación detallada de tecnologías
- `03_ANALISIS_POR_DISPOSITIVO.md` - Configuración detallada de cada equipo
- `04_DIAGRAMAS_RED.md` - Diagramas lógicos y físicos
- `05_FLUJOS_TRAFICO.md` - Análisis de flujos de datos
- `06_GUIA_PRESENTACION.md` - Guía para explicar al profesor

---

**Fecha de Documentación:** Noviembre 2025  
**Versión:** 1.0  
**Estado:** Producción
