<h1 align="center">📖 Guía – Práctica 3: Configuración SSH entre dos máquinas Linux</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Estudiante-Miguel%20Ram%C3%ADrez%20(Meli)-blue?style=for-the-badge" alt="Estudiante">
  <img src="https://img.shields.io/badge/Matr%C3%ADcula-20251367-green?style=for-the-badge" alt="Matrícula">
  <img src="https://img.shields.io/badge/Materia-Sistemas%20Operativos%203-orange?style=for-the-badge" alt="Materia">
  <img src="https://img.shields.io/badge/Maestro-Adri%C3%A1n%20Alc%C3%A1ntara-purple?style=for-the-badge" alt="Maestro">
  <img src="https://img.shields.io/badge/Instituci%C3%B3n-ITLA-red?style=for-the-badge" alt="ITLA">
</p>

Guía de los comandos usados en la práctica. Las capturas están en el [README.md](README.md).

Se usaron dos máquinas: **Linux 1** (servidor, `miguel@10.13.67.34`) y **Linux 2** (cliente, `miguel@10.13.67.35`).

---

## Índice

1. [Ver la IP de cada máquina](#1-ver-la-ip-de-cada-máquina)
2. [Instalar y activar SSH](#2-instalar-y-activar-ssh)
3. [Generar el par de claves en Linux 2](#3-generar-el-par-de-claves-en-linux-2)
4. [Primera conexión con contraseña](#4-primera-conexión-con-contraseña)
5. [Agregar la clave pública en Linux 1](#5-agregar-la-clave-pública-en-linux-1)
6. [Probar el acceso sin contraseña](#6-probar-el-acceso-sin-contraseña)
7. [Resumen de comandos](#7-resumen-de-comandos)
8. [Errores comunes](#8-errores-comunes)

---

## 1. Ver la IP de cada máquina

En **ambas** VMs:

```bash
ip a
```

Anotar la IP de la interfaz activa (`ens33` en este caso) de cada una. Son necesarias para que cada máquina sepa a cuál conectarse.

## 2. Instalar y activar SSH

En **ambas** VMs:

```bash
sudo apt install openssh-client openssh-server -y
sudo systemctl enable --now ssh
sudo systemctl status ssh
whoami
```

El `status` debe mostrar `active (running)` y `enabled`. El `whoami` confirma el usuario de cada máquina (en este caso, `miguel` en las dos).

## 3. Generar el par de claves en Linux 2

Solo en la máquina que actuará como **cliente** (Linux 2):

```bash
ssh-keygen
```

Presionar `Enter` en las 3 preguntas (ubicación por defecto, sin passphrase).

Ver la clave pública generada:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copiar esa línea completa; se pegará en Linux 1 más adelante.

## 4. Primera conexión con contraseña

Desde **Linux 2**, conectarse a **Linux 1** usando su IP:

```bash
ssh miguel@10.13.67.34
```

La primera vez pregunta si se confía en el fingerprint del host; responder `yes`. Luego pide la contraseña del usuario de Linux 1.

Verificar que se entró correctamente:

```bash
whoami
```

## 5. Agregar la clave pública en Linux 1

**Sin salir de la sesión SSH** (ya dentro de Linux 1):

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

Pegar ahí la línea completa de la clave pública de Linux 2. Guardar con `Ctrl+O`, `Enter`, y salir con `Ctrl+X`.

Ajustar los permisos del archivo y confirmar:

```bash
chmod 600 ~/.ssh/authorized_keys
ls -la ~/.ssh
cat ~/.ssh/authorized_keys
```

Los permisos correctos son obligatorios: carpeta `.ssh` en `700` y `authorized_keys` en `600`. Si no, SSH rechaza la clave sin explicar por qué.

**Nunca usar `sudo nano` aquí.** Si el archivo queda con dueño `root`, la autenticación por clave falla.

Salir de la sesión para volver a Linux 2:

```bash
exit
```

## 6. Probar el acceso sin contraseña

Desde **Linux 2**:

```bash
ssh miguel@10.13.67.34
```

Esta vez debe entrar directo, **sin pedir contraseña**. Confirmar y salir:

```bash
whoami
exit
```

## 7. Resumen de comandos

```bash
# En ambas VMs
ip a
sudo apt install openssh-client openssh-server -y
sudo systemctl enable --now ssh
sudo systemctl status ssh

# Solo en Linux 2 (cliente)
ssh-keygen
cat ~/.ssh/id_ed25519.pub
ssh miguel@10.13.67.34        # primera conexión, con contraseña

# Dentro de Linux 1, en esa misma sesión
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys    # pegar la clave pública de Linux 2
chmod 600 ~/.ssh/authorized_keys
exit

# De vuelta en Linux 2
ssh miguel@10.13.67.34        # ya no pide contraseña
```

## 8. Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `cat: ... id_ed25519.pub: No such file or directory` | Aún no se corrió `ssh-keygen` | Generar la clave primero con `ssh-keygen` |
| Sigue pidiendo contraseña después de agregar la clave | `authorized_keys` o `.ssh` con permisos o dueño incorrectos | `chmod 700 ~/.ssh`, `chmod 600 ~/.ssh/authorized_keys`, y que el dueño sea el usuario, no root |
| Se usó `sudo nano authorized_keys` | El archivo y la carpeta quedan con dueño `root` | Editar siempre sin `sudo`, como el usuario normal |
| `ssh: connect to host ... port 22: Connection refused` | El servicio SSH no está activo en el destino | `sudo systemctl enable --now ssh` en esa máquina |
| La IP cambia después de reiniciar | La IP es dinámica (DHCP) | Verificar siempre con `ip a` antes de conectar, en vez de usar una IP fija de memoria |

[⬅ Volver al README](README.md)
