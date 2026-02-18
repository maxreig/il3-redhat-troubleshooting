# Día 1 – Instalación, sistema de archivos y usuarios

> Ruta recomendada del material del Día 1: [`Index.md`](Index.md)

Bienvenido/a al **Día 1** del curso **Red Hat Enterprise Linux 8 – Diagnostics & Troubleshooting**.  
El objetivo de esta sesión es que salgas con un **laboratorio funcional** desde el primer día y con la base necesaria para afrontar el troubleshooting del resto del curso.

---

## Objetivos del alumno (al finalizar el día 1)

- Tener una VM RHEL/Rocky **instalada y operativa** (GUI/CLI) y lista para trabajar.
- Realizar **checks básicos** del sistema (versión, red, memoria, disco, hora).
- Moverte con soltura por el **sistema de archivos** y entender su estructura.
- Ejecutar operaciones habituales con ficheros y directorios (**crear, copiar, mover, borrar**) con seguridad.
- Entender **permisos, propietarios y umask** y su impacto directo en incidencias típicas (“Permission denied”).
- Editar ficheros de configuración con **vim** en modo supervivencia.
- Crear y administrar **usuarios y grupos** con criterio (incluyendo verificación en ficheros clave).
- Completar un lab de cierre: **usuarios + grupo + carpeta compartida + permisos correctos**.

---

## Temas que tocamos

### 1) Enfoque de troubleshooting (cómo trabajaremos)
- Metodología “aprender haciendo”: explicación corta → demo → lab → mini-repaso
- Checklist para evitar bloqueos en remoto

### 2) Instalación / laboratorio (RHEL/Rocky)
- Creación de VM (VirtualBox) y primeros pasos
- Checks tras el primer arranque
- (Opcional) Guest Additions y actualización del sistema

### 3) Sistema de archivos
- Estructura de directorios y mapa mental mínimo
- Navegación y comandos de supervivencia
- Gestión básica de ficheros y directorios

### 4) Permisos y propietarios
- `chmod`, `chown`, `umask`
- Diagnóstico rápido de “Permission denied”
- Conceptos clave en directorios (permiso `x`)

### 5) Edición con vim (lo imprescindible)
- Modos, guardar/salir, buscar
- Chuleta mínima para admins

### 6) Usuarios y grupos
- `useradd`, `groupadd`, `usermod`, `passwd`
- Ficheros clave: `/etc/passwd`, `/etc/shadow`, `/etc/group`
- Verificación con `id` / `getent`

---

## Bloque teórico reforzado (primeras 2 horas)

> Recomendado si quieres profundizar en los apartados con más “miga” antes del lab.

### Tramo 0:00–0:30 — Cómo piensa un administrador cuando diagnostica
- Diferencia entre **síntoma** y **causa raíz**.
- Método base: observación → hipótesis → comprobación → corrección → validación.
- Regla de oro: no corregir sin tener una evidencia mínima.

### Tramo 0:30–1:00 — Sistema de archivos como mapa mental de Linux
- Jerarquía FHS y qué esperar en `/etc`, `/var`, `/home`, `/tmp`, `/usr`, `/opt`.
- Qué rutas se tocan en operación diaria y cuáles conviene tocar con más cuidado.
- Relación entre estructura de disco y troubleshooting (ejemplo: “no arranca servicio por falta de espacio en `/var`”).

### Tramo 1:00–1:30 — Permisos bien entendidos (no solo comandos)
- Modelo `u/g/o` y diferencia entre permisos de fichero y de directorio.
- Por qué en directorios el bit `x` representa “atravesar/acceder”.
- `umask` como política por defecto: impacto directo en incidencias de acceso.
- Lectura de `ls -l` para localizar bloqueos en menos de 1 minuto.

### Tramo 1:30–2:00 — Usuarios, grupos e identidad del proceso
- Quién eres (`id`) vs con qué permisos efectivos actúas en cada contexto.
- Grupos primarios/secundarios y su impacto en carpetas compartidas.
- Ficheros de identidad (`/etc/passwd`, `/etc/shadow`, `/etc/group`) y cómo verificarlos sin romper nada.
- Mini-caso guiado: “usuario existe, pero no puede escribir en carpeta compartida”.

---

## Distribución recomendada de la sesión (4 horas)

- **Bloque 1 (2h):** teoría guiada + preguntas de diagnóstico.
- **Bloque 2 (1h 20m):** demo + lab de usuarios/permisos.
- **Bloque 3 (40m):** revisión de errores frecuentes + checklist final.

---

## Material asociado

- `comandos-dia1.md` → chuleta de comandos del día 1
- (Opcional) `labs-dia1.md` → guía de laboratorio y validaciones
- `../MEJORAS-CURSO.md` → propuesta pedagógica general para sesiones de 4 horas

---

## Cierre y preparación del día 2

Antes de terminar el día 1, verifica:
- La VM arranca y tienes terminal operativa
- Red / DNS OK
- Usuarios y grupo creados
- Permisos del lab correctos

En el **día 2** entraremos con:
- Gestión de software (dnf/yum y repositorios)
- Almacenamiento (montajes persistentes y LVM básico)
- Servicios con `systemctl`
