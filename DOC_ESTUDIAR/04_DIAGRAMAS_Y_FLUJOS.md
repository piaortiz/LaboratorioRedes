# 📊 DIAGRAMAS Y FLUJOS DE TRÁFICO

Este documento presenta diagramas visuales y análisis de flujos de tráfico en la red.

---

## 📖 Índice
1. [Topología Física](#topologia-fisica)
2. [Topología Lógica OSPF](#topologia-ospf)
3. [Segmentación VLANs](#segmentacion-vlans)
4. [Flujos de Tráfico](#flujos-trafico)
5. [Redundancia y Failover](#redundancia)

---

## 🔷 1. Topología Física {#topologia-fisica}

### **Diagrama de Conexiones Físicas**

```
                        INTERNET
                            |
                   [ISP INTERNACIONAL]
                    (164.25.0.1)
                            |
                      VLAN 100/101
                            |
                    [SW-MS-CORE]
                     /        \
              DNS Server    Web Server
           192.168.100.2  192.168.100.9
              (1.1.1.1)   (45.162.20.10)
                            |
                            |
                      164.25.0.0/29
                            |
                     [ISP LOCAL]
                    (164.25.0.2)
                      /        \
                WAN1 /          \ WAN2
            42.25.25.0/29   43.26.26.0/29
                  /              \
                 /                \
        [SW-BUENOSAIRES]    [SW-BUENOSAIRES]
         VLAN 100            VLAN 200
                \              /
                 \            /
              [ROUTER BUENOS AIRES]
              (1.1.1.1 - Router ID)
                  |        |
           VLAN 30|        |VLAN 1000
                  |        |
         [SW-BUENOSAIRES] [SW_OSPF_BACKUP]
              |                  |
         PC BS.AS          (Backup Path)
      192.168.30.10             |
                                |
              +-----------------+-----------------+
              |                                   |
      [ROUTER CORDOBA]                  [ROUTER MENDOZA]
      (2.2.2.2 - Router ID)            (3.3.3.3 - Router ID)
              |                                   |
        VLAN 10,20                          VLAN 44,55,70
              |                                   |
      [SW-CORE-DIS-CORD]              [SW-CORE-DIS-MEND]
              |                                   |
      [SW-ACC-CORD]                       [SW-ACC-MEND]
          /    \                              /    |    \
         /      \                            /     |     \
    PC2      File Server              PC Admin  DHCP   RADIUS
  VLAN10    192.168.20.10             VLAN70   Server  Server
                                                  |
                                            [Access Point]
                                              /        \
                                        WiFi Int.   WiFi Guest
                                        VLAN 44     VLAN 55
```

### **Enlaces P2P (Point-to-Point)**

```
Buenos Aires ←→ Córdoba
10.10.1.8/30 (Costo OSPF: 10)
BS.AS: .9  |  CORDOBA: .10

Buenos Aires ←→ Mendoza
10.10.1.0/30 (Costo OSPF: 10)
BS.AS: .1  |  MENDOZA: .2

Córdoba ←→ Mendoza
10.10.1.16/30 (Costo OSPF: 10)
CORDOBA: .17  |  MENDOZA: .18
```

### **Enlace de Backup OSPF**

```
        [SW_OSPF_BACKUP]
         VLAN 1000 (172.20.10.0/29)
              /    |    \
             /     |     \
          .1      .2      .3
       BS.AS  CORDOBA  MENDOZA
    (Costo: 50 en todas las interfaces)
```

---

## 🔷 2. Topología Lógica OSPF {#topologia-ospf}

### **Área OSPF 0 (Backbone)**

```
                    ÁREA 0
        ┌───────────────────────────┐
        │                           │
        │   [Router BS.AS]          │
        │   Router ID: 1.1.1.1      │
        │   Default Route Injector  │
        │          / \               │
        │         /   \              │
        │   Cost 10   Cost 10        │
        │       /       \            │
        │      /         \           │
        │  [CORDOBA]  [MENDOZA]      │
        │  RID: 2.2.2.2  RID: 3.3.3.3│
        │      \         /            │
        │       \       /             │
        │    Cost 10  /               │
        │         \ /                 │
        │                             │
        │   Backup Path (Cost 50):    │
        │   BS.AS ←→ CORDOBA ←→ MENDOZA│
        │   (VLAN 1000)               │
        └───────────────────────────┘
```

### **Redes Anunciadas en OSPF**

| Router | Redes Anunciadas |
|--------|------------------|
| **Buenos Aires** | 192.168.30.0/24 (LAN)<br>10.10.1.8/30 (P2P a Córdoba)<br>10.10.1.0/30 (P2P a Mendoza)<br>172.20.10.0/29 (Backup)<br>0.0.0.0/0 (Default) |
| **Córdoba** | 192.168.10.0/24 (Usuarios)<br>192.168.20.0/24 (Servidores)<br>10.10.1.8/30 (P2P a BS.AS)<br>10.10.1.16/30 (P2P a Mendoza)<br>172.20.10.0/29 (Backup) |
| **Mendoza** | 192.168.44.0/24 (WiFi Int.)<br>192.168.55.0/24 (WiFi Guest)<br>192.168.70.0/24 (Management)<br>10.10.1.0/30 (P2P a BS.AS)<br>10.10.1.16/30 (P2P a Córdoba)<br>172.20.10.0/29 (Backup) |

### **Cálculo de Rutas OSPF**

**Ejemplo: Córdoba → Buenos Aires**

**Ruta Primaria (Costo Total: 10)**
```
CORDOBA → BS.AS (directo)
Interface: Gig0/0/0
Costo: 10
```

**Ruta Alternativa 1 (Costo Total: 20)**
```
CORDOBA → MENDOZA → BS.AS
Costo: 10 + 10 = 20
```

**Ruta de Backup (Costo Total: 100)**
```
CORDOBA → VLAN 1000 → BS.AS
Costo: 50 + 50 = 100
```

**Resultado:** OSPF elige la ruta directa (menor costo)

---

## 🔷 3. Segmentación VLANs {#segmentacion-vlans}

### **Mapa de VLANs por Sitio**

#### **Buenos Aires**
```
┌─────────────────────────────────┐
│     ROUTER BUENOS AIRES         │
│  ┌─────────────────────────┐    │
│  │ VLAN 30 (LAN)           │    │
│  │ 192.168.30.0/24         │    │
│  │ Gateway: .1             │    │
│  │ NAT Inside              │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ VLAN 100 (WAN1)         │    │
│  │ 42.25.25.0/29           │    │
│  │ NAT Outside (Primary)   │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ VLAN 200 (WAN2)         │    │
│  │ 43.26.26.0/29           │    │
│  │ NAT Outside (Backup)    │    │
│  └─────────────────────────┘    │
└─────────────────────────────────┘
```

#### **Córdoba**
```
┌─────────────────────────────────┐
│      ROUTER CÓRDOBA             │
│  ┌─────────────────────────┐    │
│  │ VLAN 10 (Usuarios)      │    │
│  │ 192.168.10.0/24         │    │
│  │ Gateway: .1             │    │
│  │ Passive OSPF            │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ VLAN 20 (Servidores)    │    │
│  │ 192.168.20.0/24         │    │
│  │ Gateway: .1             │    │
│  │ ACL: FTP_BLOCK (out)    │    │
│  └─────────────────────────┘    │
└─────────────────────────────────┘
```

#### **Mendoza**
```
┌─────────────────────────────────┐
│      ROUTER MENDOZA             │
│  ┌─────────────────────────┐    │
│  │ VLAN 44 (WiFi Internos) │    │
│  │ 192.168.44.0/24         │    │
│  │ DHCP Relay: .70.10      │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ VLAN 55 (WiFi Invitados)│    │
│  │ 192.168.55.0/24         │    │
│  │ DHCP Relay: .70.10      │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ VLAN 70 (Management)    │    │
│  │ 192.168.70.0/24         │    │
│  │ VLAN Nativa             │    │
│  │ DHCP Server: .10        │    │
│  └─────────────────────────┘    │
└─────────────────────────────────┘
```

### **Matriz de Comunicación entre VLANs**

| Origen ↓ / Destino → | VLAN 10 | VLAN 20 | VLAN 30 | VLAN 44 | VLAN 55 | VLAN 70 | Internet |
|---------------------|---------|---------|---------|---------|---------|---------|----------|
| **VLAN 10** | ✅ | ❌ FTP | ✅ | ✅ | ✅ | ✅ | ✅ NAT |
| **VLAN 20** | ✅ | ❌ FTP | ✅ | ✅ | ✅ | ✅ | ✅ NAT |
| **VLAN 30** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ NAT |
| **VLAN 44** | ✅ | ❌ FTP | ✅ | ✅ | ✅ | ✅ | ✅ NAT |
| **VLAN 55** | ✅ | ❌ FTP | ✅ | ✅ | ✅ | ✅ | ✅ NAT |
| **VLAN 70** | ✅ | ❌ FTP | ✅ | ✅ | ✅ | ✅ | ✅ NAT |

**Leyenda:**
- ✅ = Permitido
- ❌ FTP = Bloqueado solo FTP al servidor 192.168.20.10
- ✅ NAT = Permitido con traducción NAT

---

## 🔷 4. Flujos de Tráfico {#flujos-trafico}

### **Flujo 1: PC Buenos Aires → Internet**

```
1. PC BS.AS (192.168.30.10)
   ↓
2. SW-BUENOSAIRES (VLAN 30)
   ↓
3. Router BS.AS (Gateway: 192.168.30.1)
   ↓
4. NAT: 192.168.30.10:50000 → 42.25.25.1:1024
   ↓
5. Router BS.AS (Gig0/1.100 - WAN1)
   ↓
6. ISP Local (42.25.25.2)
   ↓
7. ISP Internacional (164.25.0.1)
   ↓
8. INTERNET
```

**Tecnologías involucradas:**
- VLANs (segmentación)
- Router-on-a-Stick (inter-VLAN routing)
- NAT Overload (PAT)
- Rutas estáticas

---

### **Flujo 2: PC Córdoba → PC Mendoza**

```
1. PC2 Córdoba (192.168.10.5)
   ↓
2. SW-ACC-CORD (VLAN 10)
   ↓
3. SW-CORE-DIS-CORD (Trunk)
   ↓
4. Router CORDOBA (Gateway: 192.168.10.1)
   ↓
5. OSPF Lookup: Ruta a 192.168.44.0/24
   ↓
   Opción A (Costo 20): CORDOBA → BS.AS → MENDOZA
   Opción B (Costo 10): CORDOBA → MENDOZA (directo) ✅
   ↓
6. Router CORDOBA (Gig0/1/0) → 10.10.1.17
   ↓
7. Router MENDOZA (Gig0/1/0) ← 10.10.1.18
   ↓
8. Router MENDOZA (Gateway: 192.168.44.1)
   ↓
9. SW-CORE-DIS-MEND (Trunk)
   ↓
10. Access Point (VLAN 44)
    ↓
11. PC WiFi Mendoza (192.168.44.20)
```

**Tecnologías involucradas:**
- VLANs
- Spanning Tree (prevención de loops)
- OSPF (enrutamiento dinámico)
- Enlaces P2P

---

### **Flujo 3: Internet → Web Server (Servidor Público)**

```
1. Cliente Internet (cualquier IP)
   ↓
2. INTERNET
   ↓
3. ISP Internacional (164.25.0.1)
   ↓
4. NAT Estático: 45.162.20.10 → 192.168.100.9
   ↓
5. Router ISP Internacional (Gig0/1.101 - VLAN 101)
   ↓
6. SW-MS-CORE (VLAN 101)
   ↓
7. Web Server (192.168.100.9)
```

**Tecnologías involucradas:**
- NAT Estático (1:1)
- VLANs
- Rutas estáticas

---

### **Flujo 4: PC Córdoba → Servidor de Archivos (FTP Bloqueado)**

```
1. PC2 Córdoba (192.168.10.5) → FTP Request
   ↓
2. SW-ACC-CORD (VLAN 10)
   ↓
3. SW-CORE-DIS-CORD (Trunk)
   ↓
4. Router CORDOBA (Gateway: 192.168.10.1)
   ↓
5. Routing: Destino 192.168.20.10 (misma área OSPF)
   ↓
6. Router CORDOBA (Gig0/2.20 - VLAN 20)
   ↓
7. ACL FTP_BLOCK (outbound):
   - Línea 1: permit host 192.168.30.10 → NO MATCH
   - Línea 2: deny any → MATCH ❌ BLOQUEADO
   ↓
8. Paquete descartado
```

**Resultado:** FTP bloqueado por ACL

---

### **Flujo 5: PC Buenos Aires → Servidor de Archivos (FTP Permitido)**

```
1. PC BS.AS (192.168.30.10) → FTP Request
   ↓
2. SW-BUENOSAIRES (VLAN 30)
   ↓
3. Router BS.AS (Gateway: 192.168.30.1)
   ↓
4. OSPF Lookup: Ruta a 192.168.20.0/24
   ↓
5. Router BS.AS (Gig0/0/0) → 10.10.1.9
   ↓
6. Router CORDOBA (Gig0/0/0) ← 10.10.1.10
   ↓
7. Router CORDOBA (Gig0/2.20 - VLAN 20)
   ↓
8. ACL FTP_BLOCK (outbound):
   - Línea 1: permit host 192.168.30.10 → MATCH ✅ PERMITIDO
   ↓
9. SW-CORE-DIS-CORD (VLAN 20)
   ↓
10. SW-ACC-CORD (VLAN 20)
    ↓
11. ACL FTP_BLOCK (inbound en Fa0/2):
    - deny tcp 192.168.10.0 → NO MATCH
    - permit ip any any → MATCH ✅ PERMITIDO
    ↓
12. File Server (192.168.20.10)
```

**Resultado:** FTP permitido (origen autorizado)

---

### **Flujo 6: Cliente WiFi Invitado → DHCP Request**

```
1. Laptop Invitado (sin IP) → DHCP Discover (broadcast)
   ↓
2. Access Point (VLAN 55)
   ↓
3. SW-ACC-MEND (VLAN 55)
   ↓
4. SW-CORE-DIS-MEND (Trunk)
   ↓
5. Router MENDOZA (Gig0/2.55)
   ↓
6. DHCP Relay: Convierte broadcast a unicast
   Destino: 192.168.70.10
   ↓
7. Router MENDOZA (Gig0/2.70 - VLAN 70)
   ↓
8. SW-CORE-DIS-MEND (VLAN 70)
   ↓
9. SW-ACC-MEND (VLAN 70)
   ↓
10. DHCP Server (192.168.70.10)
    ↓
11. DHCP Offer → Router MENDOZA
    ↓
12. Router MENDOZA → Cliente (VLAN 55)
    ↓
13. Cliente recibe IP (ej: 192.168.55.100)
```

**Tecnologías involucradas:**
- DHCP Relay (IP Helper)
- VLANs
- Broadcast to Unicast conversion

---

## 🔷 5. Redundancia y Failover {#redundancia}

### **Escenario 1: Falla de WAN1**

**Estado Normal:**
```
Router BS.AS → WAN1 (42.25.25.0/29) → ISP Local → Internet
            ↓
         WAN2 (43.26.26.0/29) [Standby, AD=5]
```

**Tras Falla de WAN1:**
```
Router BS.AS → WAN1 [DOWN] ❌
            ↓
         WAN2 (43.26.26.0/29) [ACTIVO] ✅ → ISP Local → Internet
```

**Proceso:**
1. Interfaz WAN1 (Gig0/1.100) cae
2. Rutas estáticas con AD=1 se eliminan de la tabla de enrutamiento
3. Rutas estáticas con AD=5 (WAN2) se activan automáticamente
4. NAT cambia a interfaz Gig0/1.200
5. Tráfico fluye por WAN2

**Tiempo de convergencia:** ~1-2 segundos

---

### **Escenario 2: Falla de Enlace P2P BS.AS - Córdoba**

**Estado Normal:**
```
CORDOBA → BS.AS (directo, costo 10) ✅
       ↓
    MENDOZA → BS.AS (costo 20) [Standby]
       ↓
    VLAN 1000 (costo 100) [Standby]
```

**Tras Falla del Enlace Directo:**
```
CORDOBA → BS.AS (directo) [DOWN] ❌
       ↓
    CORDOBA → MENDOZA → BS.AS (costo 20) ✅ [ACTIVO]
       ↓
    VLAN 1000 (costo 100) [Standby]
```

**Proceso:**
1. OSPF detecta pérdida de adyacencia (Dead Timer: 40 seg)
2. OSPF recalcula rutas (algoritmo SPF)
3. Nueva mejor ruta: CORDOBA → MENDOZA → BS.AS (costo 20)
4. Tabla de enrutamiento se actualiza
5. Tráfico se redirige automáticamente

**Tiempo de convergencia:** ~40-50 segundos

---

### **Escenario 3: Falla de Todos los Enlaces P2P**

**Estado Normal:**
```
Enlaces P2P directos (costo 10) ✅
VLAN 1000 (costo 50) [Standby]
```

**Tras Falla de Todos los P2P:**
```
Enlaces P2P directos [DOWN] ❌
VLAN 1000 (costo 50) ✅ [ACTIVO]
```

**Topología Resultante:**
```
        [SW_OSPF_BACKUP]
         VLAN 1000
              |
    +---------+---------+
    |         |         |
  BS.AS   CORDOBA   MENDOZA
```

**Proceso:**
1. OSPF detecta pérdida de todas las adyacencias P2P
2. OSPF recalcula usando VLAN 1000 como único camino
3. Elección de DR/BDR en VLAN 1000:
   - DR: BS.AS (prioridad 100)
   - BDR: CORDOBA o MENDOZA (prioridad 50)
4. Tráfico fluye por VLAN 1000

**Tiempo de convergencia:** ~40-50 segundos

---

### **Escenario 4: Falla de Enlace en Capa 2 (STP)**

**Estado Normal (VLAN 10):**
```
SW-CORE-DIS-CORD (Root Bridge, prioridad 4096)
         |
         | (Forwarding)
         |
   SW-ACC-CORD
```

**Si se agrega un enlace redundante:**
```
SW-CORE-DIS-CORD (Root)
    |         |
    |         | (Blocked por STP)
    |         |
   SW-ACC-CORD
```

**Tras Falla del Enlace Principal:**
```
SW-CORE-DIS-CORD (Root)
    |         |
    X         | (Forwarding)
  [DOWN]      |
   SW-ACC-CORD
```

**Proceso:**
1. STP detecta pérdida de enlace
2. Puerto bloqueado transiciona:
   - Blocking → Listening (15 seg)
   - Listening → Learning (15 seg)
   - Learning → Forwarding
3. Tráfico fluye por nuevo enlace

**Tiempo de convergencia:** ~30-50 segundos

---

## 📊 Resumen de Tiempos de Convergencia

| Tecnología | Escenario | Tiempo de Convergencia |
|------------|-----------|------------------------|
| **Rutas Estáticas** | Falla de WAN1 | 1-2 segundos |
| **OSPF** | Falla de enlace P2P | 40-50 segundos |
| **OSPF** | Falla de todos los P2P | 40-50 segundos |
| **STP** | Falla de enlace L2 | 30-50 segundos |
| **NAT** | Cambio de interfaz | Inmediato (con rutas) |

---

## 🎯 Puntos Clave de Diseño

✅ **Múltiples niveles de redundancia:** WAN, OSPF, STP  
✅ **Convergencia automática:** Sin intervención manual  
✅ **Costos OSPF diferenciados:** Rutas preferidas vs backup  
✅ **Distancias administrativas:** Priorización de rutas estáticas  
✅ **STP por VLAN:** Optimización de rutas L2  
✅ **Failover transparente:** Usuarios no perciben cambios  

---

**Este documento proporciona una visión completa de la topología y flujos de tráfico de la red.**
