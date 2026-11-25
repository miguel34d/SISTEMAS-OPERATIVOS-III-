<div align="center">

# 📸 Demostración – Práctica 3
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

> **Alumno:** Miguel Ramírez Meli · **Matrícula:** 2025-1367 · **ITLA**
> **Materia:** Sistemas Operativos III · **Profesor:** Adrián Alcántara
> **Entorno:** Ubuntu en VMware y CMD de Windows

Solo se incluyen las capturas más importantes (carpeta `imagen/`).

---

### 1. Crear usuario

```bash
sudo adduser miguel
groups miguel
sudo adduser miguel2
groups miguel2
```

**Creación del usuario `miguel2`**

![Creación del usuario `miguel2`](imagen/01_adduser_miguel2.png)

**`miguel2` solo pertenece a su grupo**

![`miguel2` solo pertenece a su grupo](imagen/02_groups_miguel2_antes.png)

---

### 2. Agregar usuario a sudo

```bash
sudo usermod -aG sudo miguel2
groups miguel2
sudo -l -U miguel2
```

**`miguel2` ahora pertenece a `sudo`**

![`miguel2` ahora pertenece a `sudo`](imagen/03_groups_miguel2_sudo.png)

**`(ALL : ALL) ALL`: permisos de administrador**

![`(ALL : ALL) ALL`: permisos de administrador](imagen/04_sudo_l_miguel2.png)

---

### 3. Crear grupo

```bash
sudo addgroup guest
getent group guest
```

**Grupo `guest` creado con GID 1003**

![Grupo `guest` creado con GID 1003](imagen/05_addgroup_guest.png)

---

### 4. Crear otro usuario y agregarlo al grupo

```bash
sudo adduser julian
sudo usermod -aG guest julian
groups julian
getent group guest
```

**Grupo `guest` vacío y creación de `julian`**

![Grupo `guest` vacío y creación de `julian`](imagen/06_adduser_julian.png)

**`julian` agregado a `guest`**

![`julian` agregado a `guest`](imagen/07_usermod_julian_guest.png)

---

### 5. Eliminar usuario de un grupo

```bash
sudo gpasswd -d julian guest
groups julian
getent group guest
```

**`julian` eliminado del grupo `guest`**

![`julian` eliminado del grupo `guest`](imagen/08_gpasswd_julian.png)

**`julian` ya no está en `guest`**

![`julian` ya no está en `guest`](imagen/09_groups_julian_despues.png)

---

### 6. Eliminar usuario y grupo

```bash
sudo deluser julian
sudo delgroup guest
getent group | grep -E "julian|guest"
```

**Usuario `julian` eliminado**

![Usuario `julian` eliminado](imagen/10_deluser_julian.png)

**Grupo `guest` eliminado**

![Grupo `guest` eliminado](imagen/11_delgroup_guest.png)

**Verificación: `julian` y `guest` ya no existen**

![Verificación: `julian` y `guest` ya no existen](imagen/12_verificacion_final.png)

---

### 7. Windows: hora y fecha

```bat
time /t
date /t
```

> 📌 *Captura pendiente:* agrega aquí tu captura del CMD de Windows como `imagen/13_windows_time_date.png` y escribe `![Windows](imagen/13_windows_time_date.png)`.

---

## Conclusión

Se crearon y eliminaron usuarios y grupos, se agregó un usuario a `sudo` y a un grupo propio, y se verificó cada cambio con `groups`, `getent` y `sudo -l`. Al final, `julian` y `guest` dejaron de existir en el sistema.
