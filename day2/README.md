# Día 2 – Software, almacenamiento y servicios

Bienvenido/a al **Día 2** del curso **Red Hat Enterprise Linux 8 – Diagnostics & Troubleshooting**.  
En esta sesión vamos a reforzar tres pilares del troubleshooting en entornos Red Hat: **gestión de software**, **almacenamiento (discos/LVM)** y **servicios con systemd**.

---

## Objetivos del alumno (al finalizar el día 2)

- Gestionar software en RHEL/Rocky con **dnf/yum** con seguridad y criterio.
- Entender la diferencia entre **YUM vs DNF** y saber cuándo “pensar en repos/paquetes” al diagnosticar.
- Comprender el concepto de **parches** y tipos de actualización (seguridad, bugfix, mejora) a nivel operativo.
- Tener una visión clara del **modelo de licencias/suscripción de Red Hat** y su impacto en repositorios y soporte.
- Identificar discos y particiones y entender el rol de:
  - **/boot**
  - **swap**
  - particiones vs LVM
- Comprender el modelo **LVM** (PV → VG → LV) y sus ventajas en administración.
- Administrar servicios con **systemctl**: estado, arranque/parada, habilitar al inicio y análisis básico.
- Completar un lab: crear **filesystems persistentes** con LVM y validar el montaje con `/etc/fstab`.

---

## Temas que tocamos

> En cada tema seguimos esta estructura:
> 1) **Teoría general de Linux** (conceptos universales).
> 2) **Enfoque Red Hat** (cómo se implementa en RHEL/Rocky).
> 3) **Notas para macOS** (puntos equivalentes o diferencias clave).

### 1) Gestión de software con dnf/yum y repositorios
**Teoría general de Linux**
- Qué es un gestor de paquetes y por qué es clave en troubleshooting.
- Operaciones básicas: buscar, instalar, actualizar, eliminar, listar.
- Repositorios: qué son, cómo se habilitan/deshabilitan y cómo afectan a incidencias.
- Tipos de parches / actualizaciones (visión operativa):
  - seguridad
  - correcciones (bugfix)
  - mejoras (enhancement)

**Enfoque Red Hat (RHEL/Rocky)**
- En RHEL 8, `yum` es un wrapper de `dnf` (misma tecnología, compatibilidad histórica).
- Diferencias históricas YUM vs DNF y el impacto práctico hoy.
- **Modelo de suscripción/licencias de Red Hat (visión práctica):**
  - relación entre suscripción → repositorios oficiales → updates → soporte
  - diferencia entre RHEL (suscripción) y clones compatibles (Rocky/Alma) en labs

**Notas para macOS**
- Equivalente conceptual: Homebrew (`brew`) como gestor de paquetes.
- Diferencias clave: repositorios comunitarios, firma de paquetes y permisos.

### 2) Discos y almacenamiento: particionado básico + LVM básico
**Teoría general de Linux**
- Repaso de conceptos:
  - disco, partición, filesystem, punto de montaje
- **/boot** y por qué suele ir fuera de LVM (visión general)
- **swap**: qué es, cuándo se usa y cómo se revisa
- Modelo LVM (teoría):
  - PV (Physical Volume)
  - VG (Volume Group)
  - LV (Logical Volume)
  - Ventajas en operación: flexibilidad, crecimiento, organización

**Enfoque Red Hat (RHEL/Rocky)**
- LVM como estándar en instalaciones RHEL.
- Utilidades típicas: `lsblk`, `pvs`, `vgs`, `lvs`, `lvextend`, `xfs_growfs`.

**Notas para macOS**
- APFS y volúmenes: concepto similar a LVM, pero herramientas y flujo distintos.
- Montaje/gestión con `diskutil` (referencia conceptual, sin entrar en detalles profundos).

### 3) Servicios con systemctl (systemd)
**Teoría general de Linux**
- Qué es `systemd` y por qué es clave en troubleshooting.
- Gestión del ciclo de vida de servicios:
  - start / stop / restart
  - status
  - enable / disable

**Enfoque Red Hat (RHEL/Rocky)**
- Servicios comunes en entornos RHEL/Rocky (según lab/entorno):
  - `sshd`
  - `firewalld`
  - `NetworkManager`
  - `chronyd`
  - `rsyslog` (si aplica)
  - `crond` (según versión/config)

**Notas para macOS**
- macOS no usa `systemd`; usa `launchd` y `launchctl`.
- Enfoque equivalente: gestionar servicios/daemons con `launchctl`.

---

## Bloque teórico reforzado (primeras 2 horas)

### Tramo 0:00–0:40 — Gestión de software con visión operativa
- Diferencia real entre paquete, dependencia, repositorio y metadatos.
- Qué información necesitas antes de actualizar en un sistema crítico.
- Cómo razonar incidencias típicas: paquete no encontrado, conflicto de dependencias, repo inaccesible.
- Cuándo usar actualización completa vs actualización selectiva.

### Tramo 0:40–1:20 — Almacenamiento sin atajos mentales
- Cadena completa: disco → partición/PV → VG → LV → filesystem → montaje.
- Diferenciar problema de capa de bloque vs problema de filesystem vs problema de montaje.
- Rol de `/boot` y `swap` en operación y diagnóstico.
- Por qué LVM facilita recuperación y crecimiento, y qué riesgos introduce si no se valida.

### Tramo 1:20–2:00 — Servicios con `systemd` y lectura de estado
- Qué significa realmente “active (running)”, “failed”, “inactive”.
- Relación entre `systemctl status` y `journalctl` para encontrar causa raíz.
- Diferencia entre “arrancar ahora” y “habilitar al boot”.
- Mini-caso guiado: servicio no arranca por dependencia/configuración.

---

## Distribución recomendada de la sesión (4 horas)

- **Bloque 1 (2h):** teoría dnf/repos + almacenamiento/LVM + systemd.
- **Bloque 2 (1h 25m):** demo técnica de flujo completo (disco nuevo → LVM → fstab → verificación).
- **Bloque 3 (35m):** lab autónomo + debrief de errores frecuentes.

---

## Lab del día 2 – Filesystems persistentes con LVM

### Objetivo del lab
1. Presentar un **nuevo disco** a la VM (VirtualBox).
2. Detectar el disco desde Linux.
3. Crear una estructura LVM:
   - PV para el disco
   - VG dedicado
   - 3 a 5 LVs
4. Crear **filesystems** y montarlos.
5. Hacer los montajes **persistentes** con `/etc/fstab`.
6. Validar que todo monta correctamente y que el `fstab` está bien.

> Nota: en el repo tendrás una guía paso a paso en `labs-dia2.md` (o equivalente) con comandos y validaciones.

---

## Material asociado (repositorio)
- `comandos-dia2.md` → chuleta de comandos del día 2 (dnf/yum, discos, LVM, systemctl)
- `labs-dia2.md` → lab guiado con validaciones y checklist final
- `images/` → capturas del día 2 (si las añades)
- `../MEJORAS-CURSO.md` → propuesta pedagógica general para sesiones de 4 horas

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


##########################################