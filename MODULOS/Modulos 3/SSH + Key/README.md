<h1 align="center">Práctica 3 – Configuración SSH entre dos máquinas Linux</h1>

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
  <img src="https://img.shields.io/badge/SSH-OpenSSH-black?style=flat-square" alt="SSH">
</p>

---

## 📌 Descripción

Evidencias de la práctica realizada en dos máquinas virtuales Ubuntu Server (VMware) en el ITLA: configuración del servidor y cliente **SSH**, conexión remota con contraseña entre **Linux 1** y **Linux 2**, y autenticación por **clave pública** para entrar sin contraseña. Los comandos usados están en [GUIA.md](GUIA.md).

**Linux 1** (servidor): usuario `miguel`, equipo `miguel1`, IP `10.13.67.34`
**Linux 2** (cliente): usuario `miguel`, equipo `miguel2`, IP `10.13.67.35`

---

## 📸 Evidencias

### 1. IP de Linux 1
`ip a` en Linux 1 muestra la interfaz `ens33` con la IP `10.13.67.34`, necesaria para que Linux 2 se conecte a ella.

![ip a Linux 1](imagenes/01-ip-a-linux1.png)

### 2. IP de Linux 2
`ip a` en Linux 2 muestra la interfaz `ens33` con la IP `10.13.67.35`, la máquina cliente desde donde se inicia la conexión.

![ip a Linux 2](imagenes/02-ip-a-linux2.png)

### 3. Servicio SSH activo en Linux 1
`sudo systemctl status ssh` confirma que el servicio está `active (running)` y `enabled`, listo para recibir conexiones.

![status ssh Linux 1](imagenes/03-ssh-status-linux1.png)

### 4. Servicio SSH activo en Linux 2
El mismo servicio también está activo en Linux 2, ya que ambas máquinas necesitan tener SSH instalado.

![status ssh Linux 2](imagenes/04-ssh-status-linux2.png)

### 5. Generación del par de claves en Linux 2
`ssh-keygen` crea el par de claves ED25519 (privada y pública), mostrando el fingerprint y el randomart.

![ssh-keygen Linux 2](imagenes/05-ssh-keygen-linux2.png)

### 6. Clave pública generada
`cat ~/.ssh/id_ed25519.pub` muestra la clave pública completa de Linux 2, lista para copiarse a Linux 1.

![Clave pública](imagenes/06-clave-publica-linux2.png)

### 7. Primera conexión SSH con contraseña
Desde Linux 2 se conecta por primera vez a Linux 1 con `ssh miguel@10.13.67.34`. Se acepta el fingerprint del host y se introduce la contraseña.

![Conexión con contraseña](imagenes/07-conexion-con-password.png)

### 8. Verificación del usuario en Linux 1
Ya dentro de Linux 1, `whoami` confirma que la sesión pertenece al usuario `miguel`.

![whoami Linux 1](imagenes/08-whoami-linux1.png)

### 9. Clave pública agregada en Linux 1
Se crea `~/.ssh/authorized_keys` en Linux 1 y se pega la clave pública de Linux 2, confirmada con `cat`.

![authorized_keys pegada](imagenes/09-authorized-keys-pegada.png)

### 10. Permisos correctos de la carpeta y el archivo SSH
`chmod 600` sobre `authorized_keys` y `ls -la ~/.ssh` confirman los permisos obligatorios: `700` en la carpeta y `600` en el archivo.

![Permisos SSH](imagenes/10-permisos-ssh.png)

### 11. Conexión sin contraseña
Desde Linux 2, `ssh miguel@10.13.67.34` entra directo, sin pedir contraseña, lo que demuestra que la autenticación por clave pública funciona.

![Conexión sin contraseña](imagenes/11-conexion-sin-password.png)

---

## 👤 Autor

**Miguel Ramírez (Meli)** · Matrícula 20251367
Institución: ITLA
Materia: Sistemas Operativos 3
Profesor: Adrián Alcántara
