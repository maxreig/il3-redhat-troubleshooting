# IL3-redhat-troubleshooting
## Curso REDHAT: Diagnostics & Troubleshooting (RHEL 9 / Rocky)

<p align="center">
  <img src="day1/images/linux_spartan.jpeg" alt="Tux Spartan" width="300"><br>
  <em>Tux Spartan</em>
</p>

Repositorio de apoyo para el curso **Red Hat Enterprise Linux 8 – Diagnostics & Troubleshooting**.  
Aquí encontrarás **material por sesión**, **chuletas de comandos**, **labs** y documentación complementaria para tenerlo todo a mano durante el curso.

---
## Accesos directos

### Día 1
- 📘 [Día 1 – README](day1/README.md)
- 🧰 [Comandos Día 1](day1/comandos-dia1.md)
- 🧪 [Labs Día 1](day1/labs-dia1.md)
  
### Día 2  
- 📘 [Día 2 – README](day2/README.md) _(El jueves que viene lo vemos juntos)_
- 🧰 [Comandos Día 2](day2/comandos-dia2.md)
- 🧪 [Labs Día 2](day2/labs-dia2.md)

### Día 3
- 📘 [Día 3 – Índice](day3/Index.md)
- 🧰 [Comandos Día 3](day3/comandos-dia3.md)
- 🧪 [Labs Día 3](day3/labs-dia3.md)

### Día 4
- 📘 [Día 4 – Índice](day4/Index.md)
- 🧰 [Comandos Día 4](day4/comandos-dia4.md)
- 🧪 [Labs Día 4](day4/labs-dia4.md)

## Objetivo del curso

Al finalizar el curso, el alumno será capaz de:

- Preparar un **laboratorio funcional** (RHEL 8 o compatible como Rocky) y moverse con soltura por el sistema.
- Aplicar un método de **diagnóstico estructurado** para incidencias reales.
- Identificar y resolver problemas típicos de administración Linux:
  - permisos y accesos
  - servicios que no arrancan
  - conectividad y firewall
  - almacenamiento (discos, LVM, montajes persistentes)
  - consumo de recursos (CPU/RAM/disco)
  - análisis de logs (journald y `/var/log`)
  - nociones básicas de SELinux

---

## Estructura del repositorio

> La estructura puede crecer con el avance del curso, pero la idea es mantenerlo simple y ordenado.

```text
il3-redhat-troubleshooting/
├── README.md
├── day1/
│   ├── README.md
│   ├── comandos-dia1.md
│   └── images/
├── day2/
├── day3/
└── day4/
```

---

## Temario del curso (por sesiones)

### Sesión 1 – Instalación, sistema de archivos y usuarios
- Instalación RHEL 8 (modo gráfico y texto). Primer arranque y entorno de escritorio.
- Estructura del sistema de archivos. Navegación: `cd`, `ls`, `pwd`, etc.
- Operaciones con archivos: `touch`, `cp`, `mv`, `rm`.
- Permisos y propietarios: `chmod`, `chown`, `umask`.
- Editores: `vim`.
- Gestión de usuarios y grupos; archivos clave: `/etc/passwd`, `/etc/shadow`, `/etc/group`.
- **Lab:** instalación mínima + creación de usuarios y políticas de permisos.

### Sesión 2 – Software, almacenamiento
- Gestión de software con `dnf`/`yum` y repositorios.
- Discos y almacenamiento: particionado básico, **LVM básico**.
- Servicios con `systemctl`.
- **Lab:** creación de filesystems persistentes a través de una estructura LVM.

### Sesión 3 – Redes y otros servicios
- Configuración y diagnóstico con NetworkManager (`nmcli`) (y alternativas según entorno).
- Seguridad básica: `firewalld` y nociones de **SELinux** (modo básico).
- **Lab:** añadir y preparar un disco, habilitar un servicio y abrir un puerto en `firewalld`.

### Sesión 4 – Monitorización y troubleshooting
- Monitorización básica: CPU, RAM, disco (`top`, `free`, `df`, `iostat`).
- Logs y journald: `journalctl`, `/var/log`.
- Diagnóstico de problemas comunes (conectividad, servicios que no arrancan, falta de espacio, accesos).
- **Lab final:** escenario integrado de troubleshooting guiado.

---

## Cómo usar este repositorio

- Cada carpeta `dayX/` contiene el material asociado a la sesión.
- La **chuleta de comandos** está en `comandos-diaX.md`.
- Las imágenes se guardan dentro de `dayX/images/` y se referencian desde los README.

Ejemplo para insertar una imagen centrada:

```html
<p align="center">
  <img src="day1/images/01-checklist.png" alt="Checklist de arranque" width="700">
</p>
```

---

## Requisitos recomendados para el laboratorio

- Windows / macOS / Linux con **VirtualBox** (o alternativa equivalente).
- VM: **RHEL 9** o compatible (por ejemplo Rocky Linux).
- Recomendado: snapshots para poder volver atrás en los labs.

---

## Aviso

Este repositorio está orientado a **uso formativo** durante el curso.  
El contenido puede actualizarse a medida que avancemos en las sesiones.
