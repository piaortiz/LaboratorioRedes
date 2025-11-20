# Registro de cambio de configuración - NAT en BS.AS

## Fecha: 20/11/2025

### Problema detectado
Las PCs de Córdoba (redes 192.168.10.0/24 y 192.168.20.0/24) no podían acceder a Internet ni a los servidores detrás de NAT en Buenos Aires, ya que la access-list 1 utilizada para NAT solo incluía la red local de Buenos Aires (192.168.30.0/24).

### Solución aplicada
Se amplió la access-list 1 en el router BS.AS para incluir las redes de Córdoba:

```
access-list 1 permit 192.168.30.0 0.0.0.255
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
```

Esto permite que el tráfico de todas estas redes sea NATeado y pueda salir a Internet o acceder a los servidores protegidos por NAT.

### Resultado
- Las PCs de Córdoba pueden acceder a Internet y a los servidores detrás de NAT en Buenos Aires.
- El acceso a Internet y servicios es funcional para todas las sedes internas.

---

> Documentar este cambio permite recordar que, al usar NAT centralizado, todas las redes internas que requieran salida deben estar incluidas en la access-list utilizada por NAT.
