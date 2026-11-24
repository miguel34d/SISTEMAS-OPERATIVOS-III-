<h1 align="center">Práctica 2 – Cron, reinicio automático y tarea AT</h1>

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
  <img src="https://img.shields.io/badge/Bash-Terminal-4EAA25?style=flat-square&logo=gnubash&logoColor=white" alt="Bash">
</p>

---

## 📌 Descripción

Evidencias de la práctica realizada en Ubuntu Server (VMware) en el ITLA: se programaron tareas con **cron** (actualización diaria a las 11 PM y reinicio los domingos a las 3 AM) y una tarea única con **at** que borra el contenido de `/tmp`. Los comandos usados están en [GUIA.md](GUIA.md).

---

## 📸 Evidencias

### 1. Cron activo y zona horaria
Se verifica que el servicio `cron` está en `active (running)` y que la zona horaria es `America/Santo_Domingo`, necesaria para que "las 11 PM" sean la hora local. Los registros inferiores muestran a cron ejecutando una tarea.

![Cron activo y zona horaria](imagenes/01-cron-activo-zona-horaria.png)

### 2. Archivo `/etc/crontab`
Contenido del crontab del sistema con el formato de cada tarea: minuto, hora, día del mes, mes y día de la semana. Sirve de referencia para escribir las tareas propias.

![/etc/crontab](imagenes/02-etc-crontab.png)

### 3. Tareas programadas en el crontab de root
Las dos tareas de la práctica: actualización del sistema todos los días a las 11 PM (`0 23 * * *`) y reinicio los domingos a las 3 AM (`0 3 * * 0`). Se guardaron en el crontab de root para que tengan permisos.

![Dos tareas en crontab](imagenes/03-crontab-dos-tareas.png)

### 4. Prueba de ejecución de cron
Se programó una tarea temporal que escribe la fecha en un archivo cada minuto. El archivo con el mensaje `Cron funcionando` confirma que cron ejecuta las tareas.

![Prueba de cron](imagenes/04-prueba-cron.png)

### 5. Contenido de `/tmp` antes del borrado
Se crearon `archivo1.txt`, `archivo2.txt` y `archivo3.txt` en `/tmp` para poder comprobar después que `at` los elimina.

![/tmp antes](imagenes/05-tmp-antes.png)

### 6. Servicio `atd` activo
Después de instalar el paquete `at`, el servicio `atd` aparece `enabled` y en `active (running)`, listo para recibir tareas.

![atd activo](imagenes/06-atd-activo.png)

### 7. Tarea programada con `at`
Se ejecutó `at now + 1 minute` y se escribió `rm -rf /tmp/*`. Al confirmar con Ctrl+D aparece `<EOT>` y se crea el `job 2`, que queda programado.

![at programado](imagenes/07-at-programado.png)

### 8. `/tmp` vacío y cola de `at` sin tareas
Pasado el minuto, `ls /tmp` no muestra nada y `atq` no tiene tareas pendientes, lo que demuestra que el job se ejecutó y borró el contenido.

![/tmp después](imagenes/08-tmp-despues.png)

### 9. Eliminación de las tareas de cron
Se listaron las dos tareas y se borraron con `sudo crontab -ri`, confirmando con `y`. Al verificar, el sistema responde `no crontab for root`.

![Crontab eliminado](imagenes/09-crontab-eliminado.png)

---

## 👤 Autor

**Miguel Ramírez (Meli)** · Matrícula 20251367
Institución: ITLA
Materia: Sistemas Operativos 3
Profesor: Adrián Alcántara
