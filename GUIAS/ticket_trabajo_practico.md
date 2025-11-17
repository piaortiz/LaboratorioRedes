# Ticket de Implementación - TP Redes
**Fecha:** 17/11/2025  
**Responsable:** Equipo de Redes PIA  
**Prioridad:** Alta  
**Estado:** Fase 2 Completada - En progreso

---

## 📅 Actualización 17/11/2025 - Fase 2 Buenos Aires
**Completado:**
- Switch SW-BS-AS configurado con VLAN 20/100/200 y trunk Gi0/2 → Router BS.AS G0/1
- Router BS.AS subinterfaz G0/1.20 operativa (192.168.20.1/24)
- Conectividad LAN verificada: ping Router BS.AS ↔ PC-BS-AS exitoso
- ACL `FTP_ONLY_PC` creada (pendiente completar reglas cuando se asigne servidor)
- ISP_LOCAL validado: ruta a 192.168.20.0/24 vía 42.25.25.1 presente

**Evidencias capturadas:**
```
Router BS.AS:
- show ip interface brief: G0/1 up/up, G0/1.20 up/up (192.168.20.1)
- show ip route: 192.168.20.0/24 conectada vía G0/1.20
- ping 192.168.20.10: 100% success (5/5)

ISP_LOCAL:
- show ip route | include 192.168.20: S 192.168.20.0/24 [1/0] via 42.25.25.1
```

**Pendiente para próximas fases:**
- Configuración enlaces OSPF P2P (Fase 3)
- Rutas estáticas y NAT cuando se defina esquema WAN completo
- Completar ACL FTP con IP servidor real

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

> **Notas:** Seguir la corrección del profesor: en enlaces directos entre routers, usar únicamente interfaces físicas (sin subinterfaces .500) para OSPF point-to-point.
