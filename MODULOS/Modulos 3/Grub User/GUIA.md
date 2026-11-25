<h1 align="center">📖 Guía – Práctica 4: GRUB y recuperación de contraseña root</h1>

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

1. [Verificar el estado de root](#1-verificar-el-estado-de-root)
2. [Configurar el timeout de GRUB](#2-configurar-el-timeout-de-grub)
3. [Ver el menú de GRUB](#3-ver-el-menú-de-grub)
4. [Entrar en modo de recuperación](#4-entrar-en-modo-de-recuperación)
5. [Cambiar la contraseña de root](#5-cambiar-la-contraseña-de-root)
6. [Reiniciar y comprobar el acceso](#6-reiniciar-y-comprobar-el-acceso)
7. [Resumen de comandos](#7-resumen-de-comandos)
8. [Errores comunes](#8-errores-comunes)

---

## 1. Verificar el estado de root

Antes de tocar nada, se revisa si la cuenta root está bloqueada (así viene por defecto en Ubuntu):

```bash
sudo passwd -S root
```

`L` significa bloqueada; `P`, contraseña activa.

## 2. Configurar el timeout de GRUB

```bash
sudo nano /etc/default/grub
```

Dejar (o agregar) estas dos líneas:

```
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=20
```

`GRUB_TIMEOUT_STYLE=menu` es necesario porque Ubuntu suele traer el menú oculto (`hidden`); sin cambiarlo, el `TIMEOUT` no serviría de nada. Guardar con `Ctrl+O`, `Enter`, y salir con `Ctrl+X`.

Aplicar los cambios:

```bash
sudo update-grub
```

## 3. Ver el menú de GRUB

Antes de reiniciar, abrir la ventana de la VM en VMware (ahí se ve el arranque):

```bash
sudo reboot
```

Hacer clic dentro de la ventana de la VM. Debe aparecer el menú de GRUB con la cuenta regresiva de 20 segundos. **No dejar que arranque solo.**

## 4. Entrar en modo de recuperación

1. Con la entrada **Ubuntu** seleccionada, presionar **`e`**.
2. Ir con las flechas hasta la línea que empieza con `linux`, moverse al final con `End` y agregar:

```
rw init=/bin/bash
```

3. Presionar **Ctrl+X** (o F10) para arrancar con ese cambio. Aparece un prompt como `root@(none):/#`.

> El teclado de GRUB usa distribución **US**; si algún símbolo sale distinto en un teclado latino, revisar en pantalla que la línea quedó escrita correctamente.

## 5. Cambiar la contraseña de root

Comprobar que el sistema de archivos está en lectura/escritura:

```bash
mount | grep " / "
```

Si dice `ro`, montar de nuevo en modo escritura:

```bash
mount -o remount,rw /
```

Cambiar la contraseña:

```bash
passwd root
```

Escribir la nueva contraseña dos veces. Puede advertir que es débil, pero la acepta igual.

Guardar los cambios en disco y reiniciar a la fuerza (en este modo no hay `systemd`, por eso `reboot -f`):

```bash
sync
reboot -f
```

## 6. Reiniciar y comprobar el acceso

Si la cuenta seguía bloqueada, desbloquearla:

```bash
sudo passwd -u root
sudo passwd -S root
```

Debe mostrar `root P ...`.

Probar el acceso con la nueva contraseña:

```bash
su
whoami
exit
```

`whoami` debe mostrar `root`.

## 7. Resumen de comandos

```bash
sudo passwd -S root
sudo nano /etc/default/grub      # GRUB_TIMEOUT_STYLE=menu, GRUB_TIMEOUT=20
sudo update-grub
sudo reboot
# En el menú de GRUB: e -> agregar "rw init=/bin/bash" al final de la línea linux -> Ctrl+X
passwd root
sync
reboot -f
sudo passwd -u root
sudo passwd -S root
su
whoami
```

## 8. Errores comunes

| Error | Causa | Solución |
|---|---|---|
| El menú de GRUB no aparece | `GRUB_TIMEOUT_STYLE` quedó en `hidden` | Cambiarlo a `menu` y correr `sudo update-grub` |
| `rw init=/bin/bash` se escribe mal | Se intentó teclear en la terminal normal, no en el editor de GRUB | Escribirlo solo tras presionar `e` sobre la entrada de arranque |
| `passwd root` falla con solo lectura | El sistema de archivos montó en modo `ro` | `mount -o remount,rw /` antes de cambiar la contraseña |
| `passwd -S root` dice "You may not view..." | Se ejecutó sin `sudo` | Usar `sudo passwd -S root` |
| `sudo passwd -l root` en vez de verificar | `-l` **bloquea** la cuenta (lock), no la muestra | Usar `sudo passwd -S root` para ver el estado, y `sudo passwd -u root` para desbloquear si quedó bloqueada |
| `su` no cambia a root | No se completó el `su` o se escribió mal la contraseña | Repetir `su`, escribir la contraseña (no se ve en pantalla) y confirmar con `whoami` |

[⬅ Volver al README](README.md)
