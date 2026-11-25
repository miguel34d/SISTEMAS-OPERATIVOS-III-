<h1 align="center">📖 Guía – Práctica 3: Disco adicional, partición y EXT4</h1>

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

1. [Crear y adjuntar el disco en VMware](#1-crear-y-adjuntar-el-disco-en-vmware)
2. [Reiniciar y detectar el disco](#2-reiniciar-y-detectar-el-disco)
3. [Crear la partición](#3-crear-la-partición)
4. [Formatear en EXT4](#4-formatear-en-ext4)
5. [Crear la carpeta y montar el disco](#5-crear-la-carpeta-y-montar-el-disco)
6. [Crear el archivo en el disco](#6-crear-el-archivo-en-el-disco)
7. [Desmontar](#7-desmontar)
8. [Montar en /mnt y comprobar](#8-montar-en-mnt-y-comprobar)
9. [Resumen de comandos](#9-resumen-de-comandos)
10. [Errores comunes](#10-errores-comunes)

---

## 1. Crear y adjuntar el disco en VMware

Esta parte es una acción en VMware, no un comando. Con la VM **apagada** (`sudo poweroff`):

1. **VM → Settings → Add... → Hard Disk → Next**.
2. Tipo **SCSI** → **Create a new virtual disk → Next**.
3. Tamaño **20 GB**. Deja *Store virtual disk as a single file* y no marques *Allocate all disk space now*.
4. **Next → Finish → OK**. En la lista de hardware deben verse dos discos: 40 GB y 20 GB.

## 2. Reiniciar y detectar el disco

Enciende la VM, inicia sesión y reinicia:

```bash
sudo reboot
```

Luego detecta el disco nuevo con cualquiera de estos comandos:

```bash
lsblk
sudo fdisk -l
```

Debe aparecer `sdb` de 20G sin particiones. `sda` es el disco del sistema y **no se toca**. Si el disco nuevo tuviera otro nombre, usa ese en todos los comandos siguientes.

## 3. Crear la partición

```bash
sudo fdisk /dev/sdb
```

Dentro de `fdisk`:

| Tecla | Qué hace |
|---|---|
| `n` | Nueva partición |
| `p` | Primaria |
| `1` | Número de partición |
| `Enter` | Primer sector (por defecto) |
| `Enter` | Último sector (por defecto, usa todo el disco) |
| `w` | Guardar y salir |

Antes de escribir `w`, comprueba que el mensaje inicial diga `/dev/sdb`. Verifica:

```bash
lsblk
```

Debe verse `sdb1` de 20G debajo de `sdb`.

## 4. Formatear en EXT4

```bash
sudo mkfs.ext4 /dev/sdb1
lsblk -f
```

`lsblk -f` debe mostrar `ext4` en la columna `FSTYPE` de `sdb1`.

## 5. Crear la carpeta y montar el disco

En Ubuntu Server la carpeta `Desktop` puede no existir, por eso se usa `-p`, que la crea junto con `SO3`:

```bash
mkdir -p ~/Desktop/SO3
sudo mount /dev/sdb1 ~/Desktop/SO3
df -h | grep sdb1
```

`df -h` debe mostrar `/dev/sdb1` montado en `~/Desktop/SO3`.

## 6. Crear el archivo en el disco

```bash
cd ~/Desktop/SO3
sudo touch AdrianAlcantara.txt
ls -l
```

Aparecen `AdrianAlcantara.txt` y `lost+found` (lo crea EXT4 solo; es normal).

## 7. Desmontar

Hay que salir de la carpeta **antes** de desmontar; si no, aparece `target is busy`:

```bash
cd ~
sudo umount ~/Desktop/SO3
ls ~/Desktop/SO3
```

La carpeta queda vacía porque el disco ya no está montado en ella.

## 8. Montar en /mnt y comprobar

Se monta la **misma partición** (`sdb1`) en otro punto:

```bash
sudo mount /dev/sdb1 /mnt
ls /mnt
```

Debe salir `AdrianAlcantara.txt`. Eso demuestra que el archivo está guardado en el disco. Al terminar:

```bash
sudo umount /mnt
```

El montaje es temporal: si se reinicia la VM, hay que montar de nuevo.

## 9. Resumen de comandos

```bash
sudo reboot
lsblk
sudo fdisk -l
sudo fdisk /dev/sdb          # n, p, 1, Enter, Enter, w
lsblk
sudo mkfs.ext4 /dev/sdb1
lsblk -f
mkdir -p ~/Desktop/SO3
sudo mount /dev/sdb1 ~/Desktop/SO3
cd ~/Desktop/SO3
sudo touch AdrianAlcantara.txt
cd ~
sudo umount ~/Desktop/SO3
sudo mount /dev/sdb1 /mnt
ls /mnt
sudo umount /mnt
```

## 10. Errores comunes

| Error | Causa | Solución |
|---|---|---|
| `fdisk: cannot open /dev/sdb: No such file or directory` | La VM no tiene el disco nuevo | Agregarlo en VMware con la VM apagada y reiniciar |
| `This disk is currently in use` | Se abrió `/dev/sda`, el disco del sistema | Salir con `q` (sin guardar) y usar `/dev/sdb` |
| `umount: target is busy` | Se está dentro de la carpeta montada | `cd ~` y volver a desmontar |
| `ls /mnt` no muestra el archivo | Se montó `/dev/sda1` en lugar de `/dev/sdb1` | Montar `/dev/sdb1`, que es donde está el archivo |
| `mkdir: cannot create directory ... Desktop` | `Desktop` no existe en Ubuntu Server | Usar `mkdir -p ~/Desktop/SO3` |

[⬅ Volver al README](README.md)
