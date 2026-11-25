<h1 align="center">Práctica 4 – Configuración de GRUB y recuperación de contraseña root</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Estudiante-Miguel%20Ram%C3%ADrez%20(Meli)-blue?style=for-the-badge" alt="Estudiante">
  <img src="https://img.shields.io/badge/Matr%C3%ADcula-20251367-green?style=for-the-badge" alt="Matrícula">
  <img src="https://img.shields.io/badge/Materia-Sistemas%20Operativos%203-orange?style=for-the-badge" alt="Materia">
  <img src="https://img.shields.io/badge/Maestro-Adri%C3%A1n%20Alc%C3%A1ntara-purple?style=for-the-badge" alt="Maestro">
  <img src="https://img.shields.io/badge/Instituci%C3%B3n-ITLA-red?style=for-the-badge" alt="ITLA">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420?style=flat-square&logo=ubuntu&logoColor=white" alt="Ubuntu">
  <img src="https://img.shields.io/badge/VMware-Workstation-607078?style=flat-square&logo=vmware&logoColor=white" alt="VMware">
  <img src="https://img.shields.io/badge/GRUB-Bootloader-black?style=flat-square" alt="GRUB">
  <img src="https://img.shields.io/badge/Bash-Terminal-4EAA25?style=flat-square&logo=gnubash&logoColor=white" alt="Bash">
</p>

---

## 📌 Descripción

Evidencias de la práctica realizada en Ubuntu Server (VMware) en el ITLA: se configuró el tiempo de espera del menú de **GRUB**, se usó el modo de edición de arranque para entrar en modo de recuperación (`rw init=/bin/bash`) y se restableció la contraseña del usuario **root**. Los comandos usados están en [GUIA.md](GUIA.md).

---

## 📸 Evidencias

### 1. Configuración de GRUB
En `/etc/default/grub` se cambió `GRUB_TIMEOUT_STYLE` a `menu` (para que el menú se muestre) y `GRUB_TIMEOUT` a `20` segundos, dando tiempo de interrumpir el arranque.

![Configuración de GRUB](imagenes/01-grub-config.png)

### 2. Aplicar los cambios con `update-grub`
`sudo update-grub` regenera la configuración de arranque y confirma que encontró la imagen de Linux, terminando con `done`.

![update-grub](imagenes/02-update-grub.png)

### 3. Edición de la línea de arranque en GRUB
Desde el menú de GRUB, con `e` se edita la entrada y se agrega `rw init=/bin/bash` al final de la línea `linux`, para arrancar directo a una shell de root sin pedir contraseña.

![Línea linux editada](imagenes/03-linea-rw-init.png)

### 4. Cambio de la contraseña de root
Ya dentro de la shell de recuperación, `passwd root` permite definir una nueva contraseña, confirmada con `password updated successfully`.

![passwd root exitoso](imagenes/04-passwd-root-exitoso.png)

### 5. Cuenta root desbloqueada
Tras reiniciar, se usó `sudo passwd -u root` para desbloquear la cuenta y `sudo passwd -S root` confirma el estado `P` (contraseña activa).

![passwd -S root desbloqueado](imagenes/05-passwd-desbloqueado.png)

### 6. Acceso como root con la nueva contraseña
Con `su` se inicia sesión como root usando la contraseña restablecida; `whoami` confirma que el usuario activo ahora es `root`.

![su y whoami root](imagenes/06-su-whoami-root.png)

---

## 👤 Autor

**Miguel Ramírez (Meli)** · Matrícula 20251367
Institución: ITLA
Materia: Sistemas Operativos 3
Profesor: Adrián Alcántara
