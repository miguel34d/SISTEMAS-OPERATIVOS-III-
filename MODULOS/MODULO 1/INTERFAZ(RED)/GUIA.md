# Guía – Práctica 2: Configuración de parámetros de red

**Entorno:** Ubuntu en VM (VMware) · usuario `miguel` · interfaz real `ens33` · interfaz virtual `CLARO` (dummy)

> **Importante:** casi todo se hizo sobre la interfaz virtual `CLARO` para no perder internet en `ens33`. Lo creado con `ip` a mano se pierde al reiniciar la VM.

## Índice
- Parte 1 – Comandos `ip`, rutas, DNS y DHCP
- Parte 2 – NMCLI
- Parte 3 – DHCP y Netplan
- Notas y problemas encontrados

---

## Parte 1 - Comandos `ip`, rutas, DNS y DHCP (práctica "prueba")

Se trabaja sobre una interfaz virtual llamada **CLARO** (tipo `dummy`) para no tocar la interfaz real `ens33`, que es la salida a internet de la VM.

### 1.1 Crear la interfaz CLARO y mostrar interfaces de red

**Explicación:** `modprobe dummy` carga el módulo del kernel; `ip link add ... type dummy` crea la interfaz virtual; `ip link set ... up` la levanta; `ip a show` y `ip link show` muestran su estado. El estado `UNKNOWN` es normal en interfaces dummy.

```bash
sudo modprobe dummy
sudo ip link add CLARO type dummy
sudo ip link set CLARO up
ip a show CLARO
ip link show CLARO
```

### 1.2 Agregar una IP manualmente

**Explicación:** Asigna la IP `20.13.67.10/24` a CLARO. El kernel crea automáticamente la ruta de red `20.13.67.0/24`.

```bash
sudo ip addr add 20.13.67.10/24 dev CLARO
ip a show CLARO
```

### 1.3 Eliminar y volver a agregar la IP

**Explicación:** `ip addr del` quita una IP específica. Para quitar todas se usa `ip addr flush dev CLARO`.

```bash
sudo ip addr del 20.13.67.10/24 dev CLARO
ip a show CLARO
sudo ip addr add 20.13.67.10/24 dev CLARO
ip a show CLARO
```

### 1.4 Mostrar rutas

**Explicación:** Muestra la tabla de rutas: la default por `ens33` (DHCP), la red de `ens33` y la red de CLARO creada por el kernel.

```bash
ip route
```

### 1.5 Agregar gateway

**Explicación:** Agrega una ruta por defecto por CLARO con métrica 200. Como la de `ens33` tiene métrica 100, esa sigue mandando y no se pierde internet.

```bash
sudo ip route add default via 20.13.67.1 dev CLARO metric 200
ip route
```

### 1.6 Eliminar el gateway

**Explicación:** Borra la ruta default de CLARO; queda solo la de `ens33`.

```bash
sudo ip route del default via 20.13.67.1 dev CLARO
ip route
```

### 1.7 DEFAULT (volver a poner el gateway por defecto)

**Explicación:** Restablece la ruta default de CLARO y la muestra filtrando solo las rutas por defecto.

```bash
sudo ip route add default via 20.13.67.1 dev CLARO metric 200
ip route | grep default
ip route show default
```

### 1.8 Editar DNS

**Explicación:** En Ubuntu, `/etc/resolv.conf` es un enlace a `systemd-resolved` y dice *Do not edit*. Por eso el DNS se cambia con `resolvectl`. El stub `127.0.0.53` reenvía las consultas a los DNS configurados.

```bash
cat /etc/resolv.conf
resolvectl status
sudo resolvectl dns ens33 8.8.8.8 1.1.1.1
resolvectl status ens33
cat /etc/resolv.conf
```

### 1.9 Probar el DNS sin depender de ICMP

**Explicación:** Como el ping a internet está bloqueado, se prueba el DNS con `resolvectl query` y `nslookup`, y la red local con un ping al gateway real.

```bash
resolvectl query google.com
nslookup google.com
ping -c 4 10.13.67.2
```

### 1.10 Renovar DHCP

**Explicación:** `dhclient -r` **no aplica** en este Ubuntu porque el DHCP lo maneja `systemd-networkd`: no libera nada y el lease solo baja con el tiempo (1396 s → 1365 s). El método correcto es `networkctl renew`, que reinicia el lease (1365 s → 1769 s). La IP y los DNS manuales se mantienen.

```bash
ip a show ens33
sudo dhclient -r ens33
ip a show ens33
sudo networkctl renew ens33
ip a show ens33
resolvectl status ens33
ip route show default
```

---

## Parte 2 - NMCLI (NetworkManager)

NetworkManager no venía instalado y, una vez instalado, veía todas las interfaces como `unmanaged` porque netplan usa `networkd`. Para no tocar `ens33`, se configuró NetworkManager para gestionar **solo CLARO**.

### 2.1 Instalar NetworkManager

**Explicación:** Se instala la versión de `apt` (mejor integrada que la de snap). Tras instalar, las tres interfaces salen `unmanaged`.

```bash
nmcli device status          # nmcli no encontrado
sudo apt update
sudo apt install network-manager -y
nmcli --version
nmcli device status
```

### 2.2 Hacer que NetworkManager gestione solo CLARO

**Explicación:** Se ignoran todas las interfaces excepto CLARO. Así `ens33` no se toca y la VM conserva internet. CLARO pasa a `connected (externally)` porque ya tenía IP puesta a mano.

```bash
sudo nano /etc/NetworkManager/conf.d/10-manage-claro.conf
# contenido:
# [keyfile]
# unmanaged-devices=*,except:interface-name:CLARO
sudo systemctl restart NetworkManager
nmcli device status
```

### 2.3 Limpiar la configuración manual y crear la conexión

**Explicación:** `flush` borra la IP y con ella la ruta (por eso `del` da *No such process*, que no es un error real). Después se crea el perfil de NetworkManager con IP manual y gateway.

```bash
sudo ip addr flush dev CLARO
sudo ip route del default via 20.13.67.1 dev CLARO
sudo nmcli connection add type dummy ifname CLARO con-name CLARO ipv4.method manual ipv4.addresses 20.13.67.10/24 ipv4.gateway 20.13.67.1
sudo nmcli connection up CLARO
```

### 2.4 Mostrar conexiones de NetworkManager

**Explicación:** Había dos perfiles con el nombre CLARO: el automático (*externally*) y el nuevo. Se borró el automático por UUID y se dejó uno solo. NetworkManager aplica la IP con `noprefixroute` y la ruta default con `proto static metric 550`.

```bash
nmcli connection show
nmcli device status
ip a show CLARO
ip route show default
```

### 2.5 Configuración manual de IP (verificación)

**Explicación:** Queda un solo perfil con `ipv4.method manual`, IP `20.13.67.10/24` y gateway `20.13.67.1`.

```bash
sudo nmcli connection delete d085c19c-16c4-4a50-907d-5d6d07ac9f0a
sudo nmcli connection up CLARO
nmcli connection show
nmcli connection show CLARO | grep -E "ipv4.method|ipv4.addresses|ipv4.gateway|ipv4.dns"
```

### 2.6 DNS manual con 8.8.8.8 y 1.1.1.1

**Explicación:** `modify` solo guarda el cambio en el perfil; se aplica al reiniciar la conexión.

```bash
sudo nmcli connection modify CLARO ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection show CLARO | grep ipv4.dns
```

### 2.7 DHCP: permitir DNS manual (ignorar los del DHCP)

**Explicación:** La conexión pide IP por DHCP pero ignora los DNS del servidor y usa solo los manuales.

```bash
sudo nmcli connection modify CLARO ipv4.method auto ipv4.ignore-auto-dns yes
nmcli connection show CLARO | grep -E "ipv4.method|ignore-auto-dns|ipv4.dns:"
```

### 2.8 DHCP + DNS manuales juntos

**Explicación:** Con `ignore-auto-dns no` se usan los DNS del DHCP **y** los manuales.

```bash
sudo nmcli connection modify CLARO ipv4.ignore-auto-dns no
nmcli connection show CLARO | grep -E "ipv4.method|ignore-auto-dns|ipv4.dns:"
```

### 2.9 Volver a manual y reiniciar la conexión

**Explicación:** Como CLARO es dummy y no hay servidor DHCP, se vuelve a `manual` antes de reiniciar para no perder la IP.

```bash
sudo nmcli connection modify CLARO ipv4.method manual ipv4.addresses 20.13.67.10/24 ipv4.gateway 20.13.67.1
sudo nmcli connection down CLARO && sudo nmcli connection up CLARO
nmcli device status
ip a show CLARO
ip route show default
```

### 2.10 Probar conectividad

**Explicación:** Se verifica que el DNS llegó al sistema, que resuelve nombres, que CLARO responde y que la red real sigue bien.

```bash
resolvectl status CLARO
nslookup google.com
ping -c 4 20.13.67.10
ping -c 4 10.13.67.2
```

---

## Parte 3 - DHCP y Netplan

Se eliminan las IP estáticas, se deja CLARO en DHCP y luego se configura por Netplan con un archivo YAML propio, sin tocar `50-cloud-init.yaml` (que maneja `ens33`).

### 3.1 Eliminar todas las direcciones IP estáticas

**Explicación:** `flush` elimina todas las IP de la interfaz. Esto limpia el sistema, pero el perfil de NetworkManager todavía guarda la IP.

```bash
ip a show CLARO
sudo ip addr flush dev CLARO
ip a show CLARO
```

### 3.2 Configuración automática DHCP

**Explicación:** Con `method manual` no se pueden vaciar las direcciones (da error). Hay que cambiar a `auto` y limpiar en el **mismo comando**.

```bash
sudo nmcli connection modify CLARO ipv4.addresses "" ipv4.gateway ""
# Error: ipv4.addresses: this property cannot be empty for 'method=manual'
sudo nmcli connection modify CLARO ipv4.method auto ipv4.addresses "" ipv4.gateway ""
nmcli connection show CLARO | grep -E "ipv4.method|ipv4.addresses|ipv4.gateway"
```

### 3.3 Reiniciar

**Explicación:** Es el resultado esperado: CLARO es dummy y no hay servidor DHCP, así que la activación falla por timeout (`IP configuration could not be reserved`). Queda `disconnected`, sin IP, y solo se conserva la default de `ens33`.

```bash
sudo nmcli connection down CLARO && sudo nmcli connection up CLARO
nmcli device status
ip a show CLARO
ip route show default
```

### 3.4 Netplan: respaldo y archivo actual

**Explicación:** Antes de tocar nada se respalda. El archivo existente `50-cloud-init.yaml` configura `ens33` por DHCP y no se edita.

```bash
mkdir -p ~/netplan-backup
sudo cp /etc/netplan/*.yaml ~/netplan-backup/
ls ~/netplan-backup
cd /etc/netplan
ls
sudo cat 50-cloud-init.yaml
```

### 3.5 Netplan: crear el YAML

**Explicación:** Se declara solo CLARO. `gateway4` está obsoleto, por eso se usa `routes`. El permiso 600 evita el aviso de netplan por archivo legible por todos. La indentación debe ser con espacios.

```bash
sudo nano 01-network-manager-all.yaml
sudo chmod 600 01-network-manager-all.yaml
ls -l
sudo cat 01-network-manager-all.yaml
```

Archivo `/etc/netplan/01-network-manager-all.yaml`:

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

**Explicación:** `generate` valida el YAML. `try` aplica y revierte solo si no se confirma con Enter. El aviso `Cannot call Open vSwitch: ovsdb-server.service is not running` es inofensivo (no hay Open vSwitch instalado).

```bash
sudo netplan generate
sudo netplan try
```

### 3.7 Netplan: verificar

**Explicación:** La IP, la ruta (métrica 200), el DNS y el archivo `10-netplan-CLARO.network` demuestran que la configuración viene de netplan. NetworkManager muestra CLARO como `connected (externally)`, sin conflicto.

```bash
ip a show CLARO
ip route show default
resolvectl status CLARO
networkctl status CLARO
nmcli device status
```

---

## Notas y problemas encontrados

| Situación | Causa | Solución |
|---|---|---|
| `dhclient -r` no hace nada | El DHCP lo gestiona `systemd-networkd` | Usar `sudo networkctl renew ens33` |
| `/etc/resolv.conf` dice *Do not edit* | Es un enlace a `systemd-resolved` | Usar `resolvectl dns <interfaz> ...` |
| `ping` a internet con 100% de pérdida | ICMP bloqueado por el NAT de la VM | Probar con `resolvectl query`, `nslookup` y ping al gateway |
| `nmcli: command not found` | NetworkManager no instalado | `sudo apt install network-manager` |
| Interfaces `unmanaged` en nmcli | Netplan usa `renderer: networkd` | Archivo `.conf` con `unmanaged-devices=*,except:interface-name:CLARO` |
| `RTNETLINK answers: No such process` | `flush` ya había borrado la ruta | No es error real |
| Dos perfiles llamados CLARO | El perfil automático *externally* no se había borrado | `nmcli connection delete <UUID>` |
| `ipv4.addresses cannot be empty for 'method=manual'` | No se pueden vaciar IPs con método manual | Cambiar a `auto` y limpiar en el mismo comando |
| `IP configuration could not be reserved` | CLARO es dummy, no hay servidor DHCP | Es el comportamiento esperado |
| `Cannot call Open vSwitch...` en `netplan try` | Open vSwitch no está instalado | Aviso inofensivo |
| `gateway4` obsoleto en netplan | Sustituido en versiones nuevas | Usar `routes: - to: default via: ...` |

## Limpieza (opcional)

```bash
sudo rm /etc/netplan/01-network-manager-all.yaml
sudo netplan apply
sudo nmcli connection delete CLARO
sudo rm /etc/NetworkManager/conf.d/10-manage-claro.conf
sudo ip link del CLARO
```
