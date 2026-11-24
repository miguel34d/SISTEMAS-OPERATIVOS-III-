<h1 align="center">📖 Guía – Práctica 2: Cron, reinicio automático y tarea AT</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Estudiante-Miguel%20Ram%C3%ADrez%20(Meli)-blue?style=for-the-badge" alt="Estudiante">
  <img src="https://img.shields.io/badge/Matr%C3%ADcula-20251367-green?style=for-the-badge" alt="Matrícula">
  <img src="https://img.shields.io/badge/Materia-Sistemas%20Operativos%203-orange?style=for-the-badge" alt="Materia">
  <img src="https://img.shields.io/badge/Maestro-Adri%C3%A1n%20Alc%C3%A1ntara-purple?style=for-the-badge" alt="Maestro">
  <img src="https://img.shields.io/badge/Instituci%C3%B3n-ITLA-red?style=for-the-badge" alt="ITLA">
</p>

**Institución:** ITLA
**Materia:** Sistemas Operativos 3
**Profesor:** Adrián Alcántara
**Estudiante:** Miguel Ramírez (Meli) · 20251367

---

## Índice

1. [Requisitos](#requisitos)
2. [Guía de los campos de cron](#guía-de-los-campos-de-cron)
3. [Parte 1: Verificar cron y zona horaria](#parte-1-verificar-cron-y-zona-horaria)
4. [Parte 2: Actualización diaria a las 11 PM](#parte-2-actualización-diaria-a-las-11-pm)
5. [Parte 3: Reinicio los domingos a las 3 AM](#parte-3-reinicio-los-domingos-a-las-3-am)
6. [Parte 4: Probar que cron funciona](#parte-4-probar-que-cron-funciona)
7. [Parte 5: Tarea con AT](#parte-5-tarea-con-at)
8. [Parte 6: Eliminar tareas de cron](#parte-6-eliminar-tareas-de-cron)
9. [Problemas encontrados y soluciones](#problemas-encontrados-y-soluciones)
10. [Conclusión](#conclusión)

## Requisitos

- Ubuntu Server 24.04 en una máquina virtual (VMware).
- Usuario con permisos `sudo`.
- Conexión a internet (para instalar `at`).

## Guía de los campos de cron

Cada línea de un crontab tiene este formato:

```
m  h  dom  mon  dow  comando
```

| Campo | Significado | Valores |
|---|---|---|
| `m` | Minuto en que se ejecuta | 0-59 |
| `h` | Hora exacta, formato 24 h (0 = medianoche) | 0-23 |
| `dom` | Día del mes. Un `15` significa "el día 15"; para "cada 15 días" se usa `*/15` | 1-31 |
| `mon` | Mes del año | 1-12 |
| `dow` | Día de la semana: número o las 3 primeras letras en inglés (`mon`, `tue`, `wed`, `thu`, `fri`, `sat`, `sun`) | 0-6 (0 = domingo) |

Comandos útiles:

| Comando | Qué hace |
|---|---|
| `crontab -e` | Edita el crontab |
| `crontab -l` | Lista las tareas |
| `crontab -ri` | Borra todas las tareas pidiendo confirmación |

## Parte 1: Verificar cron y zona horaria

Las "11 PM" dependen de la zona horaria del sistema, así que primero se ajusta:

```bash
sudo timedatectl set-timezone America/Santo_Domingo
sudo systemctl restart cron
timedatectl
sudo systemctl status cron
```

Debe verse `America/Santo_Domingo (AST, -0400)` y cron en `active (running)`.

![Cron activo y zona horaria](imagenes/01-cron-activo-zona-horaria.png)

Revisar el crontab del sistema para conocer el formato:

```bash
cat /etc/crontab
```

![/etc/crontab](imagenes/02-etc-crontab.png)

## Parte 2: Actualización diaria a las 11 PM

Se edita el crontab de **root** (con `sudo`), porque `apt` necesita permisos y cron no puede escribir una contraseña. Por eso las líneas **no llevan `sudo`**.

```bash
sudo crontab -e
```

Agregar al final:

```
0 23 * * * apt update && apt upgrade -y >> /var/log/actualizacion_cron.log 2>&1
```

Guardar con `Ctrl+O`, `Enter` y salir con `Ctrl+X`.

## Parte 3: Reinicio los domingos a las 3 AM

En el mismo crontab de root agregar:

```
0 3 * * 0 /sbin/shutdown -r now
```

Verificar:

```bash
sudo crontab -l
```

![Dos tareas en crontab](imagenes/03-crontab-dos-tareas.png)

## Parte 4: Probar que cron funciona

Esperar hasta las 11 PM o el domingo no es práctico. Se agrega temporalmente una tarea que corre cada minuto:

```
* * * * * echo "Cron funcionando: $(date)" >> /tmp/prueba_cron.txt
```

Después de un par de minutos:

```bash
cat /tmp/prueba_cron.txt
```

![Prueba de cron](imagenes/04-prueba-cron.png)

Al terminar se **elimina solo esa línea** con `sudo crontab -e` (no con `crontab -r`, que borraría también las dos tareas reales) y se borra el archivo de prueba:

```bash
rm /tmp/prueba_cron.txt
```

## Parte 5: Tarea con AT

`at` ejecuta una tarea **una sola vez**; cron es para tareas repetitivas.

**1. Ver el contenido actual de `/tmp`** (con archivos de prueba):

```bash
touch /tmp/archivo1.txt /tmp/archivo2.txt /tmp/archivo3.txt
ls /tmp
```

![/tmp antes](imagenes/05-tmp-antes.png)

**2. Instalar y activar `at`:**

```bash
sudo apt install at -y
sudo systemctl enable --now atd
systemctl status atd
```

![atd activo](imagenes/06-atd-activo.png)

**3. Entrar como root y programar el borrado en 1 minuto:**

```bash
sudo su
at now + 1 minute
```

En el prompt `at>` escribir **solo** el comando, presionar Enter y confirmar con **Ctrl+D**:

```
rm -rf /tmp/*
```

Debe aparecer `<EOT>` y la línea `job N at ...`. Si no aparece, la tarea no se programó.

![at programado](imagenes/07-at-programado.png)

**4. Verificar después de 1 minuto:**

```bash
ls /tmp
atq
```

`/tmp` queda vacío y `atq` sin tareas pendientes. Luego salir de root con `exit`.

![/tmp después](imagenes/08-tmp-despues.png)

> ⚠️ `rm -rf /tmp/*` es lo que pide la práctica y en una VM de pruebas no hay problema. En un servidor real borraría archivos temporales de programas en uso.

## Parte 6: Eliminar tareas de cron

```bash
sudo crontab -l
sudo crontab -ri
sudo crontab -l
```

Responder `y` a la confirmación. El resultado final es `no crontab for root`.

![Crontab eliminado](imagenes/09-crontab-eliminado.png)

## Problemas encontrados y soluciones

| Problema | Causa | Solución |
|---|---|---|
| `crontab -i` no funciona solo | `-i` solo sirve junto con `-r` | Usar `crontab -l` para listar y `crontab -ri` para borrar con confirmación |
| `sudo` dentro del crontab | Cron no puede escribir la contraseña | Editar el crontab de root con `sudo crontab -e` y quitar `sudo` de las líneas |
| Hora en UTC | Zona horaria por defecto | `timedatectl set-timezone America/Santo_Domingo` |
| `Network is unreachable` al instalar `at` | La VM tenía IP pero no puerta de enlace por defecto | Corregir la red de la VM y repetir `apt update` / `apt install at` |
| `Unit file atd.service does not exist` | `at` no se había instalado por falta de internet | Instalar `at` con la red funcionando |
| El borrado con `at` no se ejecutó | Se salió del prompt `at>` con Ctrl+C (cancela) | Escribir el comando y confirmar con **Ctrl+D** |

## Conclusión

Se programaron tareas repetitivas con cron (actualización diaria a las 11 PM y reinicio dominical a las 3 AM) y una tarea única con `at` que borró el contenido de `/tmp`. También se comprobó la importancia de usar el crontab de root para tareas que requieren permisos, de ajustar la zona horaria y de verificar cada tarea antes de darla por buena.

[⬅ Volver al README](README.md)
