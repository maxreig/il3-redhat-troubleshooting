# Día 1 – Labs (Instalación, usuarios y permisos)

Curso **Red Hat Enterprise Linux 8 – Diagnostics & Troubleshooting**  
Material de apoyo – Día 1

> Objetivo general del día: terminar con un **laboratorio funcional** y ser capaz de diagnosticar incidencias típicas de **permisos**, **usuarios/grupos** y **orientación** en el sistema de archivos.

---

## ✅ Lab principal – “Entorno listo + permisos correctos”

### Objetivo
1. Tener el sistema instalado (RHEL 8 / Rocky equivalente) y operativo.
2. Validar checks básicos tras el primer arranque.
3. Crear usuarios `ana` y `bruno`, y el grupo `dev`.
4. Crear `/srv/proyecto` y aplicar permisos correctos para trabajo en equipo (grupo `dev`).
5. Validar que ambos usuarios pueden crear ficheros dentro del directorio.

---

### Paso 0 – Preparación (si aplica)
- Asegúrate de tener un usuario con `sudo` o entra como `root`.
- Si trabajas con VM: guarda un **snapshot** antes del lab (recomendado).

---

### Paso 1 – Checks básicos tras el primer arranque

```bash
cat /etc/redhat-release
hostnamectl
ip a
free -h
df -h
timedatectl
```

#### Problemas típicos (detección rápida)
- **Sin red / DNS**: revisa `ip a` y prueba `ping -c 3 8.8.8.8` / `ping -c 3 google.com`.
- **Particionado insuficiente** (por ejemplo `/var` o `/` pequeños): valida con `df -h`.
- **Usuario sin privilegios**: prueba `sudo -v` o revisa el grupo `wheel`.

---

### Paso 2 – Crear usuarios y grupo

```bash
sudo groupadd dev

sudo useradd ana
sudo useradd bruno

sudo passwd ana
sudo passwd bruno

sudo usermod -aG dev ana
sudo usermod -aG dev bruno
```

Validación:

```bash
id ana
id bruno
getent group dev
```

> Nota: si el usuario ya existe, `useradd` fallará. Puedes borrarlo (si es lab) con `sudo userdel -r ana` o elegir otro nombre.

---

### Paso 3 – Crear directorio del proyecto y aplicar permisos

Crear carpeta y asignar grupo:

```bash
sudo mkdir -p /srv/proyecto
sudo chown root:dev /srv/proyecto
```

Aplicar permisos recomendados **con setgid** (herencia del grupo):

```bash
sudo chmod 2770 /srv/proyecto
```

Qué significa `2770`:
- `2` → **setgid** en directorio (los ficheros nuevos heredan el grupo `dev`)
- `770` → `rwx` para propietario y grupo, nada para “otros”

Validación:

```bash
ls -ld /srv/proyecto
# Esperado: drwxrws--- ... root dev ... /srv/proyecto
```

---

### Paso 4 – Prueba funcional con ambos usuarios

Probar con `ana`:

```bash
sudo -iu ana
id
touch /srv/proyecto/prueba_ana.txt
ls -l /srv/proyecto/prueba_ana.txt
exit
```

Probar con `bruno`:

```bash
sudo -iu bruno
id
touch /srv/proyecto/prueba_bruno.txt
ls -l /srv/proyecto/prueba_bruno.txt
exit
```

**Comprobación clave:** los ficheros deberían quedar con grupo `dev` gracias al setgid del directorio.

---

## 🧪 Extra A – Mini-taller: diagnosticar “Permission denied”

### Objetivo
Aplicar un método rápido y repetible para resolver la incidencia más común del mundo Linux:
**“Permission denied”**.

### Método en 6 preguntas
1) ¿Qué usuario ejecuta la acción? (`whoami` / `id`)  
2) ¿Sobre qué ruta exacta falla? (usa **ruta absoluta**)  
3) ¿Propietario y grupo del recurso? (`ls -l` / `ls -ld`)  
4) ¿Permisos del recurso y del directorio padre? (**x** para entrar)  
5) ¿Grupo efectivo del usuario? (`id`, `groups`)  
6) ¿Afecta `umask` (máscara por defecto) a ficheros nuevos?

### Plantilla de comandos

```bash
whoami
id
namei -l /ruta/objetivo
ls -ld /ruta/objetivo
getfacl /ruta/objetivo 2>/dev/null || true
umask
```

> Tip: `namei -l` es brutal para ver permisos **en toda la ruta** (padres incluidos).

---

## 🧭 Extra B – Mini-taller: “no encuentro el fichero / me pierdo”

### Objetivo
Orientarte sin perder tiempo y localizar ficheros/configs rápidamente.

### Orientación

```bash
pwd
ls -lah
cd -
which comando
```

Buscar ficheros (ejemplo):

```bash
find /etc -maxdepth 2 -name "*.conf" 2>/dev/null | head
```

### Primer vistazo a logs (sin deep dive todavía)

```bash
ls -lah /var/log
journalctl -n 50 --no-pager
```

> Nota: en el **Día 4** entraremos fuerte en logs y troubleshooting. Hoy es “orientación y supervivencia”.

---

## ✅ Cierre del día (checklist final)

Antes de terminar, confirma:
- [ ] Instalación OK (versión correcta)
- [ ] Terminal OK
- [ ] Red/DNS OK
- [ ] Usuarios `ana` y `bruno` creados
- [ ] Grupo `dev` creado y asignado
- [ ] `/srv/proyecto` con **grupo dev** y permisos **2770**
- [ ] Ambos usuarios crean ficheros en `/srv/proyecto`
- [ ] (Recomendado) Snapshot guardado

---

## 📌 Tarea opcional (entre sesiones)
- Leer `man chmod` y `man useradd`
- Practicar vim 5 minutos (abrir → editar → buscar → guardar → salir)

## 🖥️ Extra C – Instalar GUI GNOME (si tienes Rocky 9 *minimal*)

> Si tu instalación es **mínima** y quieres tener escritorio (GNOME) para la parte formativa, puedes instalarlo con estos pasos.  
> Requisitos: **conectividad a Internet**, espacio en disco suficiente y repos habilitados.

### Opción recomendada (entorno “Server with GUI”)

```bash

### 1) Actualiza metadatos y sistema (opcional pero recomendado):
sudo dnf clean all
sudo dnf -y update

### 2) Instala el entorno gráfico:
sudo dnf -y groupinstall "Server with GUI"

### 3) Arrancar en modo gráfico por defecto + habilitar el gestor de login (GDM):
sudo systemctl set-default graphical.target
sudo systemctl enable --now gdm
sudo reboot

### Verificacion despues del reinicio
systemctl get-default
systemctl status gdm --no-pager

```
