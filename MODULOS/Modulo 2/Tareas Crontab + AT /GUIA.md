<h1 align="center">📖 Guía – Práctica 2: Cron, reinicio automático y tarea AT</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Estudiante-Miguel%20Ram%C3%ADrez%20(Meli)-blue?style=for-the-badge" alt="Estudiante">
  <img src="https://img.shields.io/badge/Matr%C3%ADcula-20251367-green?style=for-the-badge" alt="Matrícula">
  <img src="https://img.shields.io/badge/Materia-Sistemas%20Operativos%203-orange?style=for-the-badge" alt="Materia">
  <img src="https://img.shields.io/badge/Maestro-Adri%C3%A1n%20Alc%C3%A1ntara-purple?style=for-the-badge" alt="Maestro">
  <img src="https://img.shields.io/badge/Instituci%C3%B3n-ITLA-red?style=for-the-badge" alt="ITLA">
</p>

Guía de los comandos usados en la práctica. Las capturas están en el [README.md](README.md).

---

## Índice

1. [Preparación](#1-preparación)
2. [Guía de los campos de cron](#2-guía-de-los-campos-de-cron)
3. [Cron: ver el servicio y el crontab](#3-cron-ver-el-servicio-y-el-crontab)
4. [Tarea 1: actualizar el sistema a las 11 PM](#4-tarea-1-actualizar-el-sistema-todos-los-días-a-las-11-pm)
5. [Tarea 2: reiniciar los domingos a las 3 AM](#5-tarea-2-reiniciar-los-domingos-a-las-3-am)
6. [Probar que cron funciona](#6-probar-que-cron-funciona)
7. [Tarea AT: borrar /tmp en 1 minuto](#7-tarea-at-borrar-tmp-en-1-minuto)
8. [Eliminar las tareas de cron](#8-eliminar-las-tareas-de-cron)
9. [Errores comunes](#9-errores-comunes)

---

## 1. Preparación

Comprobar que la máquina virtual tiene internet (hace falta para instalar `at`):

```bash
ip a
ip route
ping -c 3 8.8.8.8
```

`ip route` debe mostrar una línea `default via ...`. Si no aparece, la VM no tiene salida: en VMware usa el adaptador **NAT** (*Connected* y *Connect at power on*) y reinicia.

Actualizar el sistema:

```bash
sudo apt update && sudo apt upgrade -y
```

Ajustar la zona horaria (por defecto es UTC y las "11 PM" no coincidirían con la hora local):

```bash
sudo timedatectl set-timezone America/Santo_Domingo
timedatectl
```

## 2. Guía de los campos de cron

```
m  h  dom  mon  dow  comando
```

| Campo | Significado | Valores |
|---|---|---|
| `m` | Minuto en que se ejecuta | 0-59 |
| `h` | Hora exacta, formato 24 h (0 = medianoche) | 0-23 |
| `dom` | Día del mes. `15` = "el día 15"; para "cada 15 días" se usa `*/15` | 1-31 |
| `mon` | Mes del año | 1-12 |
| `dow` | Día de la semana, número o 3 letras en inglés (`mon`, `tue`, `wed`, `thu`, `fri`, `sat`, `sun`) | 0-6 (0 = domingo) |

`*` significa "cualquier valor".

| Comando | Qué hace |
|---|---|
| `crontab -e` | Editar el crontab |
| `crontab -l` | Listar las tareas |
| `crontab -r` | Borrar todo **sin preguntar** |
| `crontab -ri` | Borrar todo pidiendo confirmación |

## 3. Cron: ver el servicio y el crontab

```bash
sudo systemctl status cron
cat /etc/crontab
crontab -l
```

- `status` debe decir `active (running)`. Si cron no estuviera instalado: `sudo apt install cron -y && sudo systemctl enable --now cron`.
- `cat /etc/crontab` muestra el crontab del sistema (solo para consultar el formato).
- `crontab -l` dice `no crontab for miguel` si todavía no hay tareas; es normal.

## 4. Tarea 1: actualizar el sistema todos los días a las 11 PM

Se edita el crontab de **root** (`sudo crontab -e`), porque `apt` necesita permisos. Las líneas **no llevan `sudo`**: cron no puede escribir la contraseña.

```bash
sudo crontab -e
```

La primera vez elige **nano** (opción 1). Agrega al final:

```
0 23 * * * apt update && apt upgrade -y >> /var/log/actualizacion_cron.log 2>&1
```

Guardar: `Ctrl+O`, `Enter`. Salir: `Ctrl+X`.

## 5. Tarea 2: reiniciar los domingos a las 3 AM

```bash
sudo crontab -e
```

Agrega debajo de la anterior:

```
0 3 * * 0 /sbin/shutdown -r now
```

Verificar que están las dos tareas:

```bash
sudo crontab -l
```

## 6. Probar que cron funciona

Tarea temporal que corre cada minuto:

```bash
sudo crontab -e
```

```
* * * * * echo "Cron funcionando: $(date)" >> /tmp/prueba_cron.txt
```

Después de 1 o 2 minutos:

```bash
cat /tmp/prueba_cron.txt
sudo systemctl status cron
```

Al terminar, **borra solo esa línea** con `sudo crontab -e` (no uses `crontab -r`, borraría también las dos tareas reales) y elimina el archivo:

```bash
rm /tmp/prueba_cron.txt
sudo crontab -l
```

## 7. Tarea AT: borrar /tmp en 1 minuto

`at` ejecuta un comando **una sola vez**. Cron es para tareas repetitivas.

**Ver el contenido actual de `/tmp`** (se crean archivos de prueba para que el borrado se note):

```bash
touch /tmp/archivo1.txt /tmp/archivo2.txt /tmp/archivo3.txt
ls /tmp
```

**Instalar y activar `at`:**

```bash
sudo apt install at -y
sudo systemctl enable --now atd
systemctl status atd
```

**Entrar como root y programar la tarea:**

```bash
sudo su
at now + 1 minute
```

En el prompt `at>` escribe solo esto, presiona `Enter` y confirma con **Ctrl+D**:

```
rm -rf /tmp/*
```

Debe salir `<EOT>` y una línea `job N at ...`. Si no sale, no se programó.

**Verificar:**

```bash
atq
```

Pasado el minuto:

```bash
ls /tmp
atq
exit
```

`/tmp` queda vacío, `atq` sin tareas, y `exit` sale de root.

Otros comandos de `at`:

| Comando | Qué hace |
|---|---|
| `atq` | Lista las tareas pendientes |
| `atrm N` | Elimina la tarea `N` |
| `at -c N` | Muestra el contenido de la tarea `N` |
| `at 23:30` | Programa una tarea a una hora exacta |

> ⚠️ `rm -rf /tmp/*` es lo que pide la práctica; en una VM de pruebas no hay problema, pero en un servidor real borraría archivos temporales de programas en uso.

## 8. Eliminar las tareas de cron

```bash
sudo crontab -l
sudo crontab -ri
sudo crontab -l
```

Responde `y` a la confirmación. El resultado final es `no crontab for root`.

## 9. Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `Network is unreachable` al instalar `at` | La VM no tiene puerta de enlace | Usar adaptador NAT y reiniciar la VM |
| `Unit file atd.service does not exist` | `at` no está instalado | `sudo apt install at -y` con internet |
| El `at` no borró `/tmp` | Se salió del prompt con Ctrl+C (cancela) | Confirmar con **Ctrl+D** y comprobar que sale `job N` |
| La tarea de cron no hace nada | Se puso `sudo` dentro del crontab o se usó el crontab del usuario | Usar `sudo crontab -e` y quitar `sudo` de la línea |
| La hora de cron no coincide | Zona horaria en UTC | `sudo timedatectl set-timezone America/Santo_Domingo` |

[⬅ Volver al README](README.md)
