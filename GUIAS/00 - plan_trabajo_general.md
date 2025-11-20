# Plan de Trabajo Global TP Redes
**Fecha:** 19/11/2025 - ACTUALIZADO CON NUEVOS REQUERIMIENTOS  
**Objetivo:** Ejecutar el trabajo práctico completo siguiendo el orden óptimo sugerido, con estimación de esfuerzo y checklist por fase. Los tiempos son aproximados suponiendo dedicación continua en Cisco Packet Tracer.

**⚠️ CAMBIOS CRÍTICOS:**
1. Buenos Aires usa **VLAN 30** (red 192.168.30.0/24) en lugar de VLAN 20
2. Enlaces P2P entre routers: **interfaces físicas directas** (NO subinterfaces .500)
3. Rutas estáticas con **métricas diferentes** para evitar ECMP

**Referencias:**
- `NUEVOSREQUERIMIENTOS`: Documento oficial con cambios del profesor
- `Analisis y requisitos/CAMBIOS_CRITICOS_PROFESOR.md`: Análisis detallado de cambios

---

## Resumen por Fases
| Fase | Alcance | Estimación | Dependencias | Estado (20/11) |
|------|---------|------------|--------------|----------------|
| 1 | ISP_LOCAL + Internet Cloud | 1-2 h | Ninguna | ✅ **COMPLETA** |
| 2 | Router y Switch BS.AS (VLAN 30 + trunk) | 3-4 h | Fase 1 | ✅ **COMPLETA** |
| 3 | Enlaces P2P y OSPF entre sitios | 4-6 h | Fases 1-2 (BS.AS operativo) | ✅ **COMPLETA** |
| 4A | VLAN 1000 / SW_OSPF_BACKUP (Broadcast OSPF) | 1-2 h | Fase 3 | ✅ **COMPLETA** |
| 4B | Segmento Córdoba Local (VLANs 10, 20) | 2-3 h | Fase 3 | ✅ **COMPLETA** (20/11) |
| 5 | STP, WiFi y servicios locales (SSID, root bridges) | 2-3 h | Fases 2-4 | ⏳ Pendiente |
| 6 | NAT Servicios Internet + pruebas integrales + documentación | 2 h | Todas las anteriores | ⏳ Pendiente |

Total estimado: **15-21 horas** según nivel de detalle de las pruebas y captura de evidencias.

**Progreso actual: 75% - Fases 1, 2, 3, 4A y 4B completadas (20/11/2025)**

---

## 🎉 Logros Recientes (19/11/2025)

### ✅ Infraestructura Internet Operativa (Fase 1)
- ISP_LOCAL y ISP_INTERNACIONAL configurados y conectados
- Servidores DNS (192.168.100.2) y WEB (192.168.100.9) activos
- Rutas estáticas hacia red Buenos Aires (192.168.30.0/24)
- Conectividad Internet verificada

### ✅ Segmento Buenos Aires Completo (Fase 2)
- **PC-BS-AS:** Configurada en VLAN 30 (192.168.30.10/24)
- **Switch SW-BS-AS:** VLANs 30/100/200 con trunk operativo
- **Router BS.AS:** Router-on-a-Stick con 3 subinterfaces:
  - G0/1.30: LAN (192.168.30.1/24)
  - G0/1.100: WAN1 (42.25.25.1/29)
  - G0/1.200: WAN2 (43.26.26.1/29)
- **NAT overload:** Doble salida WAN configurada y funcional
- **Rutas de retorno:** Configuradas en ISP_INTERNACIONAL para tráfico NAT bidireccional
- **Conectividad total verificada:**
  - ✅ PC-BS-AS → Gateway (192.168.30.1)
  - ✅ PC-BS-AS → Servidores DNS/WEB (con NAT)
  - ✅ PC-BS-AS → Internet (164.25.0.1 con NAT)
  - ✅ Router BS.AS → ISP_LOCAL (ambos enlaces WAN)

### 📋 Documentación Actualizada
- Guía 01: Segmento WAN ✅
- Guía 02: Segmento Buenos Aires ✅
- Tickets de trabajo actualizados con evidencias
- Plan de trabajo general actualizado

### ✅ Red OSPF Operativa (Fases 3 y 4A)
- **Enlaces P2P configurados:** BS.AS ↔ CÓRDOBA, BS.AS ↔ MENDOZA (CÓRDOBA ↔ MENDOZA con issue de cable)
- **Red de backup VLAN 1000:** Operativa con conectividad completa (172.20.10.0/29)
- **Switch SW_OSPF_BACKUP:** Configurado con trunk en native VLAN 1 (crítico)
- **Adyacencias OSPF establecidas:**
  - BS.AS: 4 vecinos (2 P2P + 2 backup)
  - CÓRDOBA: 3 vecinos (1 P2P + 2 backup)
  - MENDOZA: 3 vecinos (1 P2P + 2 backup)
- **Rutas OSPF:** Todas las LANs visibles desde todos los routers
- **Default route:** Propagada desde BS.AS a CÓRDOBA y MENDOZA

### ✅ Segmento Córdoba Completo (Fase 4B - 20/11/2025)
- **Router CÓRDOBA:** Configurado con router-on-a-stick en Gig0/2
  - Gig0/2.10: Gateway VLAN 10 (192.168.10.1/24)
  - Gig0/2.20: Gateway VLAN 20 (192.168.20.1/24)
  - Gig0/0.1000: Backup OSPF (172.20.10.2/29)
  - Gig0/0/0: P2P a BS.AS (10.10.1.10/30)
  - Interfaces pasivas configuradas en OSPF
- **Switch DIS-CORD:** Core/Distribución operativo
  - VLANs 10 y 20 creadas
  - Root bridge para VLANs 10 y 20 (priority 4096)
  - Trunk Gig0/1 hacia Router CÓRDOBA
  - Trunk Gig0/2 hacia ACC-CORDO
- **Switch ACC-CORDO:** Switch de acceso configurado
  - Puertos de acceso: Fa0/1 (PC2-VLAN10), Fa0/2 (FILE SERVER-VLAN20)
  - Trunk Gig0/2 hacia DIS-CORD con VLANs 10,20
- **PC2 - VLAN 10:** IP 192.168.10.10/24, conectividad completa ✅
- **FILE SERVER - VLAN 20:** IP 192.168.20.10/24, servicios activos ✅
- **Conectividad verificada:**
  - ✅ PC2 → Gateway (192.168.10.1)
  - ✅ PC2 → FILE SERVER (192.168.20.10)
  - ✅ FILE SERVER → PC2 (routing inter-VLAN)
  - ✅ Tablas MAC correctas en todos los switches

### 🚨 Lecciones Aprendidas Críticas (Fases 3-4)

**1. Native VLAN en Trunk con Subinterfaces:**
- ⚠️ **CRÍTICO:** Cuando se usan subinterfaces con 802.1Q, el native VLAN del switch DEBE ser 1 (default)
- ❌ **ERROR:** Configurar `switchport trunk native vlan 1000` rompe la conectividad Layer 2
- ✅ **SOLUCIÓN:** Usar solo `switchport mode trunk` y `switchport trunk allowed vlan 1000`

**2. Interfaces de Backup según Topología Física:**
- BS.AS: Gig0/2 → Gig0/2.1000 (interface dedicada)
- CÓRDOBA: Gig0/0 → Gig0/0.1000 (compartida con trunk al switch)
- MENDOZA: Gig0/1 → Gig0/1.1000 (compartida con LANs y backup en mismo trunk)

**3. Comandos OSPF en Packet Tracer:**
- ❌ `ip ospf network broadcast` NO funciona en subinterfaces (error)
- ✅ OSPF auto-detecta correctamente el tipo broadcast en subinterfaces sobre switch

**4. Diagnóstico de Conectividad Layer 2:**
- Usar `show mac address-table` en switch para verificar aprendizaje
- Usar `show arp` en routers para detectar problemas de resolución
- Tabla ARP vacía + MACs aprendidas = problema de native VLAN o tagging

**5. Router-on-a-Stick: Puerto TRUNK vs ACCESS (20/11/2025):**
- ⚠️ **ERROR COMÚN:** Configurar puerto como trunk cuando debería ser access para dispositivos finales
- ✅ **SOLUCIÓN:** 
  - Puertos de **acceso** (PC, servers): `switchport mode access` + VLAN específica
  - Puertos de **trunk** (switch a switch, switch a router): `switchport mode trunk` + VLANs permitidas
  - Verificar con `show interfaces status` que los puertos estén en la VLAN correcta
- 📝 **Ejemplo:** En ACC-CORDO, Fa0/1 debe ser ACCESS VLAN 10, NO trunk

### 🎯 Siguiente Paso
**Fase 5:** Configurar segmento Mendoza local y luego STP, WiFi y servicios

---

## Fase 1: ISP_LOCAL + Internet Cloud (1-2 h) ✅ COMPLETA
**Objetivo:** Dejar operativos los enlaces `G0/0` (Internet), `G0/1` y `G0/2` hacia BS.AS, y asegurar que el router `ISP_INTERNACIONAL` responda.

**Guía detallada:** Ver `01 - guia_segmento_wan.md`

Checklist:
1. ☑ Configurar `ISP_LOCAL` con interfaces G0/0, G0/1, G0/2
2. ☑ Crear rutas estáticas en ISP_LOCAL hacia **192.168.30.0/24** (red BS.AS)
3. ☑ Configurar ruta por defecto hacia Internet (164.25.0.1)
4. ☑ Configurar `ISP_INTERNACIONAL` (G0/0=164.25.0.1)
5. ☑ Configurar `SW_MS_CORE` (VLAN 100 y 101) y conectar servidores
6. ☑ Configurar servidores WEB y DNS con IPs internas (192.168.100.x)
7. ☑ Activar servicios HTTP y DNS en servidores
8. ☑ Verificar: `show ip interface brief`, `show ip route`, `ping 164.25.0.1`
9. ☑ Documentar evidencias: capturas de configuración y pruebas

**Resultado:** ✅ ISP_LOCAL e ISP_INTERNACIONAL operativos y verificados (19/11/2025)

---

## Fase 2: Router y Switch BS.AS (3-4 h) ✅ COMPLETA
**Objetivo:** Habilitar la LAN (**VLAN 30**) y trunk hacia Router BS.AS. Configurar enlaces WAN hacia ISP_LOCAL.

**⚠️ IMPORTANTE:** Según NUEVOSREQUERIMIENTOS, BS.AS usa VLAN 30 (red 192.168.30.0/24) para evitar conflicto con Córdoba.

**Guía detallada:** Ver `02 - guia_segmento_bsas.md`

Checklist:
1. ☑ Configurar PC-BS-AS: IP 192.168.30.10/24, gateway 192.168.30.1, DNS 1.1.1.1
2. ☑ Switch `SW-BS-AS`: Crear VLANs 30, 100, 200
3. ☑ Configurar puerto Fa0/2 en modo access VLAN 30 (hacia PC)
4. ☑ Configurar trunk Gi0/2 con VLANs 30,100,200, nativa VLAN 30 (hacia Router BS.AS)
5. ☑ Router BS.AS: Crear subinterfaz G0/1.30 (VLAN 30)
6. ☑ Asignar IP 192.168.30.1/24 a subinterfaz G0/1.30
7. ☑ Configurar `ip nat inside` en G0/1.30, `ip nat outside` en G0/1.100 y G0/1.200
8. ☑ Crear ACL base FTP_ONLY_PC (reglas se completarán en Fase 6)
9. ☑ Configurar subinterfaces WAN hacia ISP_LOCAL (G0/1.100 y G0/1.200)
10. ☑ Configurar NAT overload en ambas salidas WAN con doble redundancia
11. ☑ Configurar rutas estáticas hacia servidores y ruta por defecto
12. ☑ **Configurar rutas de retorno en ISP_INTERNACIONAL (42.25.25.0/29 y 43.26.26.0/29)**
13. ☑ Pruebas: `ping 192.168.30.10` desde router, `ping 192.168.30.1` desde PC
14. ☑ Pruebas WAN: `ping 42.25.25.2`, `ping 43.26.26.2`, `ping 164.25.0.1`
15. ☑ Pruebas NAT desde PC: `ping 192.168.100.2`, `ping 192.168.100.9`, `ping 164.25.0.1`
16. ☑ Verificar trunk: `show interfaces trunk` en switch
17. ☑ Verificar NAT: `show ip nat translations` y `show ip nat statistics`
18. ☑ Documentar evidencias: configuraciones y pruebas

**Resultado:** ✅ Segmento Buenos Aires completamente funcional con LAN, WAN doble, NAT operativo y conectividad completa a Internet y servidores (19/11/2025)

---

## Fase 3: Enlaces P2P y OSPF entre sitios (4-6 h) ✅ COMPLETA
**Objetivo:** Configurar conectividad entre BS.AS ↔ Córdoba ↔ Mendoza y propagar redes locales mediante OSPF.

**⚠️ CAMBIO CRÍTICO:** Configurar IPs **directamente en interfaces físicas** (NO usar subinterfaces .500) debido a limitación de Packet Tracer.

**Guías detalladas:** 
- Ver `03 - PASO A PASO - OSPF y Enlaces P2P.md` para enlaces y OSPF
- Ver `04 - PASO A PASO - Segmento CORDOBA.md` para configuración local de Córdoba
- Ver `05 - PASO A PASO - Segmento MENDOZA.md` para configuración local de Mendoza (pendiente)

Checklist:
1. ☑ Configurar switches y routers de Córdoba (VLANs 10, 20) - Ver Guía 04 ✅ (20/11/2025)
2. ☐ Configurar switches y routers de Mendoza (VLANs 44, 55, 70) - Ver Guía 05 (pendiente)
3. ☑ Enlace P2P BS.AS ↔ Córdoba: Gig0/0/0 (10.10.1.9/30) ↔ Gig0/0 (10.10.1.10/30)
4. ☑ Enlace P2P BS.AS ↔ Mendoza: Gig0/1/0 (10.10.1.1/30) ↔ Gig0/0/0 (10.10.1.2/30)
5. ⚠️ Enlace P2P Córdoba ↔ Mendoza: Gig0/1/0 ↔ Gig0/1/0 (cable down - issue físico)
6. ☑ Configurar `ip ospf network point-to-point` en cada interfaz P2P
7. ☑ Configurar `router ospf 1` en cada sitio con router-id (1.1.1.1, 2.2.2.2, 3.3.3.3)
8. ☑ Anunciar redes locales: BS.AS (192.168.30.0/24), Córdoba (192.168.10.0/24, 192.168.20.0/24), Mendoza (192.168.44.0/24, 192.168.55.0/24, 192.168.70.0/24)
9. ☑ Anunciar redes P2P en OSPF
10. ☑ Configurar interfaces pasivas en LANs locales
11. ☑ Configurar 2 rutas estáticas en BS.AS hacia ISP_LOCAL con **métricas diferentes**:
    - `ip route 0.0.0.0 0.0.0.0 42.25.25.2 1`
    - `ip route 0.0.0.0 0.0.0.0 43.26.26.2 10`
12. ☑ Configurar `default-information originate` en Router BS.AS
13. ☑ Verificar vecindades: `show ip ospf neighbor` (estado FULL en enlaces disponibles)
14. ☑ Verificar rutas: `show ip route` (rutas OSPF "O" y default route "O*E2")
15. ☑ Pruebas de conectividad entre todos los sitios (exitosas vía enlaces disponibles)
16. ☑ Documentar evidencias y correcciones en guía

**Resultado:** ✅ OSPF operativo con 2 de 3 enlaces P2P activos. Todos los routers tienen visibilidad de todas las LANs vía OSPF. Default route propagada correctamente.

---

## Fase 4: VLAN 1000 / SW_OSPF_BACKUP (1-2 h) ✅ COMPLETA
**Objetivo:** Implementar red de respaldo (broadcast) entre los tres routers mediante VLAN 1000.

**✅ NOTA:** Esta configuración SÍ usa subinterfaces .1000 (no afectada por el cambio del profesor).

**Guía detallada:** Ver `03 - PASO A PASO - OSPF y Enlaces P2P.md`

Checklist:
1. ☑ Configurar `SW_OSPF_BACKUP`: crear VLAN 1000
2. ☑ Configurar puertos del switch en modo **trunk** (NO access) con VLAN 1000 allowed
3. ☑ Router BS.AS: subinterfaz Gig0/2.1000 con IP 172.20.10.1/29
4. ☑ Router Córdoba: subinterfaz Gig0/0.1000 con IP 172.20.10.2/29 (compartida con P2P trunk)
5. ☑ Router Mendoza: subinterfaz Gig0/1.1000 con IP 172.20.10.3/29 (compartida con LANs)
6. ☑ Configurar `encapsulation dot1Q 1000` en cada subinterfaz
7. ☑ NO configurar IP en interfaces físicas
8. ⚠️ **NO configurar** `ip ospf network broadcast` (auto-detectado correctamente)
9. ☑ Configurar `ip ospf priority` y `ip ospf cost 50` para control y diferenciación
10. ☑ Anunciar red 172.20.10.0/29 en OSPF de cada router
11. ☑ Verificar vecindades: `show ip ospf neighbor` (BS.AS=DR, CÓRDOBA=BDR, MENDOZA=DROTHER)
12. ☑ Probar conectividad: `ping` entre subinterfaces .1000 (100% éxito)
13. ☑ **FIX CRÍTICO:** Native VLAN debe ser 1 (NO 1000) en puertos trunk del switch
14. ☑ Documentar evidencias y lecciones aprendidas

**Resultado:** ✅ Red de respaldo completamente operativa. Conectividad 172.20.10.0/29 verificada entre todos los routers. Switch aprende 3 MACs en VLAN 1000. Vecindades OSPF broadcast establecidas correctamente.

---

## Fase 4B: Segmento Córdoba Local (2-3 h) ✅ COMPLETA (20/11/2025)
**Objetivo:** Configurar el segmento local de Córdoba con VLANs 10 y 20, switches de acceso y distribución, y verificar conectividad inter-VLAN.

**Guía detallada:** Ver `04 - PASO A PASO - Segmento CORDOBA.md`

Checklist:
1. ☑ Configurar Switch ACC-CORDO: VLANs 10 y 20, puertos de acceso
2. ☑ Puerto Fa0/1: Access VLAN 10 para PC2
3. ☑ Puerto Fa0/2: Access VLAN 20 para FILE SERVER
4. ☑ Puerto Gig0/2: Trunk con VLANs 10,20 hacia DIS-CORD
5. ☑ Configurar Switch DIS-CORD: VLANs 10 y 20, trunks bidireccionales
6. ☑ Trunk Gig0/1 hacia Router CÓRDOBA con VLANs 10,20
7. ☑ Trunk Gig0/2 hacia ACC-CORDO con VLANs 10,20
8. ☑ Configurar STP Root Bridge: priority 4096 para VLANs 10 y 20
9. ☑ Router CÓRDOBA: Configurar interfaz física Gig0/2 sin IP (trunk)
10. ☑ Subinterfaz Gig0/2.10: 192.168.10.1/24 con encapsulación dot1Q 10
11. ☑ Subinterfaz Gig0/2.20: 192.168.20.1/24 con encapsulación dot1Q 20
12. ☑ Configurar OSPF: Interfaces pasivas en Gig0/2.10 y Gig0/2.20
13. ☑ Configurar PC2: IP 192.168.10.10/24, gateway 192.168.10.1
14. ☑ Configurar FILE SERVER: IP 192.168.20.10/24, gateway 192.168.20.1
15. ☑ Verificar conectividad: PC2 → Gateway (ping exitoso)
16. ☑ Verificar routing inter-VLAN: PC2 → FILE SERVER (ping exitoso)
17. ☑ Verificar tablas MAC en switches (dispositivos aprendidos correctamente)
18. ☑ Verificar tabla ARP en router (ambos dispositivos visibles)
19. ☑ **FIX CRÍTICO:** Corregir puerto Fa0/1 de trunk a access VLAN 10
20. ☑ Documentar configuraciones finales y pruebas exitosas

**Resultado:** ✅ Segmento Córdoba completamente operativo con routing inter-VLAN funcional. Conectividad local verificada entre PC2 (VLAN 10) y FILE SERVER (VLAN 20). Router CÓRDOBA integrado correctamente con OSPF propagando redes 192.168.10.0/24 y 192.168.20.0/24.

---

## Fase 5: STP, WiFi y servicios locales (2-3 h)
**Objetivo:** Configurar STP (switches Core como root bridge), WiFi dual-SSID y servicios DHCP.

Checklist:
1. ☐ STP Buenos Aires: Configurar SW Core como root bridge para VLANs 30,100,200
2. ☐ STP Córdoba: Configurar SW_CORE_DS_CORO como root bridge para VLANs 10,20
3. ☐ STP Mendoza: Configurar SW_CORE_DS_MEND como root bridge para VLANs 44,55,70
4. ☐ Verificar: `show spanning-tree vlan <id>` en cada switch
5. ☐ Configurar Access Point en Mendoza con IP 192.168.70.3
6. ☐ Crear SSID "INTERNOS" asociado a VLAN 44
7. ☐ Crear SSID "INVITADOS" asociado a VLAN 55
8. ☐ Configurar VLAN 70 como management del AP
9. ☐ Configurar servidor DHCP para VLANs necesarias (opcional según diseño)
10. ☐ Probar conectividad WiFi: laptop conectada a SSID-INTERNOS obtiene IP en VLAN 44
11. ☐ Probar conectividad WiFi: laptop conectada a SSID-INVITADOS obtiene IP en VLAN 55
12. ☐ Documentar configuración de AP y pruebas

**Resultado esperado:** STP configurado, WiFi operativo con segmentación correcta.

---

## Fase 6: NAT Servicios Internet, pruebas finales y documentación (2 h)
**Objetivo:** Configurar NAT para salida a Internet y servicios públicos, implementar ACL, realizar pruebas integrales.

Checklist NAT:
1. ☐ Configurar NAT PAT en Router BS.AS para salida a Internet:
   - Ambas interfaces hacia ISP_LOCAL como `ip nat outside`
   - Subinterfaz G0/1.30 como `ip nat inside`
   - NAT por traducción de puertos (PAT/desborde) usando IP de cada interfaz
2. ☐ Configurar NAT estático para WEB Server: IP interna → 45.162.20.10
3. ☐ Configurar NAT estático para DNS Server: IP interna → 1.1.1.1

Checklist ACL:
4. ☐ Configurar ACL extendida FTP_ONLY_PC
5. ☐ Permitir: PC-BS-AS (192.168.30.10) → FTP Server puerto 21
6. ☐ Denegar: Todos los demás → FTP Server
7. ☐ Aplicar ACL en interfaz/dirección correcta

Checklist Pruebas:
8. ☐ Desde PC-BS-AS: `ping 1.1.1.1`, `ping 45.162.20.10`
9. ☐ Desde PC-BS-AS: `nslookup www.consultas.labo.com.ar 1.1.1.1`
10. ☐ Desde PC-BS-AS: acceso HTTP a 45.162.20.10
11. ☐ Desde PC-BS-AS: acceso FTP a servidor FTP (✅ debe funcionar)
12. ☐ Desde PC-Cordoba: acceso FTP a servidor FTP (❌ debe fallar)
13. ☐ Desde cualquier PC: `ping` a sitios remotos vía OSPF
14. ☐ Verificar tablas NAT: `show ip nat translations`

Checklist Documentación:
15. ☐ Capturar todas las configuraciones finales (`show run`)
16. ☐ Capturar tablas de ruteo de todos los routers
17. ☐ Capturar vecindades OSPF
18. ☐ Capturar pruebas exitosas y fallidas
19. ☐ Completar `ticket_trabajo_practico.md` con todas las evidencias
20. ☐ Revisar que todos los requerimientos están cumplidos

**Resultado esperado:** Proyecto completamente funcional y documentado.

---

## Seguimiento y Actualización
- Usa este plan como checklist maestro
- Marca cada ítem (☐ → ☑) cuando esté completado y probado
- Documenta evidencias en `CONFIG POR DISP` y `ticket_trabajo_practico.md`
- Consulta las guías detalladas (01, 02, 03) para cada fase
- Ante dudas, revisar `CAMBIOS_CRITICOS_PROFESOR.md`

---

**Última actualización:** 20/11/2025  
**Estado del proyecto:** 75% completado - Fases 1-4B operativas
- Infraestructura Internet ✅
- Segmento Buenos Aires ✅
- Enlaces P2P y OSPF ✅ (2 de 3 enlaces)
- Red de backup VLAN 1000 ✅
- **Segmento Córdoba local ✅ (20/11/2025)**
- Pendiente: Segmento Mendoza local, STP completo, WiFi, NAT servicios públicos

**Próximos pasos:**
- Configurar segmento Mendoza local (VLANs 44, 55, 70)
- Fase 5: Configurar STP restante y WiFi dual-SSID en Mendoza
- Fase 6: NAT servicios Internet, ACL, documentación final
- Opcional: Resolver cable P2P CÓRDOBA-MENDOZA para 4 vecinos completos

**Guías actualizadas:**
- ✅ `01 - guia_segmento_wan.md` - Configuración ISP y servidores
- ✅ `02 - guia_segmento_bsas.md` - Segmento Buenos Aires con NAT
- ✅ `03 - PASO A PASO - OSPF y Enlaces P2P.md` - Enlaces P2P y OSPF
- ✅ `04 - PASO A PASO - Segmento CORDOBA.md` - Segmento Córdoba local ✅ (20/11/2025)
- ⏳ `05 - PASO A PASO - Segmento MENDOZA.md` - Segmento Mendoza local (pendiente)
- ⏳ `03 - guia_ospf_enlaces_p2p.md` - Versión detallada (pendiente actualizar)

---

**Nota importante:** Después de cada fase, actualiza `ticket_trabajo_practico.md` con fecha, responsable y evidencias adjuntas. Si aparece una corrección del profesor, regresa a la fase correspondiente, ajusta y vuelve a ejecutar las verificaciones señaladas.
