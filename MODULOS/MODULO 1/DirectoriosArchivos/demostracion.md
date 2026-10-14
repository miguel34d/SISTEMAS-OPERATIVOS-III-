<div align="center">

# 📸 Demostración – Práctica 4
## Gestión de Permisos de Archivos

![Alumno](https://img.shields.io/badge/Alumno-Miguel%20Ram%C3%ADrez%20Meli-2563eb?style=for-the-badge&logo=github&logoColor=white)
![Matrícula](https://img.shields.io/badge/Matr%C3%ADcula-2025--1367-16a34a?style=for-the-badge)
![ITLA](https://img.shields.io/badge/Instituci%C3%B3n-ITLA-ea580c?style=for-the-badge)
![Materia](https://img.shields.io/badge/Materia-Sistemas%20Operativos%20III-7c3aed?style=for-the-badge&logo=linux&logoColor=white)
![Profesor](https://img.shields.io/badge/Profesor-Adri%C3%A1n%20Alc%C3%A1ntara-db2777?style=for-the-badge)
![Práctica](https://img.shields.io/badge/Pr%C3%A1ctica-4%20Permisos%20de%20Archivos-0891b2?style=for-the-badge)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%20(VMware)-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)

</div>

> **Alumno:** Miguel Ramírez Meli · **Matrícula:** 2025-1367 · **ITLA**
> **Materia:** Sistemas Operativos III · **Profesor:** Adrián Alcántara
> **Entorno:** Ubuntu en VMware

Solo se incluyen las capturas más importantes (carpeta `imagen/`).

---

### 1. Crear carpeta y archivo

```bash
mkdir Materia
cd Materia
vi estudiante.txt
ls
cat estudiante.txt
```

**Archivo escrito en el editor `vi`**

![Archivo escrito en el editor `vi`](imagen/01_vi_estudiante.png)

**`ls` y `cat` muestran el archivo y su contenido**

![`ls` y `cat` muestran el archivo y su contenido](imagen/02_ls_cat_estudiante.png)

---

### 2. Cambiar permisos con chmod 0

```bash
ls -l estudiante.txt
chmod 0 estudiante.txt
ls -l estudiante.txt
cat estudiante.txt
```

**Permisos `----------` y `Permission denied`**

![Permisos `----------` y `Permission denied`](imagen/03_chmod_0.png)

---

### 3. Permisos para usuario, grupo y otros

```bash
chmod u+rwx estudiante.txt
ls -l estudiante.txt
chmod g+rwx estudiante.txt
ls -l estudiante.txt
chmod o+rwx estudiante.txt
ls -l estudiante.txt
```

**Permisos agregados a usuario, grupo y otros**

![Permisos agregados a usuario, grupo y otros](imagen/04_chmod_u_g_o.png)

---

### 4. Quitar permisos y permisos numéricos

```bash
chmod o-rwx estudiante.txt
chmod g-w estudiante.txt
ls -l estudiante.txt
chmod 644 estudiante.txt
chmod 755 estudiante.txt
chmod 600 estudiante.txt
stat -c "%a %A %n" estudiante.txt
```

**Permisos simbólicos y numéricos con `stat`**

![Permisos simbólicos y numéricos con `stat`](imagen/05_chmod_numerico.png)

---

### 5. Segunda carpeta y exploración de directorios

```bash
cd ~
mkdir Materia2
ls
cd Materia
ls
cd -
cd Materia2
pwd
ls
cd ~
```

**Creación de `Materia2` y navegación**

![Creación de `Materia2` y navegación](imagen/06_segunda_carpeta.png)

---

### 6. Permisos en carpetas

```bash
ls -ld Materia Materia2
chmod 000 Materia2
cd Materia2
chmod 755 Materia2
cd Materia2
pwd
cd ~
```

**`cd` denegado con `000` y permitido con `755`**

![`cd` denegado con `000` y permitido con `755`](imagen/07_permisos_carpetas.png)

---

### 7. Copiar archivo a otra carpeta

```bash
cd ~/Materia
cp estudiante.txt ~/Materia2
ls
ls -l ~/Materia2
cd ~
```

**Archivo copiado a `Materia2`**

![Archivo copiado a `Materia2`](imagen/08_copiar_archivo.png)

---

### 8. Cambiar dueño y grupo

```bash
sudo chown root ~/Materia2/estudiante.txt
ls -l ~/Materia2
sudo chgrp sudo ~/Materia2/estudiante.txt
ls -l ~/Materia2
sudo chown miguel:miguel ~/Materia2/estudiante.txt
ls -l ~/Materia2
```

**Cambio de dueño y grupo, y restauración**

![Cambio de dueño y grupo, y restauración](imagen/09_chown_chgrp.png)

---

### 9. Probar permisos con otro usuario

```bash
sudo adduser --disabled-password --gecos "" prueba
chmod o+x ~
chmod 600 ~/Materia/estudiante.txt
sudo -u prueba cat ~/Materia/estudiante.txt
chmod 644 ~/Materia/estudiante.txt
sudo -u prueba cat ~/Materia/estudiante.txt
chmod o-x ~
```

**`prueba` denegado con `600` y permitido con `644`**

![`prueba` denegado con `600` y permitido con `644`](imagen/10_prueba_otro_usuario.png)

---

### 10. Eliminar carpetas y usuario de prueba

```bash
cd ~
rm -r Materia
ls
rm -r Materia2
ls
sudo deluser --remove-home prueba
```

**Carpetas y usuario de prueba eliminados**

![Carpetas y usuario de prueba eliminados](imagen/11_eliminar_carpetas.png)

---

## Conclusión

Se crearon archivos y carpetas, se modificaron sus permisos de forma simbólica y numérica, se copiaron y se cambió su dueño y grupo. La prueba con un segundo usuario mostró que un archivo con `600` es denegado y con `644` es legible. Al final se eliminaron las carpetas y el usuario de prueba.
