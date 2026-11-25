<div align="center">

# 🌐 Guía – Práctica 2
## Configuración de Parámetros de Red

![Alumno](https://img.shields.io/badge/Alumno-Miguel%20Ram%C3%ADrez%20Meli-2563eb?style=for-the-badge&logo=github&logoColor=white)
![Matrícula](https://img.shields.io/badge/Matr%C3%ADcula-2025--1367-16a34a?style=for-the-badge)
![ITLA](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-ea580c?style=for-the-badge)
![Materia](https://img.shields.io/badge/Materia-Sistemas%20Operativos%20III-7c3aed?style=for-the-badge&logo=linux&logoColor=white)
![Profesor](https://img.shields.io/badge/Profesor-Adri%C3%A1n%20Alc%C3%A1ntara-db2777?style=for-the-badge)
![Práctica](https://img.shields.io/badge/Pr%C3%A1ctica-2%20Par%C3%A1metros%20de%20Red-0891b2?style=for-the-badge)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%20(VMware)-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

</div>

---

## 🪪 Datos generales

| | |
|---|---|
| 👨‍🎓 **Alumno** | Miguel Ramírez Meli |
| 🆔 **Matrícula** | 2025-1367 |
| 🏫 **Institución** | ITLA |
| 📚 **Materia** | Sistemas Operativos III |
| 👨‍🏫 **Profesor** | Adrián Alcántara |
| 🖥️ **Entorno** | Ubuntu en VMware · usuario `miguel` |
| 🔗 **Interfaces** | `ens33` (real, DHCP) · `CLARO` (virtual, dummy) |

> [!IMPORTANT]
> Casi todo se hizo sobre la interfaz virtual **CLARO** para no perder internet en `ens33`.
> Lo creado a mano con `ip` se pierde al reiniciar la VM.

## 🎯 Objetivo

Configurar y verificar parámetros de red en Linux: direcciones IP, rutas, gateway, DNS y DHCP, usando `ip`, `resolvectl`/`networkctl`, `nmcli` y Netplan.

## 🗺️ Resumen

| Parte | Tema | Herramientas |
|:-:|---|---|
| 1️⃣ | IP, rutas, gateway, DNS y renovar DHCP | `ip`, `resolvectl`, `networkctl` |
| 2️⃣ | Gestión con NetworkManager | `nmcli` |
| 3️⃣ | DHCP automático y configuración persistente | `nmcli`, `netplan` |

## 📑 Índice

- **Parte 1 - Comandos ip, rutas, DNS y DHCP (práctica "prueba")**
  - [1.1 Crear la interfaz CLARO y mostrar interfaces de red](#11-crear-la-interfaz-claro-y-mostrar-interfaces-de-red)
  - [1.2 Agregar una IP manualmente](#12-agregar-una-ip-manualmente)
  - [1.3 Eliminar y volver a agregar la IP](#13-eliminar-y-volver-a-agregar-la-ip)
  - [1.4 Mostrar rutas](#14-mostrar-rutas)
  - [1.5 Agregar gateway](#15-agregar-gateway)
  - [1.6 Eliminar el gateway](#16-eliminar-el-gateway)
  - [1.7 DEFAULT (volver a poner el gateway por defecto)](#17-default-volver-a-poner-el-gateway-por-defecto)
  - [1.8 Editar DNS](#18-editar-dns)
  - [1.9 Probar el DNS sin depender de ICMP](#19-probar-el-dns-sin-depender-de-icmp)
  - [1.10 Renovar DHCP](#110-renovar-dhcp)
- **Parte 2 - NMCLI (NetworkManager)**
  - [2.1 Instalar NetworkManager](#21-instalar-networkmanager)
  - [2.2 Hacer que NetworkManager gestione solo CLARO](#22-hacer-que-networkmanager-gestione-solo-claro)
  - [2.3 Limpiar la configuración manual y crear la conexión](#23-limpiar-la-configuración-manual-y-crear-la-conexión)
  - [2.4 Mostrar conexiones de NetworkManager](#24-mostrar-conexiones-de-networkmanager)
  - [2.5 Configuración manual de IP (verificación)](#25-configuración-manual-de-ip-verificación)
  - [2.6 DNS manual con 8.8.8.8 y 1.1.1.1](#26-dns-manual-con-8888-y-1111)
  - [2.7 DHCP: permitir DNS manual (ignorar los del DHCP)](#27-dhcp-permitir-dns-manual-ignorar-los-del-dhcp)
  - [2.8 DHCP + DNS manuales juntos](#28-dhcp--dns-manuales-juntos)
  - [2.9 Volver a manual y reiniciar la conexión](#29-volver-a-manual-y-reiniciar-la-conexión)
  - [2.10 Probar conectividad](#210-probar-conectividad)
- **Parte 3 - DHCP y Netplan**
  - [3.1 Eliminar todas las direcciones IP estáticas](#31-eliminar-todas-las-direcciones-ip-estáticas)
  - [3.2 Configuración automática DHCP](#32-configuración-automática-dhcp)
  - [3.3 Reiniciar](#33-reiniciar)
  - [3.4 Netplan: respaldo y archivo actual](#34-netplan-respaldo-y-archivo-actual)
  - [3.5 Netplan: crear el YAML](#35-netplan-crear-el-yaml)
  - [3.6 Netplan: validar y aplicar](#36-netplan-validar-y-aplicar)
  - [3.7 Netplan: verificar](#37-netplan-verificar)
- [Problemas encontrados y soluciones](#-problemas-encontrados-y-soluciones)
- [Limpieza](#-limpieza-opcional)

---

## 🔌 Parte 1 - Comandos `ip`, rutas, DNS y DHCP (práctica "prueba")

> Se trabaja sobre una interfaz virtual llamada **CLARO** (tipo `dummy`) para no tocar la interfaz real `ens33`, que es la salida a internet de la VM.

### 1.1 Crear la interfaz CLARO y mostrar interfaces de red

💡 `modprobe dummy` carga el módulo del kernel; `ip link add ... type dummy` crea la interfaz virtual; `ip link set ... up` la levanta; `ip a show` y `ip link show` muestran su estado. El estado `UNKNOWN` es normal en interfaces dummy.

```bash
# Carga el módulo dummy del kernel
sudo modprobe dummy
# Crea la interfaz virtual CLARO
sudo ip link add CLARO type dummy
# Levanta (activa) la interfaz
sudo ip link set CLARO up
# Muestra IP y estado de CLARO
ip a show CLARO
# Muestra el enlace de CLARO (MAC, estado)
ip link show CLARO
```

### 1.2 Agregar una IP manualmente

💡 Asigna la IP `20.13.67.10/24` a CLARO. El kernel crea automáticamente la ruta de red `20.13.67.0/24`.

```bash
# Agrega la IP manualmente
sudo ip addr add 20.13.67.10/24 dev CLARO
# Muestra IP y estado de CLARO
ip a show CLARO
```

### 1.3 Eliminar y volver a agregar la IP

💡 `ip addr del` quita una IP específica. Para quitar todas se usa `ip addr flush dev CLARO`.

```bash
# Elimina esa IP
sudo ip addr del 20.13.67.10/24 dev CLARO
# Muestra IP y estado de CLARO
ip a show CLARO
# Agrega la IP manualmente
sudo ip addr add 20.13.67.10/24 dev CLARO
# Muestra IP y estado de CLARO
ip a show CLARO
```

### 1.4 Mostrar rutas

💡 Muestra la tabla de rutas: la default por `ens33` (DHCP), la red de `ens33` y la red de CLARO creada por el kernel.

```bash
# Muestra la tabla de rutas
ip route
```

### 1.5 Agregar gateway

💡 Agrega una ruta por defecto por CLARO con métrica 200. Como la de `ens33` tiene métrica 100, esa sigue mandando y no se pierde internet.

```bash
# Agrega el gateway por defecto (métrica 200: ens33 sigue mandando)
sudo ip route add default via 20.13.67.1 dev CLARO metric 200
# Muestra la tabla de rutas
ip route
```

### 1.6 Eliminar el gateway

💡 Borra la ruta default de CLARO; queda solo la de `ens33`.

```bash
# Elimina el gateway de CLARO
sudo ip route del default via 20.13.67.1 dev CLARO
# Muestra la tabla de rutas
ip route
```

### 1.7 DEFAULT (volver a poner el gateway por defecto)

💡 Restablece la ruta default de CLARO y la muestra filtrando solo las rutas por defecto.

```bash
# Agrega el gateway por defecto (métrica 200: ens33 sigue mandando)
sudo ip route add default via 20.13.67.1 dev CLARO metric 200
# Filtra solo las rutas default
ip route | grep default
# Muestra solo las rutas por defecto
ip route show default
```

### 1.8 Editar DNS

💡 En Ubuntu, `/etc/resolv.conf` es un enlace a `systemd-resolved` y dice *Do not edit*. Por eso el DNS se cambia con `resolvectl`. El stub `127.0.0.53` reenvía las consultas a los DNS configurados.

```bash
# Muestra el archivo de DNS
cat /etc/resolv.conf
# Estado DNS de todas las interfaces
resolvectl status
# Cambia los DNS de ens33
sudo resolvectl dns ens33 8.8.8.8 1.1.1.1
# Estado DNS de ens33
resolvectl status ens33
# Muestra el archivo de DNS
cat /etc/resolv.conf
```

### 1.9 Probar el DNS sin depender de ICMP

💡 Como el ping a internet está bloqueado, se prueba el DNS con `resolvectl query` y `nslookup`, y la red local con un ping al gateway real.

```bash
# Resuelve un nombre usando el DNS
resolvectl query google.com
# Consulta DNS clásica
nslookup google.com
# Ping al gateway real (4 paquetes)
ping -c 4 10.13.67.2
```

### 1.10 Renovar DHCP

💡 `dhclient -r` **no aplica** en este Ubuntu porque el DHCP lo maneja `systemd-networkd`: no libera nada y el lease solo baja con el tiempo (1396 s → 1365 s). El método correcto es `networkctl renew`, que reinicia el lease (1365 s → 1769 s). La IP y los DNS manuales se mantienen.

```bash
# Muestra IP y lease DHCP de ens33
ip a show ens33
# Intenta liberar el lease (no aplica: usa systemd-networkd)
sudo dhclient -r ens33
# Muestra IP y lease DHCP de ens33
ip a show ens33
# Renueva el lease DHCP (método correcto)
sudo networkctl renew ens33
# Muestra IP y lease DHCP de ens33
ip a show ens33
# Estado DNS de ens33
resolvectl status ens33
# Muestra solo las rutas por defecto
ip route show default
```

---

## 🛠️ Parte 2 - NMCLI (NetworkManager)

> NetworkManager no venía instalado y, una vez instalado, veía todas las interfaces como `unmanaged` porque netplan usa `networkd`. Para no tocar `ens33`, se configuró NetworkManager para gestionar **solo CLARO**.

### 2.1 Instalar NetworkManager

💡 Se instala la versión de `apt` (mejor integrada que la de snap). Tras instalar, las tres interfaces salen `unmanaged`.

```bash
nmcli device status          # nmcli no encontrado
# Actualiza la lista de paquetes
sudo apt update
# Instala NetworkManager
sudo apt install network-manager -y
# Verifica la instalación
nmcli --version
# Muestra el estado de las interfaces en NetworkManager
nmcli device status
```

### 2.2 Hacer que NetworkManager gestione solo CLARO

💡 Se ignoran todas las interfaces excepto CLARO. Así `ens33` no se toca y la VM conserva internet. CLARO pasa a `connected (externally)` porque ya tenía IP puesta a mano.

```bash
# Crea el archivo que hace gestionar solo CLARO
sudo nano /etc/NetworkManager/conf.d/10-manage-claro.conf
# contenido:
# [keyfile]
# unmanaged-devices=*,except:interface-name:CLARO
# Reinicia NetworkManager
sudo systemctl restart NetworkManager
# Muestra el estado de las interfaces en NetworkManager
nmcli device status
```

### 2.3 Limpiar la configuración manual y crear la conexión

💡 `flush` borra la IP y con ella la ruta (por eso `del` da *No such process*, que no es un error real). Después se crea el perfil de NetworkManager con IP manual y gateway.

```bash
# Elimina TODAS las IP de la interfaz
sudo ip addr flush dev CLARO
# Elimina el gateway de CLARO
sudo ip route del default via 20.13.67.1 dev CLARO
# Crea la conexión con IP y gateway manuales
sudo nmcli connection add type dummy ifname CLARO con-name CLARO ipv4.method manual ipv4.addresses 20.13.67.10/24 ipv4.gateway 20.13.67.1
# Activa la conexión
sudo nmcli connection up CLARO
```

### 2.4 Mostrar conexiones de NetworkManager

💡 Había dos perfiles con el nombre CLARO: el automático (*externally*) y el nuevo. Se borró el automático por UUID y se dejó uno solo. NetworkManager aplica la IP con `noprefixroute` y la ruta default con `proto static metric 550`.

```bash
# Muestra las conexiones de NetworkManager
nmcli connection show
# Muestra el estado de las interfaces en NetworkManager
nmcli device status
# Muestra IP y estado de CLARO
ip a show CLARO
# Muestra solo las rutas por defecto
ip route show default
```

### 2.5 Configuración manual de IP (verificación)

💡 Queda un solo perfil con `ipv4.method manual`, IP `20.13.67.10/24` y gateway `20.13.67.1`.

```bash
sudo nmcli connection delete d085c19c-16c4-4a50-907d-5d6d07ac9f0a
# Activa la conexión
sudo nmcli connection up CLARO
# Muestra las conexiones de NetworkManager
nmcli connection show
nmcli connection show CLARO | grep -E "ipv4.method|ipv4.addresses|ipv4.gateway|ipv4.dns"
```

### 2.6 DNS manual con 8.8.8.8 y 1.1.1.1

💡 `modify` solo guarda el cambio en el perfil; se aplica al reiniciar la conexión.

```bash
# Asigna DNS manuales
sudo nmcli connection modify CLARO ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection show CLARO | grep ipv4.dns
```

### 2.7 DHCP: permitir DNS manual (ignorar los del DHCP)

💡 La conexión pide IP por DHCP pero ignora los DNS del servidor y usa solo los manuales.

```bash
# DHCP e ignora los DNS del servidor (solo manuales)
sudo nmcli connection modify CLARO ipv4.method auto ipv4.ignore-auto-dns yes
nmcli connection show CLARO | grep -E "ipv4.method|ignore-auto-dns|ipv4.dns:"
```

### 2.8 DHCP + DNS manuales juntos

💡 Con `ignore-auto-dns no` se usan los DNS del DHCP **y** los manuales.

```bash
# Usa DNS del DHCP y los manuales
sudo nmcli connection modify CLARO ipv4.ignore-auto-dns no
nmcli connection show CLARO | grep -E "ipv4.method|ignore-auto-dns|ipv4.dns:"
```

### 2.9 Volver a manual y reiniciar la conexión

💡 Como CLARO es dummy y no hay servidor DHCP, se vuelve a `manual` antes de reiniciar para no perder la IP.

```bash
# Vuelve a IP manual
sudo nmcli connection modify CLARO ipv4.method manual ipv4.addresses 20.13.67.10/24 ipv4.gateway 20.13.67.1
# Reinicia la conexión (down + up)
sudo nmcli connection down CLARO && sudo nmcli connection up CLARO
# Muestra el estado de las interfaces en NetworkManager
nmcli device status
# Muestra IP y estado de CLARO
ip a show CLARO
# Muestra solo las rutas por defecto
ip route show default
```

### 2.10 Probar conectividad

💡 Se verifica que el DNS llegó al sistema, que resuelve nombres, que CLARO responde y que la red real sigue bien.

```bash
# Estado DNS de CLARO
resolvectl status CLARO
# Consulta DNS clásica
nslookup google.com
# Ping a la IP propia de CLARO
ping -c 4 20.13.67.10
# Ping al gateway real (4 paquetes)
ping -c 4 10.13.67.2
```

---

## 📄 Parte 3 - DHCP y Netplan

> Se eliminan las IP estáticas, se deja CLARO en DHCP y luego se configura por Netplan con un archivo YAML propio, sin tocar `50-cloud-init.yaml` (que maneja `ens33`).

### 3.1 Eliminar todas las direcciones IP estáticas

💡 `flush` elimina todas las IP de la interfaz. Esto limpia el sistema, pero el perfil de NetworkManager todavía guarda la IP.

```bash
# Muestra IP y estado de CLARO
ip a show CLARO
# Elimina TODAS las IP de la interfaz
sudo ip addr flush dev CLARO
# Muestra IP y estado de CLARO
ip a show CLARO
```

### 3.2 Configuración automática DHCP

💡 Con `method manual` no se pueden vaciar las direcciones (da error). Hay que cambiar a `auto` y limpiar en el **mismo comando**.

```bash
sudo nmcli connection modify CLARO ipv4.addresses "" ipv4.gateway ""
# Error: ipv4.addresses: this property cannot be empty for 'method=manual'
# Pasa a DHCP y borra IP y gateway estáticos
sudo nmcli connection modify CLARO ipv4.method auto ipv4.addresses "" ipv4.gateway ""
nmcli connection show CLARO | grep -E "ipv4.method|ipv4.addresses|ipv4.gateway"
```

### 3.3 Reiniciar

💡 Es el resultado esperado: CLARO es dummy y no hay servidor DHCP, así que la activación falla por timeout (`IP configuration could not be reserved`). Queda `disconnected`, sin IP, y solo se conserva la default de `ens33`.

```bash
# Reinicia la conexión (down + up)
sudo nmcli connection down CLARO && sudo nmcli connection up CLARO
# Muestra el estado de las interfaces en NetworkManager
nmcli device status
# Muestra IP y estado de CLARO
ip a show CLARO
# Muestra solo las rutas por defecto
ip route show default
```

### 3.4 Netplan: respaldo y archivo actual

💡 Antes de tocar nada se respalda. El archivo existente `50-cloud-init.yaml` configura `ens33` por DHCP y no se edita.

```bash
# Crea la carpeta de respaldo
mkdir -p ~/netplan-backup
# Respalda los YAML de netplan
sudo cp /etc/netplan/*.yaml ~/netplan-backup/
# Verifica el respaldo
ls ~/netplan-backup
# Entra a la carpeta de netplan
cd /etc/netplan
# Lista los archivos
ls
# Muestra el YAML original (ens33 por DHCP)
sudo cat 50-cloud-init.yaml
```

### 3.5 Netplan: crear el YAML

💡 Se declara solo CLARO. `gateway4` está obsoleto, por eso se usa `routes`. El permiso 600 evita el aviso de netplan por archivo legible por todos. La indentación debe ser con espacios.

```bash
# Crea el YAML nuevo (solo CLARO)
sudo nano 01-network-manager-all.yaml
# Permisos solo para root (evita aviso)
sudo chmod 600 01-network-manager-all.yaml
# Lista con permisos
ls -l
# Muestra el YAML guardado
sudo cat 01-network-manager-all.yaml
```

📄 Contenido de `/etc/netplan/01-network-manager-all.yaml`:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    CLARO:
      dhcp4: no
      addresses:
        - 20.13.67.10/24
      routes:
        - to: default
          via: 20.13.67.1
          metric: 200
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

### 3.6 Netplan: validar y aplicar

💡 `generate` valida el YAML. `try` aplica y revierte solo si no se confirma con Enter. El aviso `Cannot call Open vSwitch: ovsdb-server.service is not running` es inofensivo (no hay Open vSwitch instalado).

```bash
# Valida el YAML y genera la configuración
sudo netplan generate
# Aplica y revierte solo si no se confirma con Enter
sudo netplan try
```

### 3.7 Netplan: verificar

💡 La IP, la ruta (métrica 200), el DNS y el archivo `10-netplan-CLARO.network` demuestran que la configuración viene de netplan. NetworkManager muestra CLARO como `connected (externally)`, sin conflicto.

```bash
# Muestra IP y estado de CLARO
ip a show CLARO
# Muestra solo las rutas por defecto
ip route show default
# Estado DNS de CLARO
resolvectl status CLARO
# Estado de CLARO en systemd-networkd
networkctl status CLARO
# Muestra el estado de las interfaces en NetworkManager
nmcli device status
```

---

## 🧯 Problemas encontrados y soluciones

| ⚠️ Situación | 🔍 Causa | ✅ Solución |
|---|---|---|
| `dhclient -r` no hace nada | El DHCP lo gestiona `systemd-networkd` | `sudo networkctl renew ens33` |
| `/etc/resolv.conf` dice *Do not edit* | Es un enlace a `systemd-resolved` | `resolvectl dns <interfaz> ...` |
| `ping` a internet con 100% de pérdida | ICMP bloqueado por el NAT de la VM | Probar con `resolvectl query`, `nslookup` y ping al gateway |
| `nmcli: command not found` | NetworkManager no instalado | `sudo apt install network-manager` |
| Interfaces `unmanaged` en nmcli | Netplan usa `renderer: networkd` | `.conf` con `unmanaged-devices=*,except:interface-name:CLARO` |
| `RTNETLINK answers: No such process` | `flush` ya había borrado la ruta | No es error real |
| Dos perfiles llamados CLARO | El perfil automático *externally* seguía existiendo | `nmcli connection delete <UUID>` |
| `ipv4.addresses cannot be empty for 'method=manual'` | No se pueden vaciar IPs con método manual | Cambiar a `auto` y limpiar en el mismo comando |
| `IP configuration could not be reserved` | CLARO es dummy, no hay servidor DHCP | Comportamiento esperado |
| `Cannot call Open vSwitch...` en `netplan try` | Open vSwitch no instalado | Aviso inofensivo |
| `gateway4` obsoleto | Reemplazado en netplan moderno | Usar `routes: - to: default via: ...` |

## 🧹 Limpieza (opcional)

```bash
sudo rm /etc/netplan/01-network-manager-all.yaml
sudo netplan apply
sudo nmcli connection delete CLARO
sudo rm /etc/NetworkManager/conf.d/10-manage-claro.conf
sudo ip link del CLARO
```

---

<div align="center">

**Miguel Ramírez Meli · 2025-1367 · ITLA**
Sistemas Operativos III · Prof. Adrián Alcántara

</div>
