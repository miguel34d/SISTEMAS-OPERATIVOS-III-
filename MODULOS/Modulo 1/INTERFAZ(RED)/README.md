<div align="center">

# 📸 Demostración – Práctica 2
## Configuración de Parámetros de Red

![Alumno](https://img.shields.io/badge/Alumno-Miguel%20Ram%C3%ADrez%20Meli-2563eb?style=for-the-badge&logo=github&logoColor=white)
![Matrícula](https://img.shields.io/badge/Matr%C3%ADcula-2025--1367-16a34a?style=for-the-badge)
![ITLA](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-ea580c?style=for-the-badge)
![Materia](https://img.shields.io/badge/Materia-Sistemas%20Operativos%20III-7c3aed?style=for-the-badge&logo=linux&logoColor=white)
![Profesor](https://img.shields.io/badge/Profesor-Adri%C3%A1n%20Alc%C3%A1ntara-db2777?style=for-the-badge)
![Práctica](https://img.shields.io/badge/Pr%C3%A1ctica-2%20Par%C3%A1metros%20de%20Red-0891b2?style=for-the-badge)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%20(VMware)-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

</div>

> **Alumno:** Miguel Ramírez Meli · **Matrícula:** 2025-1367 · **ITLA**
> **Materia:** Sistemas Operativos III · **Profesor:** Adrián Alcántara
> **Entorno:** Ubuntu en VMware · interfaz real `ens33` · interfaz virtual `CLARO`

Cada paso muestra el comando ejecutado y su captura (carpeta `imagen/`).

---

## Parte 1 - Comandos `ip`, rutas, DNS y DHCP (práctica "prueba")

### 1.1 Crear la interfaz CLARO y mostrar interfaces de red

```bash
sudo modprobe dummy
sudo ip link add CLARO type dummy
sudo ip link set CLARO up
ip a show CLARO
ip link show CLARO
```

**Creación de la interfaz CLARO y verificación con `ip a show` e `ip link show`**

![Creación de la interfaz CLARO y verificación con `ip a show` e `ip link show`](imagen/01_crear_interfaz_CLARO.png)

### 1.2 Agregar una IP manualmente

```bash
sudo ip addr add 20.13.67.10/24 dev CLARO
ip a show CLARO
```

**IP 20.13.67.10/24 asignada a CLARO**

![IP 20.13.67.10/24 asignada a CLARO](imagen/02_agregar_ip_manual.png)

### 1.3 Eliminar y volver a agregar la IP

```bash
sudo ip addr del 20.13.67.10/24 dev CLARO
ip a show CLARO
sudo ip addr add 20.13.67.10/24 dev CLARO
ip a show CLARO
```

**IP eliminada: solo queda la IPv6 link-local**

![IP eliminada: solo queda la IPv6 link-local](imagen/03_eliminar_ip.png)

**IP agregada nuevamente**

![IP agregada nuevamente](imagen/04_agregar_ip_otra_vez.png)

### 1.4 Mostrar rutas

```bash
ip route
```

**Tabla de rutas con la red `20.13.67.0/24` de CLARO**

![Tabla de rutas con la red `20.13.67.0/24` de CLARO](imagen/05_mostrar_rutas.png)

### 1.5 Agregar gateway

```bash
sudo ip route add default via 20.13.67.1 dev CLARO metric 200
ip route
```

**Dos rutas default: `ens33` (100) y CLARO (200)**

![Dos rutas default: `ens33` (100) y CLARO (200)](imagen/06_agregar_gateway.png)

### 1.6 Eliminar el gateway

```bash
sudo ip route del default via 20.13.67.1 dev CLARO
ip route
```

**Ruta default de CLARO eliminada**

![Ruta default de CLARO eliminada](imagen/07_eliminar_gateway.png)

### 1.7 DEFAULT (volver a poner el gateway por defecto)

```bash
sudo ip route add default via 20.13.67.1 dev CLARO metric 200
ip route | grep default
ip route show default
```

**Default agregada y filtrada con `grep`**

![Default agregada y filtrada con `grep`](imagen/08_default_agregar_grep.png)

**`ip route show default`**

![`ip route show default`](imagen/09_default_mostrar.png)

### 1.8 Editar DNS

```bash
cat /etc/resolv.conf
resolvectl status
sudo resolvectl dns ens33 8.8.8.8 1.1.1.1
resolvectl status ens33
cat /etc/resolv.conf
```

**`/etc/resolv.conf` antes del cambio**

![`/etc/resolv.conf` antes del cambio](imagen/10_dns_antes_resolv_conf.png)

**`resolvectl status`: DNS `10.13.67.2` entregado por DHCP**

![`resolvectl status`: DNS `10.13.67.2` entregado por DHCP](imagen/11_dns_resolvectl_status.png)

**DNS cambiado con `resolvectl dns`**

![DNS cambiado con `resolvectl dns`](imagen/12_dns_resolvectl_dns.png)

**`/etc/resolv.conf` con `8.8.8.8` y `1.1.1.1`**

![`/etc/resolv.conf` con `8.8.8.8` y `1.1.1.1`](imagen/13_dns_despues_resolv_conf.png)

**`ping` sin respuesta: ICMP a internet bloqueado por el NAT de la VM (el nombre sí se resolvió)**

![`ping` sin respuesta: ICMP a internet bloqueado por el NAT de la VM (el nombre sí se resolvió)](imagen/14_ping_sin_respuesta.png)

### 1.9 Probar el DNS sin depender de ICMP

```bash
resolvectl query google.com
nslookup google.com
ping -c 4 10.13.67.2
```

**`resolvectl query google.com`**

![`resolvectl query google.com`](imagen/15_resolvectl_query.png)

**`nslookup` y ping al gateway `10.13.67.2` con 0% de pérdida**

![`nslookup` y ping al gateway `10.13.67.2` con 0% de pérdida](imagen/16_nslookup_ping_gateway.png)

### 1.10 Renovar DHCP

```bash
ip a show ens33
sudo dhclient -r ens33
ip a show ens33
sudo networkctl renew ens33
ip a show ens33
resolvectl status ens33
ip route show default
```

**`ens33` antes: lease de 1396 s**

![`ens33` antes: lease de 1396 s](imagen/17_dhcp_antes.png)

**`dhclient -r` sin efecto**

![`dhclient -r` sin efecto](imagen/18_dhclient_r.png)

**Lease en 1365 s: solo pasó el tiempo**

![Lease en 1365 s: solo pasó el tiempo](imagen/19_dhcp_despues_dhclient.png)

**`networkctl renew ens33`**

![`networkctl renew ens33`](imagen/20_networkctl_renew.png)

**Lease renovado (1769 s), DNS manuales y rutas intactos**

![Lease renovado (1769 s), DNS manuales y rutas intactos](imagen/21_dhcp_renovado.png)

---

## Parte 2 - NMCLI (NetworkManager)

### 2.1 Instalar NetworkManager

```bash
nmcli device status          # nmcli no encontrado
sudo apt update
sudo apt install network-manager -y
nmcli --version
nmcli device status
```

**`nmcli` no está instalado**

![`nmcli` no está instalado](imagen/22_nmcli_no_instalado.png)

**Todas las interfaces `unmanaged`**

![Todas las interfaces `unmanaged`](imagen/23_nmcli_unmanaged.png)

### 2.2 Hacer que NetworkManager gestione solo CLARO

```bash
sudo nano /etc/NetworkManager/conf.d/10-manage-claro.conf
# contenido:
# [keyfile]
# unmanaged-devices=*,except:interface-name:CLARO
sudo systemctl restart NetworkManager
nmcli device status
```

**CLARO `connected (externally)`; `ens33` y `lo` siguen `unmanaged`**

![CLARO `connected (externally)`; `ens33` y `lo` siguen `unmanaged`](imagen/24_claro_gestionada.png)

### 2.3 Limpiar la configuración manual y crear la conexión

```bash
sudo ip addr flush dev CLARO
sudo ip route del default via 20.13.67.1 dev CLARO
sudo nmcli connection add type dummy ifname CLARO con-name CLARO ipv4.method manual ipv4.addresses 20.13.67.10/24 ipv4.gateway 20.13.67.1
sudo nmcli connection up CLARO
```

**`flush` correcto y `RTNETLINK answers: No such process`**

![`flush` correcto y `RTNETLINK answers: No such process`](imagen/25_flush_y_ruta.png)

**Conexión creada (aviso de nombre duplicado)**

![Conexión creada (aviso de nombre duplicado)](imagen/26_nmcli_add.png)

**Conexión activada**

![Conexión activada](imagen/27_nmcli_up.png)

### 2.4 Mostrar conexiones de NetworkManager

```bash
nmcli connection show
nmcli device status
ip a show CLARO
ip route show default
```

**Dos perfiles llamados CLARO**

![Dos perfiles llamados CLARO](imagen/28_connection_show_duplicado.png)

**CLARO `connected` ya sin *externally***

![CLARO `connected` ya sin *externally*](imagen/29_device_status_connected.png)

**IP con `noprefixroute` aplicada por NetworkManager**

![IP con `noprefixroute` aplicada por NetworkManager](imagen/30_ip_noprefixroute.png)

**Ruta default de CLARO con métrica 550**

![Ruta default de CLARO con métrica 550](imagen/31_ruta_metric_550.png)

### 2.5 Configuración manual de IP (verificación)

```bash
sudo nmcli connection delete d085c19c-16c4-4a50-907d-5d6d07ac9f0a
sudo nmcli connection up CLARO
nmcli connection show
nmcli connection show CLARO | grep -E "ipv4.method|ipv4.addresses|ipv4.gateway|ipv4.dns"
```

**Perfil único con IP manual**

![Perfil único con IP manual](imagen/32_perfil_unico_manual.png)

### 2.6 DNS manual con 8.8.8.8 y 1.1.1.1

```bash
sudo nmcli connection modify CLARO ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection show CLARO | grep ipv4.dns
```

**`ipv4.dns: 8.8.8.8,1.1.1.1`**

![`ipv4.dns: 8.8.8.8,1.1.1.1`](imagen/33_nmcli_dns_manual.png)

### 2.7 DHCP: permitir DNS manual (ignorar los del DHCP)

```bash
sudo nmcli connection modify CLARO ipv4.method auto ipv4.ignore-auto-dns yes
nmcli connection show CLARO | grep -E "ipv4.method|ignore-auto-dns|ipv4.dns:"
```

**`method auto` con `ignore-auto-dns yes`**

![`method auto` con `ignore-auto-dns yes`](imagen/34_dhcp_ignore_dns_yes.png)

### 2.8 DHCP + DNS manuales juntos

```bash
sudo nmcli connection modify CLARO ipv4.ignore-auto-dns no
nmcli connection show CLARO | grep -E "ipv4.method|ignore-auto-dns|ipv4.dns:"
```

**`method auto` con `ignore-auto-dns no`**

![`method auto` con `ignore-auto-dns no`](imagen/35_dhcp_ignore_dns_no.png)

### 2.9 Volver a manual y reiniciar la conexión

```bash
sudo nmcli connection modify CLARO ipv4.method manual ipv4.addresses 20.13.67.10/24 ipv4.gateway 20.13.67.1
sudo nmcli connection down CLARO && sudo nmcli connection up CLARO
nmcli device status
ip a show CLARO
ip route show default
```

**Perfil de nuevo en `manual`**

![Perfil de nuevo en `manual`](imagen/36_volver_a_manual.png)

**`down` y `up` correctos; CLARO conserva la IP**

![`down` y `up` correctos; CLARO conserva la IP](imagen/37_reiniciar_conexion.png)

**Ruta default de CLARO reaplicada (métrica 550)**

![Ruta default de CLARO reaplicada (métrica 550)](imagen/38_rutas_tras_reinicio.png)

### 2.10 Probar conectividad

```bash
resolvectl status CLARO
nslookup google.com
ping -c 4 20.13.67.10
ping -c 4 10.13.67.2
```

**DNS `8.8.8.8 1.1.1.1` en el link CLARO**

![DNS `8.8.8.8 1.1.1.1` en el link CLARO](imagen/39_resolvectl_claro.png)

**`nslookup google.com`**

![`nslookup google.com`](imagen/40_nslookup_claro.png)

**Ping a `20.13.67.10`, 0% pérdida**

![Ping a `20.13.67.10`, 0% pérdida](imagen/41_ping_ip_propia.png)

**Ping a `10.13.67.2`, 0% pérdida**

![Ping a `10.13.67.2`, 0% pérdida](imagen/42_ping_gateway.png)

---

## Parte 3 - DHCP y Netplan

### 3.1 Eliminar todas las direcciones IP estáticas

```bash
ip a show CLARO
sudo ip addr flush dev CLARO
ip a show CLARO
```

**CLARO con IP antes del `flush`**

![CLARO con IP antes del `flush`](imagen/43_dhcp_ip_antes.png)

**`ip addr flush dev CLARO`**

![`ip addr flush dev CLARO`](imagen/44_dhcp_flush.png)

**CLARO sin IPv4**

![CLARO sin IPv4](imagen/45_dhcp_ip_despues.png)

### 3.2 Configuración automática DHCP

```bash
sudo nmcli connection modify CLARO ipv4.addresses "" ipv4.gateway ""
# Error: ipv4.addresses: this property cannot be empty for 'method=manual'
sudo nmcli connection modify CLARO ipv4.method auto ipv4.addresses "" ipv4.gateway ""
nmcli connection show CLARO | grep -E "ipv4.method|ipv4.addresses|ipv4.gateway"
```

**`method auto`, sin direcciones ni gateway**

![`method auto`, sin direcciones ni gateway](imagen/46_dhcp_method_auto.png)

### 3.3 Reiniciar

```bash
sudo nmcli connection down CLARO && sudo nmcli connection up CLARO
nmcli device status
ip a show CLARO
ip route show default
```

**Timeout de DHCP al activar**

![Timeout de DHCP al activar](imagen/47_dhcp_reinicio_timeout.png)

**CLARO `disconnected`**

![CLARO `disconnected`](imagen/48_dhcp_device_disconnected.png)

**CLARO sin IP**

![CLARO sin IP](imagen/49_dhcp_sin_ip.png)

**Solo queda la default de `ens33`**

![Solo queda la default de `ens33`](imagen/50_dhcp_solo_ens33.png)

### 3.4 Netplan: respaldo y archivo actual

```bash
mkdir -p ~/netplan-backup
sudo cp /etc/netplan/*.yaml ~/netplan-backup/
ls ~/netplan-backup
cd /etc/netplan
ls
sudo cat 50-cloud-init.yaml
```

**Respaldo creado**

![Respaldo creado](imagen/51_netplan_respaldo.png)

**`ls` en `/etc/netplan`**

![`ls` en `/etc/netplan`](imagen/52_netplan_ls.png)

**Contenido de `50-cloud-init.yaml`**

![Contenido de `50-cloud-init.yaml`](imagen/53_netplan_cloud_init.png)

### 3.5 Netplan: crear el YAML

```bash
sudo nano 01-network-manager-all.yaml
sudo chmod 600 01-network-manager-all.yaml
ls -l
sudo cat 01-network-manager-all.yaml
```

**YAML en `nano`**

![YAML en `nano`](imagen/54_netplan_yaml_nano.png)

**Permisos `600`**

![Permisos `600`](imagen/55_netplan_permisos.png)

**`cat` del YAML**

![`cat` del YAML](imagen/56_netplan_yaml_cat.png)

### 3.6 Netplan: validar y aplicar

```bash
sudo netplan generate
sudo netplan try
```

**`netplan generate` y `netplan try`: *Configuration accepted***

![`netplan generate` y `netplan try`: *Configuration accepted*](imagen/57_netplan_try.png)

### 3.7 Netplan: verificar

```bash
ip a show CLARO
ip route show default
resolvectl status CLARO
networkctl status CLARO
nmcli device status
```

**IP, rutas y DNS aplicados por netplan**

![IP, rutas y DNS aplicados por netplan](imagen/58_netplan_verificacion.png)

**`networkctl status CLARO` con `10-netplan-CLARO.network`**

![`networkctl status CLARO` con `10-netplan-CLARO.network`](imagen/59_networkctl_status.png)

**Log de reconfiguración y `nmcli device status`**

![Log de reconfiguración y `nmcli device status`](imagen/60_networkctl_nmcli.png)

---

## Conclusión

Se configuraron parámetros de red en Linux de tres formas: con el comando `ip` (IP, rutas, gateway), con `resolvectl`/`networkctl` (DNS y DHCP) y con `nmcli` y Netplan de forma persistente. Todo se verificó con `ip`, `resolvectl`, `networkctl`, `nslookup` y `ping`, sin perder la conectividad de la interfaz real `ens33`.
