# Checklist de Verificación Paso a Paso - Acceso de PC2 Córdoba a WEB/DNS Server y a Internet

## 1. Verificar configuración de VLAN y puertos en switches de Córdoba
- [ ] El puerto donde está conectada PC2 está en modo access y en VLAN 10.
- [ ] El puerto donde está conectado el FILE SERVER está en modo access y en VLAN 20.
- [ ] Los trunks entre SW-ACC-CORDO, DIS-CORD y el router permiten VLANs 10 y 20.
- [ ] Comando útil: `show vlan brief`, `show interfaces trunk` en ambos switches.

## 2. Verificar configuración IP en PC2 y FILE SERVER
- [ ] PC2 tiene IP en 192.168.10.x/24, máscara 255.255.255.0, gateway 192.168.10.1.
- [ ] FILE SERVER tiene IP en 192.168.20.x/24, máscara 255.255.255.0, gateway 192.168.20.1.

## 3. Verificar configuración de subinterfaces y OSPF en router CORDOBA
- [ ] Subinterfaces G0/2.10 y G0/2.20 están up y con IP correcta.
- [ ] OSPF anuncia redes 192.168.10.0/24 y 192.168.20.0/24.
- [ ] Comando útil: `show ip ospf interface brief`, `show ip route`.

## 4. Verificar ruta por defecto en router CORDOBA
- [ ] Existe `ip route 0.0.0.0 0.0.0.0 10.10.1.9`.
- [ ] Comando útil: `show running-config | include ip route`.

## 5. Verificar conectividad desde CORDOBA a BS.AS
- [ ] Desde router CORDOBA, ping a 10.10.1.9 (BS.AS).
- [ ] Desde router CORDOBA, ping a 192.168.30.1 (BS.AS).
- [ ] Desde router CORDOBA, ping a 192.168.100.2 y 192.168.100.9 (DNS/WEB Server).

## 6. Verificar NAT en BS.AS
- [ ] La access-list de NAT incluye 192.168.10.0/24 y 192.168.20.0/24.
- [ ] NAT dinámico (PAT) está configurado para salida a Internet.
- [ ] NAT estático está configurado:
  - [ ] Para el WEB Server: `ip nat inside source static 192.168.100.9 45.162.20.10`
  - [ ] Para el DNS Server: `ip nat inside source static 192.168.100.2 1.1.1.1`
- [ ] Comando útil: `show running-config | include nat`, `show ip nat translations`.

## 7. Verificar rutas y reachability en BS.AS
- [ ] BS.AS tiene rutas OSPF para 192.168.10.0/24 y 192.168.20.0/24.
- [ ] Comando útil: `show ip route`.

## 8. Verificar conectividad desde PC2
- [ ] PC2 puede hacer ping a su gateway (192.168.10.1).
- [ ] PC2 puede hacer ping a 192.168.100.2 y 192.168.100.9.
- [ ] PC2 puede hacer ping a 8.8.8.8 (Internet).

## 9. Verificar ACLs
- [ ] No hay ACLs en BS.AS o CORDOBA que bloqueen tráfico entre Córdoba y los servidores.

---

> Realiza cada verificación en orden y anota el resultado. Si algún paso falla, detente y corrige antes de continuar.
