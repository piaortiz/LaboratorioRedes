# Ticket de Implementación - TP Redes
**Fecha:** 19/11/2025  
**Responsable:** Equipo de Redes PIA  
**Prioridad:** Alta  
**Estado:** ✅ Fases 1 y 2 Completadas - 33% Progreso Total

---

## 📅 Actualización 19/11/2025 - Fases 1 y 2 COMPLETADAS ✅

### **Fase 1: ISP_LOCAL + Internet Cloud** ✅
**Completado:**
- ISP_LOCAL configurado con interfaces G0/0 (Internet), G0/1 (WAN1), G0/2 (WAN2)
- ISP_INTERNACIONAL configurado (164.25.0.1/29)
- SW_MS_CORE con VLANs 100 (DNS) y 101 (WEB)
- Servidores DNS (192.168.100.2) y WEB (192.168.100.9) operativos con servicios activos
- Rutas estáticas en ISP_LOCAL hacia 192.168.30.0/24 (red Buenos Aires)
- Conectividad Internet verificada: ISP_LOCAL ↔ ISP_INTERNACIONAL

### **Fase 2: Segmento Buenos Aires** ✅
**Completado:**
- **PC-BS-AS** configurada: IP 192.168.30.10/24, gateway 192.168.30.1, DNS 1.1.1.1
- **Switch SW-BS-AS** con VLANs 30/100/200:
  - Fa0/2: access VLAN 30 (PC-BS-AS)
  - Gi0/2: trunk hacia Router BS.AS (VLANs 30,100,200)
  - Gi0/1: access VLAN 100 (WAN1 → ISP_LOCAL)
  - Fa0/24: access VLAN 200 (WAN2 → ISP_LOCAL)
- **Router BS.AS** con Router-on-a-Stick:
  - G0/1.30: 192.168.30.1/24 (LAN, ip nat inside)
  - G0/1.100: 42.25.25.1/29 (WAN1, ip nat outside)
  - G0/1.200: 43.26.26.1/29 (WAN2, ip nat outside)
- **NAT overload** configurado en ambas salidas WAN (doble redundancia)
- **Rutas estáticas:**
  - Hacia servidores: 192.168.100.0/29 vía 42.25.25.2 y 43.26.26.2
  - Ruta por defecto: 0.0.0.0/0 vía 42.25.25.2 y 43.26.26.2
- **Rutas de retorno en ISP_INTERNACIONAL:**
  - ip route 42.25.25.0 255.255.255.248 164.25.0.2
  - ip route 43.26.26.0 255.255.255.248 164.25.0.2
- **ACL FTP_ONLY_PC** creada (pendiente reglas finales)

**Evidencias capturadas:**
```
PC-BS-AS:
- ipconfig: 192.168.30.10/24, gateway 192.168.30.1
- ping 192.168.30.1: ✅ SUCCESS
- ping 192.168.100.2 (DNS): ✅ SUCCESS (con NAT)
- ping 192.168.100.9 (WEB): ✅ SUCCESS (con NAT)
- ping 164.25.0.1 (Internet): ✅ SUCCESS (con NAT)

Router BS.AS:
- show ip interface brief: G0/1 up/up, subinterfaces .30/.100/.200 todas up/up
- show ip route: Rutas estáticas a 192.168.100.0/29 y 0.0.0.0/0 presentes
- ping 192.168.30.10: ✅ SUCCESS (5/5)
- ping 42.25.25.2: ✅ SUCCESS (WAN1)
- ping 43.26.26.2: ✅ SUCCESS (WAN2)
- ping 164.25.0.1: ✅ SUCCESS (Internet)
- show ip nat translations: Traducciones activas verificadas
- show ip nat statistics: Overload en ambas interfaces confirmado

Switch SW-BS-AS:
- show vlan brief: VLANs 30, 100, 200 presentes
- show interfaces trunk: Gi0/2 trunk activo (VLANs 30,100,200)

ISP_LOCAL:
- show ip route: Ruta hacia 192.168.30.0/24 presente
- ping 164.25.0.1: ✅ SUCCESS

ISP_INTERNACIONAL:
- show ip route: Rutas de retorno 42.25.25.0/29 y 43.26.26.0/29 vía 164.25.0.2
- Tráfico NAT bidireccional funcionando correctamente
```

**Pendiente para próximas fases:**
- Configuración enlaces OSPF P2P (Fase 3)
- Configuración sitios Córdoba y Mendoza
- Completar ACL FTP con IP servidor real
- VLAN 1000 y SW_OSPF_BACKUP (Fase 4)

---

## 1. Resumen Ejecutivo
Implementar la red corporativa multi-sitio (Buenos Aires, Córdoba y Mendoza) con conectividad redundante, segmentación VLAN, OSPF, WiFi dual-SSID, NAT y políticas de seguridad ACL, siguiendo los requerimientos del trabajo práctico y las notas del profesor.

---

## 2. Alcance
- Configuración completa de routers BS.AS, Córdoba y Mendoza
- Configuración de switches de core, distribución y acceso en los tres sitios
- Configuración de ISP_LOCAL e ISP_INTERNACIONAL para salida a Internet
- Implementación del Access Point WiFi en Mendoza
- Puesta en marcha de servicios NAT, DHCP, ACL y pruebas finales

Fuera de alcance: adquisición de hardware, monitoreo continuo, documentación de usuario final.

---

## 3. Prerrequisitos
1. Packet Tracer con topología actualizada (archivo PKT provisto)
2. Acceso administrativo a todos los routers, switches y servidores
3. Direccionamiento IP aprobado (ver `ANALISIS_PROYECTO.md`)
4. Credenciales para ISP_LOCAL e ISP_INTERNACIONAL
5. Tabla de VLANs y dispositivos validada por el equipo docente

---

## 4. Inventario de Dispositivos
| Sitio/Elemento | Dispositivos a tocar |
|----------------|----------------------|
| Buenos Aires | Router BS.AS, SW_DIST_IB, SW_CORP_BA_COLP, PC-BS-AS |
| Córdoba | Router Córdoba, SW_CORE_DS_CORO, SW11_CORDOBA, PC2-VLAN10, FILESERVER-INTERNO |
| Mendoza | Router Mendoza, SW_CORE_DS_MEND, SW11_MENDOZA, SW_CORE_DS_HEND, SW_MEST_DHCP_SERVER, AP Mendoza, laptops WiFi, PC-DA-MENDOZA |
| Enlaces P2P/Backup | SW_OSPF_BACKUP, enlaces fibra BS.AS-CBA-MDZ |
| ISP/Servicios | Router ISP_LOCAL, Router ISP_INTERNACIONAL, Servidores Web/DNS/FTP |

---

## 5. Procedimiento Paso a Paso
Cada fase incluye objetivo, dispositivos involucrados y acciones detalladas.

### Fase 1: Preparación y Configuración Básica
- **Objetivo:** Dejar todos los equipos identificados y con interfaces físicas operativas según topología.
- **Dispositivos:** Todos los routers y switches.
- **Acciones:**
  1. Asignar hostnames y banners de seguridad.
  2. Configurar IPs directamente en interfaces físicas de enlaces P2P (sin subinterfaces .500).
  3. Crear VLANs requeridas en cada switch y etiquetar puertos trunk/access.
  4. Verificar conectividad de capa 2 con `show vlan`, `show interfaces trunk`.

#### Tickets Detallados - Fase 1

| ID | Objetivo | Dispositivos | Pasos | Validación |
|----|----------|--------------|-------|------------|
| **F1-01** | Normalizar identidad de equipos (hostname y banners) | Router BS.AS, Router Córdoba, Router Mendoza, SW_DIST_IB, SW_CORE_DS_CORO, SW_CORE_DS_MEND, SW11_CORDOBA, SW11_MENDOZA, SW_CORE_DS_HEND, SW_MEST_DHCP_SERVER, SW_CORP_BA_COLP, SW_OSPF_BACKUP | 1) Conectarse por consola/SSH. 2) Ejecutar `configure terminal`. 3) Asignar hostname siguiendo convención (`hostname R-BSAS`, etc.). 4) Configurar banners `banner motd` con texto de aviso. 5) Guardar con `wr mem`. | `show running-config | include hostname` y `show banner motd` en cada equipo. |
| **F1-02** | Configurar interfaces físicas P2P según plan de direccionamiento | Router BS.AS, Router Córdoba, Router Mendoza | 1) Identificar interfaces serie/Gigabit que conectan a cada enlace. 2) En `conf t`, aplicar IP y máscara correspondientes (ej: `interface g0/1`, `ip address 10.10.1.1 255.255.255.252`). 3) Habilitar `no shutdown`. 4) Repetir para cada extremo respetando nota del profesor (sin subinterfaces .500). 5) Documentar descripción (`description P2P BSAS-CBA`). | `show ip interface brief` debe mostrar interfaces `up/up`. `ping` extremo contrario para cada enlace. |
| **F1-03** | Implementar VLANs y puertos en switches de cada sitio | SW_DIST_IB, SW_CORP_BA_COLP, SW_CORE_DS_CORO, SW11_CORDOBA, SW_CORE_DS_MEND, SW11_MENDOZA, SW_CORE_DS_HEND, SW_MEST_DHCP_SERVER | 1) Crear VLANs con `vlan <id>` y nombrarlas (`name WIFI_INTERNOS`). 2) Configurar puertos de acceso (`interface fa0/1`, `switchport mode access`, `switchport access vlan 44`). 3) Configurar puertos trunk (`switchport mode trunk`, `switchport trunk allowed vlan ...`, `switchport trunk native vlan 70` donde aplique). 4) Guardar configuración. | `show vlan brief` para verificar creación y asignación. `show interfaces trunk` para confirmar etiquetado. |
| **F1-04** | Validar capa 2 y documentar resultados | Todos los switches | 1) Correr `show mac address-table` para detectar aprendizaje. 2) Ejecutar `ping` entre hosts de misma VLAN para confirmar conectividad. 3) Registrar capturas de `show vlan` y `show interfaces trunk`. 4) Actualizar bitácora del ticket con resultados y anomalías. | Evidencias guardadas, pings exitosos y tablas coherentes. |

> **Nota Fase 1:** Completar los tickets en orden. No avanzar a fases superiores hasta que las verificaciones estén documentadas.

### Fase 2: Routing OSPF
- **Objetivo:** Establecer vecindades OSPF y propagar redes LAN.
- **Dispositivos:** Routers BS.AS, Córdoba, Mendoza.
- **Acciones:**
  1. Configurar OSPF área 0 en cada router.
  2. Declarar enlaces P2P como red tipo point-to-point.
  3. Configurar subinterfaz `.1000` en cada router para VLAN 1000 (Routing on a Stick) y unirlos al SW_OSPF_BACKUP como red broadcast.
  4. Marcar interfaces LAN como `passive-interface`.
  5. Validar con `show ip ospf neighbor` y `show ip route`.

### Fase 3: Routing Estático hacia Internet
- **Objetivo:** Garantizar salida redundante a Internet y comunicación entre ISPs.
- **Dispositivos:** Router BS.AS, ISP_LOCAL, ISP_INTERNACIONAL.
- **Acciones:**
  1. En Router BS.AS, crear dos rutas estáticas por defecto apuntando a cada interfaz WAN hacia ISP_LOCAL.
  2. En ISP_LOCAL e ISP_INTERNACIONAL, configurar rutas específicas permitiendo retorno hacia redes corporativas.
  3. Probar reachability con `ping` y `traceroute` desde BS.AS hacia IPs públicas simuladas.

### Fase 4: Spanning Tree Protocol (STP)
- **Objetivo:** Evitar loops y definir root bridge por sitio.
- **Dispositivos:** Todos los switches de LAN interna.
- **Acciones:**
  1. En cada LAN, establecer prioridad inferior en switches core (`SW_CORE_DS_CORO` y `SW_CORE_DS_MEND`) y en BA (`SW_DIST_IB`).
  2. Configurar modo Rapid-PVST+ si está disponible.
  3. Ajustar `portfast` y `bpduguard` en puertos de acceso.
  4. Verificar estado con `show spanning-tree`.

### Fase 5: WiFi y DHCP Mendoza
- **Objetivo:** Propagar SSID internos e invitados con gestión separada.
- **Dispositivos:** Router Mendoza, SW_CORE_DS_MEND, AP Mendoza, Servidor DHCP.
- **Acciones:**
  1. Configurar VLAN 70 como nativa/management en switches y AP (IP 192.168.70.3).
  2. En el AP, crear SSID `SSID-INTERNOS` (VLAN 44) y `SSID-INVITADOS` (VLAN 55) con seguridad WPA2.
  3. Configurar scopes DHCP para VLAN 44 y 55 en el servidor correspondiente.
  4. Validar con laptops que cada SSID obtenga IP correcta y haya aislamiento entre VLANs.

### Fase 6: NAT
- **Objetivo:** Traducir tráfico saliente y publicar servicios públicos.
- **Dispositivos:** Router BS.AS, Router de Servicios (donde residan Web/DNS).
- **Acciones:**
  1. Configurar PAT en Router BS.AS usando ambas interfaces WAN como overload (política por interfaz).
  2. Crear NAT estático para Web Server → 45.162.20.10 y DNS Server → 1.1.1.1.
  3. Documentar IPs internas utilizadas y asegurar rutas de retorno.
  4. Probar acceso externo simulando clientes desde ISP_LOCAL.

### Fase 7: ACL de Seguridad
- **Objetivo:** Restringir acceso al servidor FTP.
- **Dispositivos:** Router donde resida el servidor FTP (probablemente Córdoba) y Router BS.AS.
- **Acciones:**
  1. Crear ACL extendida permitiendo únicamente a PC-BS-AS (192.168.0.0/24) hacia IP del FTP en puerto 21.
  2. Denegar el resto y aplicar ACL en interfaz de entrada adecuada.
  3. Validar con pruebas desde PC autorizado y desde otros dispositivos.

### Fase 8: Pruebas Integrales y Cierre
- **Objetivo:** Confirmar cumplimiento total de requerimientos.
- **Dispositivos:** Todos.
- **Acciones:**
  1. Ejecutar plan de pruebas: conectividad inter-sitio, acceso Internet, failover OSPF, WiFi, NAT, ACL.
  2. Registrar resultados y capturas de comandos clave (`show run`, `show ip route`).
  3. Actualizar `ANALISIS_PROYECTO.md` con cualquier desviación.
  4. Validar con el profesor y cerrar ticket.

---

## 6. Riesgos y Mitigaciones
| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Interfaces P2P mal configuradas | Vecindad OSPF caída | Verificar direcciones antes de activar OSPF |
| Root Bridge incorrecto | Loops/STP lento | Forzar prioridades en switches core |
| NAT inconsistente entre WANs | Pérdida de acceso a Internet | Probar cada interfaz con `show ip nat translations` |
| DHCP WiFi mezclado | Dispositivos en VLAN equivocada | Revisar etiquetado trunk y scopes |

---

## 7. Entregables
1. Archivo PKT actualizado con configuraciones finales
2. Capturas de comandos de verificación (OSPF, STP, NAT, ACL)
3. Registro de pruebas exitosas
4. Actualización del documento `ANALISIS_PROYECTO.md` si corresponde

---

## 8. Contactos
- **Equipo de Redes PIA:** redes@pia.edu
- **Profesor Revisor:** contacto docente según aula virtual

---

## 9. Resumen de Progreso del Proyecto

### Estado Global: 33% Completado (2/6 fases principales)

| Fase | Estado | Fecha Completado | Notas |
|------|--------|------------------|-------|
| **Fase 1: ISP + Internet** | ✅ COMPLETA | 19/11/2025 | ISP_LOCAL, ISP_INTERNACIONAL y servidores operativos |
| **Fase 2: Buenos Aires** | ✅ COMPLETA | 19/11/2025 | Segmento completo con LAN, WAN doble, NAT y conectividad total |
| **Fase 3: OSPF P2P** | ⏳ Pendiente | - | Requiere configuración Córdoba y Mendoza |
| **Fase 4: VLAN 1000 Backup** | ⏳ Pendiente | - | Depende de Fase 3 |
| **Fase 5: WiFi/STP** | ⏳ Pendiente | - | Configuración Mendoza WiFi |
| **Fase 6: NAT Servicios** | ⏳ Pendiente | - | NAT estático para servicios públicos |

### Logros Destacados:
✅ **Red Buenos Aires totalmente funcional** con Router-on-a-Stick (VLAN 30)  
✅ **NAT overload con doble salida WAN** (42.25.25.1 y 43.26.26.1)  
✅ **Rutas de retorno configuradas en ISP_INTERNACIONAL** para tráfico NAT bidireccional  
✅ **Conectividad completa:** PC-BS-AS → Internet, servidores DNS/WEB  
✅ **Infraestructura ISP lista** para recibir tráfico de todos los sitios  

### Próximo Hito:
🎯 **Fase 3:** Configurar sitios Córdoba y Mendoza, establecer enlaces P2P y activar OSPF área 0

---

> **Notas:** Seguir la corrección del profesor: en enlaces directos entre routers, usar únicamente interfaces físicas (sin subinterfaces .500) para OSPF point-to-point.
