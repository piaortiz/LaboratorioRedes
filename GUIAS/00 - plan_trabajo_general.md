# Plan de Trabajo Global TP Redes
**Fecha:** 17/11/2025  
**Objetivo:** Ejecutar el trabajo práctico completo siguiendo el orden óptimo sugerido, con estimación de esfuerzo y checklist por fase. Los tiempos son aproximados suponiendo dedicación continua en Cisco Packet Tracer.

---

## Resumen por Fases
| Fase | Alcance | Estimación | Dependencias | Estado (17/11) |
|------|---------|------------|--------------|----------------|
| 1 | ISP_LOCAL + Internet Cloud | 1-2 h | Ninguna | ✅ Completada: routers ISP_INTERNACIONAL/ISP_LOCAL y SW_MS_CORE operativos; pendientes de esta fase solo las pruebas que dependen de Router BS.AS (pings 42/43.x) |
| 2 | Router y Switch BS.AS (LAN + trunk) | 3-4 h | Fase 1 | ✅ Completada: LAN VLAN 20 operativa, trunk Gi0/2↔Router BS.AS G0/1 funcional, ACL FTP creada; pendientes rutas/NAT para fases posteriores |
| 3 | Enlaces P2P y OSPF entre sitios | 4-6 h | Fases 1-2 (BS.AS operativo) | ▶️ Próxima: configurar enlaces P2P y OSPF según notas profesor |
| 4 | VLAN 1000 / SW_OSPF_BACKUP (Broadcast OSPF) | 1-2 h | Fase 3 | ⏳ Programada |
| 5 | STP, WiFi y servicios locales (SSID, root bridges) | 2-3 h | Fases 2-4 | ⏳ Programada |
| 6 | NAT Servicios Internet + pruebas integrales + documentación | 2 h | Todas las anteriores | ⏳ Programada |

Total estimado: **13-19 horas** según nivel de detalle de las pruebas y captura de evidencias.

---

## Fase 1: ISP_LOCAL + Internet Cloud (1-2 h)
**Objetivo:** Dejar operativos los enlaces `G0/0` (Internet), `G0/1` y `G0/2` hacia BS.AS, y asegurar que el router `ISP_INTERNACIONAL` responda.

Checklist:
1. `ISP_LOCAL` → `show ip interface brief` (G0/0-2 en `up/up`). ✅ Capturado.
2. `ISP_LOCAL` → `ip route 0.0.0.0 0.0.0.0 164.25.0.1` + rutas a 192.168.20.0/24. ✅ Configurado (quedan sin probar las rutas hacia 42/43.x hasta que exista Router BS.AS).
3. Pings: `164.25.0.1`, `42.25.25.1`, `43.26.26.1`. ✅ 164.25.0.1 responde; 42/43.x pendientes por falta de Router BS.AS (documentado en ticket).
4. Internet Cloud: configurar `ISP_INTERNACIONAL` (G0/0=164.25.0.1), `SW_MS_CORE` (VLAN 100) y servidores con IPs internas + servicios activos. ✅ Terminado; IPs públicas se aplicarán luego vía NAT en Router BS.AS según guía 01.
5. Evidencias: `show running-config`, `show ip route`, `show interfaces trunk`, pings `164.25.0.2`. ✅ Guardadas.

> Estado actual fase 1: ISP_INTERNACIONAL y SW_MS_CORE quedan operativos y documentados; servidores WEB/DNS tienen IPs 192.168.100.9/2 con servicios activos, a la espera del NAT estático en la Fase 6.

---

## Fase 2: Router y Switch BS.AS (3-4 h)
**Objetivo:** Habilitar la LAN (VLAN 20) y trunk hacia Router BS.AS. NAT y rutas WAN se configurarán en fases posteriores según topología final.

Checklist:
1. Switch `SW-BS-AS`: VLAN 20/100/200, puerto Fa0/2 (PC) access VLAN 20, trunk `Gi0/2` → Router BS.AS G0/1. ✅ Completado.
2. `Router BS.AS`:
   - Subinterfaz `G0/1.20` con IP 192.168.20.1/24 (`encapsulation dot1Q 20`, `ip nat inside`). ✅ Completado.
   - ACL FTP: crear `FTP_ONLY_PC` con remark (reglas pendientes de IP servidor). ✅ Completado.
3. Pruebas desde Router BS.AS: `ping 192.168.20.10` exitoso. ✅ Completado.
4. Validación ISP_LOCAL: ruta a 192.168.20.0/24 vía 42.25.25.1 presente. ✅ Completado.
5. Evidencias guardadas: `show ip int brief`, `show ip route`, `show interfaces trunk` (switch). ✅ Completado.

> Estado actual fase 2: Router BS.AS con LAN operativa (G0/1.20 up/up), trunk funcional hacia switch, ACL FTP base creada. Pendientes: configuración WAN directa y NAT cuando se defina esquema completo en fase OSPF.

---

## Fase 3: Enlaces P2P y OSPF entre sitios (4-6 h)
**Objetivo:** Configurar conectividad entre BS.AS ↔ Córdoba ↔ Mendoza respetando la nota del profesor (sin subinterfaces .500) y propagar redes locales.

Checklist:
1. Configurar IPs directamente en interfaces físicas de cada enlace P2P.
2. `router ospf <process>` en cada sitio:
   - Tipo punto a punto en cada enlace (`ip ospf network point-to-point`).
   - Anunciar redes locales (`network 192.168.x.0 0.0.0.255 area 0`).
   - `passive-interface` en LAN.
3. Verificar vecinos: `show ip ospf neighbor` en cada router.
4. Pings cruzados entre routers (loopbacks o interfaces WAN) para validar SPF.
5. Documentar `show ip route` mostrando rutas OSPF (O) hacia las LAN remotas.

---

## Fase 4: VLAN 1000 / SW_OSPF_BACKUP (1-2 h)
**Objetivo:** Implementar el enlace compartido en broadcast que une los tres routers mediante la VLAN 1000.

Checklist:
1. Crear VLAN 1000 en `SW_OSPF_BACKUP`, puertos configurados como trunk/access según corresponda.
2. En cada router, subinterfaz `.1000` con `encapsulation dot1Q 1000` e IP correcta.
3. `ip ospf network broadcast` y, si se necesita, `ip ospf priority` para definir DR/BDR.
4. `show ip ospf neighbor` debe mostrar relaciones de tipo BDR/DR entre los tres routers.
5. Pruebas: `ping` entre las subinterfaces `.1000` y verificación de estabilidad (sin cambios de DR frecuentes).

---

## Fase 5: STP, WiFi y servicios locales (2-3 h)
**Objetivo:** Cumplir STP (core como root bridge), configurar SSID en VLAN 44/55 y afinar servicios internos.

Checklist:
1. Switches de cada LAN: `spanning-tree vlan <id> priority <value>` para fijar root/secondary.
2. Access points / WLC (según PKT): asignar SSID `INTERNOS` → VLAN 44, `INVITADOS` → VLAN 55.
3. DHCP/Reservas según VLAN (si aplica).
4. Pruebas: dispositivos WiFi conectándose a cada SSID con la VLAN correcta (`ipconfig` para verificar).
5. Capturas de `show spanning-tree vlan` y configuración de los APs.

---

## Fase 6: NAT Servicios Internet, pruebas finales y documentación (2 h)
**Objetivo:** Terminar NAT estático para WEB/DNS, validar acceso desde LAN e Internet, y cerrar evidencias/tickets.

Checklist:
1. En router de servicios (ISP_INTERNACIONAL o firewall según diseño):
   - `ip nat inside source static 192.168.10.9 45.162.20.10`
   - `ip nat inside source static 192.168.100.2 1.1.1.1`
2. Desde PC-BS-AS: `nslookup www.consultas.labo.com.ar 1.1.1.1`, `ping 45.162.20.10`.
3. Desde servidores: comprobar salida a Internet (`ping 164.25.0.2` y destinos públicos).
4. Revisión final de ACLs para asegurar que solo la PC autorizada accede al FTP.
5. Documentar todo en `ticket_trabajo_practico.md`: evidencias por fase, comandos ejecutados, resultados.

---

## Seguimiento y Actualización
- Usa este plan como checklist maestro. Marca cada ítem cuando tengas las pruebas guardadas.
- Después de cada fase, actualiza `ticket_trabajo_practico.md` con fecha, responsable y evidencias adjuntas.
- Si aparece una corrección del profesor, regresa a la fase correspondiente, ajusta y vuelve a ejecutar las verificaciones señaladas.
