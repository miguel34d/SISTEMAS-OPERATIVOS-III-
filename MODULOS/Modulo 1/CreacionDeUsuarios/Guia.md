# 👥 Guía – Práctica 3
## Gestión de Usuarios y Grupos

![Alumno](https://img.shields.io/badge/Alumno-Miguel%20Ram%C3%ADrez%20Meli-2563eb?style=for-the-badge&logo=github&logoColor=white)
![Matrícula](https://img.shields.io/badge/Matr%C3%ADcula-2025--1367-16a34a?style=for-the-badge)
![ITLA](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-ea580c?style=for-the-badge)
![Materia](https://img.shields.io/badge/Materia-Sistemas%20Operativos%20III-7c3aed?style=for-the-badge&logo=linux&logoColor=white)
![Profesor](https://img.shields.io/badge/Profesor-Adri%C3%A1n%20Alc%C3%A1ntara-db2777?style=for-the-badge)
![Práctica](https://img.shields.io/badge/Pr%C3%A1ctica-3%20Usuarios%20y%20Grupos-0891b2?style=for-the-badge)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%20(VMware)-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Windows](https://img.shields.io/badge/CMD-Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)

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
| 🖥️ **Entorno** | Ubuntu en VMware (usuario `miguel`) y CMD de Windows |

## 🎯 Objetivo

Crear, modificar y eliminar usuarios y grupos en Linux, asignar permisos de administrador con `sudo` y verificar cada cambio con `groups` y `getent`.

## 📑 Índice

- [1. Crear usuario](#1-crear-usuario)
- [2. Agregar usuario a sudo](#2-agregar-usuario-a-sudo)
- [3. Crear grupo](#3-crear-grupo)
- [4. Crear otro usuario y agregarlo al grupo](#4-crear-otro-usuario-y-agregarlo-al-grupo)
- [5. Eliminar usuario de un grupo](#5-eliminar-usuario-de-un-grupo)
- [6. Eliminar usuario y grupo](#6-eliminar-usuario-y-grupo)
- [7. Windows: hora y fecha](#7-windows-hora-y-fecha)
- [Notas](#-notas)
- [Limpieza](#-limpieza-opcional)

---

### 1. Crear usuario

💡 `adduser` crea el usuario, su grupo personal y su carpeta en `/home`. `miguel` ya existía (es el usuario de la VM), por eso el primer `adduser` avisa que ya existe; para demostrar la creación real se usó `miguel2`.

```bash
# Intenta crear a miguel (ya existe)
sudo adduser miguel
# Muestra los grupos de miguel
groups miguel
# Crea el usuario miguel2 (pide contraseña y datos)
sudo adduser miguel2
# Muestra sus grupos: solo su grupo personal
groups miguel2
```

### 2. Agregar usuario a sudo

💡 `usermod -aG` **añade** el usuario a un grupo (`-a` es clave: sin él se reemplazan los grupos que ya tenía). `sudo -l -U` confirma que puede ejecutar comandos como administrador.

```bash
# Agrega miguel2 al grupo sudo
sudo usermod -aG sudo miguel2
# Verifica que aparece sudo
groups miguel2
# Lista los permisos de sudo de miguel2
sudo -l -U miguel2
```

### 3. Crear grupo

💡 `addgroup` crea un grupo nuevo. `getent group` lo consulta en la base de datos del sistema; el campo final vacío indica que no tiene miembros.

```bash
# Crea el grupo guest (GID 1003)
sudo addgroup guest
# Muestra el grupo: guest:x:1003: (sin miembros)
getent group guest
```

### 4. Crear otro usuario y agregarlo al grupo

💡 Se crea `julian` y se agrega a `guest`. En `getent group guest` ahora aparece `julian` como miembro.

```bash
# Crea el usuario julian
sudo adduser julian
# Agrega julian al grupo guest
sudo usermod -aG guest julian
# Muestra: julian : julian guest
groups julian
# Muestra: guest:x:1003:julian
getent group guest
```

### 5. Eliminar usuario de un grupo

💡 `gpasswd -d` quita al usuario del grupo sin borrar ni el usuario ni el grupo.

```bash
# Quita a julian del grupo guest
sudo gpasswd -d julian guest
# Verifica: julian : julian
groups julian
# Verifica: guest:x:1003: (sin miembros)
getent group guest
```

### 6. Eliminar usuario y grupo

💡 `deluser` borra el usuario (y su grupo personal si queda vacío, de ahí el *Warning* que es normal) y `delgroup` borra el grupo. La carpeta `/home/julian` **no** se borra a menos que se use `--remove-home`. `grep` sin salida confirma que ya no existen.

```bash
# Elimina el usuario julian
sudo deluser julian
# Elimina el grupo guest
sudo delgroup guest
# Sin salida = ya no existen
getent group | grep -E "julian|guest"
```

### 7. Windows: hora y fecha

💡 Se ejecutan en **CMD** de Windows (en PowerShell `time /t` da error). Muestran la hora y la fecha actuales.

```bat
REM Muestra la hora actual
time /t
REM Muestra la fecha actual
date /t
```

---

## 📝 Notas

| ⚠️ Situación | 🔍 Explicación |
|---|---|
| `adduser miguel` dice que ya existe | `miguel` es el usuario con el que se trabaja en la VM |
| `usermod -G` sin `-a` | Reemplaza todos los grupos secundarios; siempre usar `-aG` |
| `Warning: group 'julian' has no more members` | Normal: `deluser` borra el grupo personal al quedar vacío |
| La carpeta `/home/julian` sigue existiendo | Para borrarla: `sudo deluser --remove-home julian` |
| `time /t` falla en PowerShell | Ejecutarlo en CMD |
| Usuario extra `ronald` | Se creó durante las pruebas; no forma parte de la práctica |

## 🧹 Limpieza (opcional)

```bash
sudo deluser --remove-home miguel2
sudo deluser --remove-home ronald
sudo rm -rf /home/julian      # revisar bien la ruta antes de ejecutar
getent passwd | grep -E "miguel2|ronald|julian"   # no debe imprimir nada
```

---

<div align="center">

**Miguel Ramírez Meli · 2025-1367 · ITLA**
Sistemas Operativos III · Prof. Adrián Alcántara

</div>
