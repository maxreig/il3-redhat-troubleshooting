# Día 2 – Software, almacenamiento y servicios

## Índice del Día 2 (material asociado)

- [`tema1-gestion-software.md`](tema1-gestion-software.md) → teoría ampliada del tema 1.
- [`tema2-almacenamiento-lvm.md`](tema2-almacenamiento-lvm.md) → teoría ampliada del tema 2.
- [`tema3-servicios-systemctl.md`](tema3-servicios-systemctl.md) → teoría ampliada del tema 3.
- [`comandos-dia2.md`](comandos-dia2.md) → chuleta de comandos del día 2 (`dnf/yum`, discos, LVM, `systemctl`).
- [`labs-dia2.md`](labs-dia2.md) → lab guiado con validaciones y checklist final.

---

# Índice visual del Día 2

![Índice visual del Día 2](images/indice-visual-dia2.svg)

> Idea clave: en troubleshooting, piensa por capas. Primero identifica si el problema es de **software**, **almacenamiento** o **servicio**; después aplica comandos de esa capa y valida antes de pasar a la siguiente.

---

Bienvenido/a al **Día 2** del curso **Red Hat Enterprise Linux 8 – Diagnostics & Troubleshooting**.  
En esta sesión vamos a reforzar tres pilares del troubleshooting en entornos Red Hat: **gestión de software**, **almacenamiento (discos/LVM)** y **servicios con systemd**.

---

## Recordatorio del Día 1

En el primer día dejamos una VM RHEL/Rocky instalada y validada con checks básicos de sistema.  
Trabajamos la estructura del sistema de archivos y operaciones esenciales con ficheros y directorios.  
Reforzamos permisos, propietarios y `umask` para diagnosticar errores típicos de acceso.  
También practicamos edición básica con `vim` y administración de usuarios/grupos, cerrando con un lab de permisos en carpeta compartida.

---

## Objetivos del alumno (al finalizar el día 2)

- Recordar cosas del día 1.
- Gestionar software con `dnf/yum` y resolver incidencias básicas de repos y dependencias.
- Entender almacenamiento en capas (disco/partición/LVM/filesystem/montaje) y validar persistencia en `/etc/fstab`.
- Administrar servicios con `systemctl` y usar `journalctl` para diagnóstico rápido.
- Completar el lab del día creando filesystems persistentes sobre LVM.

---

## Distribución recomendada de la sesión (4 horas)

- **Bloque 1 (1h 15m):** repaso guiado de conceptos clave (software, almacenamiento y servicios).
- **Bloque 2 (2h 15m):** laboratorios prácticos del Día 2.
- **Bloque 3 (30m):** revisión de resultados, errores frecuentes y cierre.

---

## Laboratorios del Día 2

### Lab 0 – Gestión mínima de software con `dnf`
**Tiempo estimado alumno:** 20–30 minutos.

1. Verificar repos disponibles y refrescar metadatos.
2. Comprobar si un paquete está instalado (`rpm -q`).
3. Buscar paquetes en repositorios (`dnf search` / `dnf info`).
4. Instalar y validar una herramienta (ejemplo: `tree`).
5. Revisar historial básico de cambios (`dnf history`).

### Lab 1 – Filesystems persistentes con LVM
**Tiempo estimado alumno:** 50–70 minutos.

1. Presentar un **nuevo disco** a la VM (VirtualBox).
2. Detectar el disco desde Linux.
3. Crear una estructura LVM:
   - PV para el disco
   - VG dedicado
   - 3 a 5 LVs
4. Crear **filesystems** y montarlos.
5. Hacer los montajes **persistentes** con `/etc/fstab`.
6. Validar que todo monta correctamente y que el `fstab` está bien.

### Lab 2 – Ampliar SWAP con un disco nuevo (VirtualBox)
**Tiempo estimado alumno:** 30–45 minutos.

1. Presentar un segundo disco a la VM.
2. Detectarlo con `lsblk`.
3. Crear espacio de swap (recomendado con LVM en entorno RHEL).
4. Activar la swap con `mkswap` y `swapon`.
5. Dejarla persistente en `/etc/fstab`.
6. Validar con `swapon --show` y `free -h`.

### Lab 3 – Crear un servicio `systemd` desde un script simple
**Tiempo estimado alumno:** 25–35 minutos.

1. Crear un script básico (ejemplo: chequeo de uso de filesystems con `df -h`).
2. Guardarlo como ejecutable en `/usr/local/bin/`.
3. Crear una unidad `systemd` (por ejemplo `check-fs.service`).
4. Recargar unidades, arrancar y revisar estado con `systemctl`.
5. Validar salida en logs con `journalctl`.

> Nota: puedes centralizar los pasos de estos labs en `labs-dia2.md` para ejecución guiada en clase.

---

## Cierre y preparación del día 3

Antes de terminar el día 2, verifica:
- Repos y gestión de paquetes controlados (sabes buscar/instalar/actualizar con criterio).
- Estructura LVM creada y comprendida (PV/VG/LV).
- Montajes persistentes verificados con `/etc/fstab`.
- Manejas `systemctl` para revisar y operar servicios comunes.

En el **día 3** entraremos con:
- redes (NetworkManager / `nmcli`)
- seguridad base (`firewalld`) y SELinux (modo básico)
- lab integrando disco + servicio + firewall

Mención final del **laboratorio del Día 2**: la sesión práctica combina gestión de software con `dnf`, LVM persistente, ampliación de swap y creación de un servicio `systemd` simple.
