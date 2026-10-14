# 🔐 Guía – Práctica 4
## Gestión de Permisos de Archivos

![Alumno](https://img.shields.io/badge/Alumno-Miguel%20Ram%C3%ADrez%20Meli-2563eb?style=for-the-badge&logo=github&logoColor=white)
![Matrícula](https://img.shields.io/badge/Matr%C3%ADcula-2025--1367-16a34a?style=for-the-badge)
![ITLA](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-ea580c?style=for-the-badge)
![Materia](https://img.shields.io/badge/Materia-Sistemas%20Operativos%20III-7c3aed?style=for-the-badge&logo=linux&logoColor=white)
![Profesor](https://img.shields.io/badge/Profesor-Adri%C3%A1n%20Alc%C3%A1ntara-db2777?style=for-the-badge)
![Práctica](https://img.shields.io/badge/Pr%C3%A1ctica-4%20Permisos%20de%20Archivos-0891b2?style=for-the-badge)
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

## 🎯 Objetivo

Crear archivos y carpetas, modificar sus permisos (simbólicos y numéricos), copiarlos, cambiar dueño y grupo, y comprobar el efecto de los permisos con otro usuario.

## 🔢 Referencia rápida de permisos

| Permiso | Letra | Valor | En archivos | En carpetas |
|:-:|:-:|:-:|---|---|
| Lectura | `r` | 4 | Leer el contenido | Listar el contenido |
| Escritura | `w` | 2 | Modificar | Crear y borrar dentro |
| Ejecución | `x` | 1 | Ejecutar | **Entrar** (`cd`) |

`-rwxr-xr--` → dueño `rwx` (7), grupo `r-x` (5), otros `r--` (4) = **754**

## 📑 Índice

- [1. Crear carpeta y archivo](#1-crear-carpeta-y-archivo)
- [2. Cambiar permisos con chmod 0](#2-cambiar-permisos-con-chmod-0)
- [3. Permisos para usuario, grupo y otros](#3-permisos-para-usuario-grupo-y-otros)
- [4. Quitar permisos y permisos numéricos](#4-quitar-permisos-y-permisos-numéricos)
- [5. Segunda carpeta y exploración de directorios](#5-segunda-carpeta-y-exploración-de-directorios)
- [6. Permisos en carpetas](#6-permisos-en-carpetas)
- [7. Copiar archivo a otra carpeta](#7-copiar-archivo-a-otra-carpeta)
- [8. Cambiar dueño y grupo](#8-cambiar-dueño-y-grupo)
- [9. Probar permisos con otro usuario](#9-probar-permisos-con-otro-usuario)
- [10. Eliminar carpetas y usuario de prueba](#10-eliminar-carpetas-y-usuario-de-prueba)
- [Notas](#-notas)

---

### 1. Crear carpeta y archivo

💡 `mkdir` crea la carpeta, `vi` abre el editor para escribir el archivo (`i` para escribir, `Esc` y `:wq` para guardar y salir) y `cat` muestra su contenido.

```bash
# Crea la carpeta Materia
mkdir Materia
# Entra a la carpeta
cd Materia
# Crea y edita el archivo con vi
vi estudiante.txt
# Lista los archivos
ls
# Muestra el contenido del archivo
cat estudiante.txt
```

### 2. Cambiar permisos con chmod 0

💡 `chmod 0` quita todos los permisos. Aunque `miguel` es el dueño, ya no puede leer su propio archivo: `cat` da *Permission denied*.

```bash
# Permisos antes: -rw-rw-r--
ls -l estudiante.txt
# Quita todos los permisos
chmod 0 estudiante.txt
# Permisos después: ----------
ls -l estudiante.txt
# Permission denied
cat estudiante.txt
```

### 3. Permisos para usuario, grupo y otros

💡 Con la forma simbólica se agregan permisos por clase: `u` (dueño), `g` (grupo) y `o` (otros). Cada `chmod` amplía el acceso hasta dejar el archivo en `777`.

```bash
# PARA USUARIOS: -rwx------
chmod u+rwx estudiante.txt
# Verifica
ls -l estudiante.txt
# PARA GRUPOS: -rwxrwx---
chmod g+rwx estudiante.txt
# Verifica
ls -l estudiante.txt
# PARA OTROS: -rwxrwxrwx
chmod o+rwx estudiante.txt
# Verifica
ls -l estudiante.txt
```

### 4. Quitar permisos y permisos numéricos

💡 `-` quita permisos. En modo numérico `r=4`, `w=2`, `x=1` y se suman por dueño, grupo y otros: `644` = `rw-r--r--`, `755` = `rwxr-xr-x`, `600` = `rw-------`. `stat` muestra el número y el texto juntos.

```bash
# Quita todo a otros
chmod o-rwx estudiante.txt
# Quita escritura al grupo
chmod g-w estudiante.txt
# Resultado: -rwxr-x---
ls -l estudiante.txt
# rw-r--r--
chmod 644 estudiante.txt
# rwxr-xr-x
chmod 755 estudiante.txt
# rw-------
chmod 600 estudiante.txt
# Muestra permisos en número y texto
stat -c "%a %A %n" estudiante.txt
```

### 5. Segunda carpeta y exploración de directorios

💡 `mkdir` crea la segunda carpeta, `cd -` vuelve a la carpeta anterior y `pwd` muestra la ruta actual.

```bash
# Va a la carpeta personal
cd ~
# Crea la segunda carpeta
mkdir Materia2
# Lista las carpetas
ls
# Entra a Materia
cd Materia
# Lista su contenido
ls
# Regresa a la carpeta anterior
cd -
# Entra a Materia2
cd Materia2
# Muestra la ruta actual
pwd
# Lista su contenido (vacía)
ls
# Regresa a la carpeta personal
cd ~
```

### 6. Permisos en carpetas

💡 En una carpeta, el permiso `x` es el que permite **entrar**. Con `000` el `cd` falla; con `755` vuelve a funcionar. La `d` inicial de `ls -ld` indica que es un directorio.

```bash
# Permisos de las carpetas (drwxrwxr-x)
ls -ld Materia Materia2
# Quita todos los permisos
chmod 000 Materia2
# Permission denied
cd Materia2
# Restaura permisos
chmod 755 Materia2
# Ahora sí entra
cd Materia2
# Confirma la ruta
pwd
# Regresa
cd ~
```

### 7. Copiar archivo a otra carpeta

💡 `cp` crea una copia y no mueve el original. La copia conserva los permisos (`600`) y el dueño sigue siendo `miguel`.

```bash
# Entra a Materia
cd ~/Materia
# Copia el archivo a Materia2
cp estudiante.txt ~/Materia2
# El original sigue en Materia
ls
# Verifica la copia
ls -l ~/Materia2
# Regresa
cd ~
```

### 8. Cambiar dueño y grupo

💡 `chown` cambia el dueño y `chgrp` el grupo del archivo. Se usó `root` y `sudo` porque son cuentas que existen siempre; al final se restaura a `miguel:miguel`.

```bash
# Cambia el dueño a root
sudo chown root ~/Materia2/estudiante.txt
# root miguel
ls -l ~/Materia2
# Cambia el grupo a sudo
sudo chgrp sudo ~/Materia2/estudiante.txt
# root sudo
ls -l ~/Materia2
# Restaura dueño y grupo
sudo chown miguel:miguel ~/Materia2/estudiante.txt
# miguel miguel
ls -l ~/Materia2
```

### 9. Probar permisos con otro usuario

💡 Se crea un usuario sin privilegios (`prueba`) y se lee el archivo con `sudo -u`. Con `600` es denegado; con `644` puede leerlo. Se abre temporalmente el paso a la carpeta personal con `chmod o+x ~` y se cierra al final.

```bash
# Crea el usuario de prueba
sudo adduser --disabled-password --gecos "" prueba
# Permite el paso a la carpeta personal
chmod o+x ~
# Solo el dueño puede leer
chmod 600 ~/Materia/estudiante.txt
# Permission denied
sudo -u prueba cat ~/Materia/estudiante.txt
# Otros pueden leer
chmod 644 ~/Materia/estudiante.txt
# Muestra el contenido
sudo -u prueba cat ~/Materia/estudiante.txt
# Cierra el paso a la carpeta personal
chmod o-x ~
```

### 10. Eliminar carpetas y usuario de prueba

💡 `rm -r` borra carpetas con su contenido y no pregunta. `deluser --remove-home` elimina al usuario de prueba y su carpeta personal.

```bash
# Va a la carpeta personal
cd ~
# Elimina Materia
rm -r Materia
# Verifica
ls
# Elimina Materia2
rm -r Materia2
# Verifica
ls
# Elimina el usuario de prueba y su carpeta
sudo deluser --remove-home prueba
```

---

## 📝 Notas

| ⚠️ Situación | 🔍 Explicación |
|---|---|
| Carpetas `Materia` y `Materia2` | Linux distingue mayúsculas: se usaron los nombres exactos que se crearon |
| Ruta `/home/miguel-20251367/Escritorio/...` | No existe en la VM; se usó `~/Materia2` |
| `z2 cd ..` de la plantilla | Era un error de escritura y se eliminó |
| `cat` da *Permission denied* con `chmod 0` | Sin permisos ni el dueño puede leer el archivo |
| Archivo en `777` | Es lo más abierto posible; se corrigió con permisos numéricos |
| `cd` falla con `chmod 000` en carpeta | Sin `x` no se puede entrar a un directorio |
| `chown miguel2` dio *invalid user* | Ese usuario ya no existía; se usó `root` |
| `chmod o+x ~` en la prueba | Permite a `prueba` atravesar la carpeta personal; se revierte con `o-x` |
| `rm -r` no pide confirmación | Revisar bien el nombre antes de ejecutar |

---

<div align="center">

**Miguel Ramírez Meli · 2025-1367 · ITLA**
Sistemas Operativos III · Prof. Adrián Alcántara

</div>
