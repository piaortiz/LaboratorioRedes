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
| Fase | Alcance | Estimación | Dependencias | Estado (19/11) |
|------|---------|------------|--------------|----------------|
| 1 | ISP_LOCAL + Internet Cloud | 1-2 h | Ninguna | ⏳ Pendiente |
| 2 | Router y Switch BS.AS (VLAN 30 + trunk) | 3-4 h | Fase 1 | ⏳ Pendiente |
| 3 | Enlaces P2P y OSPF entre sitios | 4-6 h | Fases 1-2 (BS.AS operativo) | ⏳ Pendiente |
| 4 | VLAN 1000 / SW_OSPF_BACKUP (Broadcast OSPF) | 1-2 h | Fase 3 | ⏳ Pendiente |
| 5 | STP, WiFi y servicios locales (SSID, root bridges) | 2-3 h | Fases 2-4 | ⏳ Pendiente |
| 6 | NAT Servicios Internet + pruebas integrales + documentación | 2 h | Todas las anteriores | ⏳ Pendiente |

Total estimado: **13-19 horas** según nivel de detalle de las pruebas y captura de evidencias.

**Progreso actual: 0% - Proyecto desde cero con nuevos requerimientos**

---

## Fase 1: ISP_LOCAL + Internet Cloud (1-2 h)
**Objetivo:** Dejar operativos los enlaces `G0/0` (Internet), `G0/1` y `G0/2` hacia BS.AS, y asegurar que el router `ISP_INTERNACIONAL` responda.

**Guía detallada:** Ver `01 - guia_segmento_wan.md`

Checklist:
1. ☐ Configurar `ISP_LOCAL` con interfaces G0/0, G0/1, G0/2
2. ☐ Crear rutas estáticas en ISP_LOCAL hacia **192.168.30.0/24** (red BS.AS)
3. ☐ Configurar ruta por defecto hacia Internet (164.25.0.1)
4. ☐ Configurar `ISP_INTERNACIONAL` (G0/0=164.25.0.1)
5. ☐ Configurar `SW_MS_CORE` (VLAN 100) y conectar servidores
6. ☐ Configurar servidores WEB y DNS con IPs internas (192.168.100.x)
7. ☐ Activar servicios HTTP y DNS en servidores
8. ☐ Verificar: `show ip interface brief`, `show ip route`, `ping 164.25.0.1`
9. ☐ Documentar evidencias: capturas de configuración y pruebas

**Resultado esperado:** ISP_LOCAL e ISP_INTERNACIONAL operativos, listos para recibir tráfico desde BS.AS.

---

## Fase 2: Router y Switch BS.AS (3-4 h)
**Objetivo:** Habilitar la LAN (**VLAN 30**) y trunk hacia Router BS.AS. Configurar enlaces WAN hacia ISP_LOCAL.

**⚠️ IMPORTANTE:** Según NUEVOSREQUERIMIENTOS, BS.AS usa VLAN 30 (red 192.168.30.0/24) para evitar conflicto con Córdoba.

**Guía detallada:** Ver `02 - guia_segmento_bsas.md`

Checklist:
1. ☐ Configurar PC-BS-AS: IP 192.168.30.10/24, gateway 192.168.30.1, DNS 1.1.1.1
2. ☐ Switch `SW-BS-AS`: Crear VLANs 30, 100, 200
3. ☐ Configurar puerto Fa0/2 en modo access VLAN 30 (hacia PC)
4. ☐ Configurar trunk Gi0/2 con VLANs 30,100,200, nativa VLAN 30 (hacia Router BS.AS)
5. ☐ Router BS.AS: Crear subinterfaz G0/1.30 (VLAN 30)
6. ☐ Asignar IP 192.168.30.1/24 a subinterfaz G0/1.30
7. ☐ Configurar `ip nat inside` en G0/1.30
8. ☐ Crear ACL base FTP_ONLY_PC (reglas se completarán en Fase 6)
9. ☐ Configurar enlaces WAN hacia ISP_LOCAL (subinterfaces o físicas según diseño)
10. ☐ Pruebas: `ping 192.168.30.10` desde router, `ping 192.168.30.1` desde PC
11. ☐ Verificar trunk: `show interfaces trunk` en switch
12. ☐ Documentar evidencias: configuraciones y pruebas

**Resultado esperado:** LAN de BS.AS operativa en VLAN 30, PC puede comunicarse con el router.

---

## Fase 3: Enlaces P2P y OSPF entre sitios (4-6 h)
**Objetivo:** Configurar conectividad entre BS.AS ↔ Córdoba ↔ Mendoza y propagar redes locales mediante OSPF.

**⚠️ CAMBIO CRÍTICO:** Configurar IPs **directamente en interfaces físicas** (NO usar subinterfaces .500) debido a limitación de Packet Tracer.

**Guía detallada:** Ver `03 - guia_ospf_enlaces_p2p.md`

Checklist:
1. ☐ Configurar switches y routers de Córdoba (VLANs 10, 20)
2. ☐ Configurar switches y routers de Mendoza (VLANs 44, 55, 70)
3. ☐ Enlace P2P BS.AS ↔ Córdoba: IPs directas en interfaces físicas (ej: 10.10.1.9/30 y 10.10.1.10/30)
4. ☐ Enlace P2P BS.AS ↔ Mendoza: IPs directas en interfaces físicas (ej: 10.10.1.1/30 y 10.10.1.2/30)
5. ☐ Enlace P2P Córdoba ↔ Mendoza: IPs directas en interfaces físicas (ej: 10.10.1.17/30 y 10.10.1.18/30)
6. ☐ Configurar `ip ospf network point-to-point` en cada interfaz P2P
7. ☐ Configurar `router ospf 1` en cada sitio
8. ☐ Anunciar redes locales: BS.AS (192.168.30.0/24), Córdoba (192.168.10.0/24, 192.168.20.0/24), Mendoza (192.168.44.0/24, 192.168.55.0/24, 192.168.70.0/24)
9. ☐ Anunciar redes P2P en OSPF
10. ☐ Configurar interfaces pasivas en LANs locales
11. ☐ Configurar 2 rutas estáticas en BS.AS hacia ISP_LOCAL con **métricas diferentes**:
    - `ip route 0.0.0.0 0.0.0.0 42.25.25.2 1`
    - `ip route 0.0.0.0 0.0.0.0 43.26.26.2 10`
12. ☐ Configurar `default-information originate` en Router BS.AS
13. ☐ Verificar vecindades: `show ip ospf neighbor` (estado FULL, tipo P2P)
14. ☐ Verificar rutas: `show ip route` (rutas OSPF "O" y estáticas "S*")
15. ☐ Pruebas de conectividad entre todos los sitios
16. ☐ Documentar evidencias

**Resultado esperado:** Todos los sitios conectados vía OSPF, ruta por defecto propagada desde BS.AS.

---

## Fase 4: VLAN 1000 / SW_OSPF_BACKUP (1-2 h)
**Objetivo:** Implementar red de respaldo (broadcast) entre los tres routers mediante VLAN 1000.

**✅ NOTA:** Esta configuración SÍ usa subinterfaces .1000 (no afectada por el cambio del profesor).

**Guía detallada:** Ver sección 3.1 de `03 - guia_ospf_enlaces_p2p.md`

Checklist:
1. ☐ Configurar `SW_OSPF_BACKUP`: crear VLAN 1000
2. ☐ Configurar puertos del switch en modo access VLAN 1000 hacia cada router
3. ☐ Router BS.AS: subinterfaz Fa0/22.1000 con IP 172.20.10.1/29
4. ☐ Router Córdoba: subinterfaz Fa0/24.1000 con IP 172.20.10.2/29
5. ☐ Router Mendoza: subinterfaz Fa0/23.1000 con IP 172.20.10.3/29
6. ☐ Configurar `encapsulation dot1Q 1000` en cada subinterfaz
7. ☐ NO configurar IP en interfaces físicas (Fa0/22, Fa0/24, Fa0/23)
8. ☐ Configurar `ip ospf network broadcast` en cada subinterfaz .1000
9. ☐ Opcional: Configurar `ip ospf priority` para control de DR/BDR
10. ☐ Anunciar red 172.20.10.0/29 en OSPF de cada router
11. ☐ Verificar vecindades: `show ip ospf neighbor` (tipo DR/BDR/DROTHER)
12. ☐ Probar conectividad: `ping` entre subinterfaces .1000
13. ☐ Verificar estabilidad (no cambios frecuentes de DR)
14. ☐ Documentar evidencias

**Resultado esperado:** Red de respaldo operativa, vecindades OSPF tipo Broadcast establecidas.

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

**Última actualización:** 19/11/2025  
**Estado del proyecto:** Reiniciado con nuevos requerimientos - 0% completado
- Después de cada fase, actualiza `ticket_trabajo_practico.md` con fecha, responsable y evidencias adjuntas.
- Si aparece una corrección del profesor, regresa a la fase correspondiente, ajusta y vuelve a ejecutar las verificaciones señaladas.
