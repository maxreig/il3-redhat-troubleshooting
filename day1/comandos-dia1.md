# Día 1 – Comandos esenciales (cheatsheet)

Curso **Red Hat Enterprise Linux 8 – Diagnostics & Troubleshooting**  
Material de apoyo – Día 1

Este documento recoge los **comandos más utilizados** durante el Día 1 del curso.  
Está pensado como **chuleta rápida para admins junior**, tanto en laboratorio como en incidencias reales.

---

## 1) Checks básicos tras la instalación / primer arranque

Usa esto para validar rápido: **versión**, **red**, **memoria**, **disco**, **hora**.

```bash
cat /etc/redhat-release
hostname
hostnamectl
uptime
date
timedatectl

ip a
ip -4 a
ip r

free -h
df -h
df -hTP
```

Pruebas rápidas de conectividad / DNS:

```bash
ping -c 3 8.8.8.8
ping -c 3 google.com
curl -I https://www.google.com
```

Extras útiles (si algo “no cuadra”):

```bash
uname -r
dmesg | tail -n 50
```

---

## 2) Atajos útiles en terminal y documentación

Comandos:

```bash
clear
history | tail
man comando
info comando
```

Atajos de teclado:
- `Ctrl + R` → buscar en el histórico
- `Ctrl + C` → cortar proceso
- `Ctrl + L` → limpiar pantalla
- `Ctrl + D` → salir (EOF) / cerrar sesión

Tips:
- En `man`: `/texto` busca, `n` siguiente, `q` salir
- Si no recuerdas opciones: `comando --help`

---

## 3) Orientación en el sistema de archivos

Comandos básicos:

```bash
pwd
ls
ls -l
ls -lah
cd /etc
cd ~
cd -
cd ..
```

Trucos rápidos:
- `cd -` → vuelve al directorio anterior
- `TAB` → autocompletar rutas y comandos

Ver qué ocupa espacio (directorio actual):

```bash
du -sh * 2>/dev/null | sort -h
```

---

## 4) Gestión básica de ficheros

Crear zona de trabajo:

```bash
mkdir -p ~/lab1
cd ~/lab1
touch a.txt b.txt
```

Copiar / mover / borrar:

```bash
cp a.txt copia_a.txt
mv b.txt renombrado.txt
rm copia_a.txt
```

Copiar directorios preservando atributos (recomendado):

```bash
cp -a dir1 dir2
```

Buenas prácticas (modo admin):
- Antes de borrar, **lista lo que vas a borrar**
- Evita `rm -rf` “a ciegas”
- Si estás bajo presión, usa rutas absolutas

Ejemplo seguro:

```bash
ls archivos_a_borrar*
rm archivos_a_borrar*
```

---

## 5) Buscar ficheros y comandos (orientación rápida)

Encontrar ficheros (ejemplos):

```bash
find /etc -maxdepth 2 -name "*.conf" 2>/dev/null | head
find / -name "nombre_fichero" 2>/dev/null | head
```

Saber dónde está un comando:

```bash
which ls
which vim
type ls
```

Buscar texto dentro de ficheros (muy útil en troubleshooting):

```bash
grep -R "cadena" /etc 2>/dev/null | head
```

---

## 6) Permisos y propietarios (chmod / chown / umask)

Leer permisos:

```bash
ls -l
ls -ld directorio
stat fichero
```

Ejemplo típico:

```text
-rw-r----- 1 ana dev 120 Jan 1 10:00 informe.txt
```

- Tres conjuntos: usuario (u), grupo (g), otros (o)
- Tres permisos: lectura (r), escritura (w), ejecución (x)

📌 Directorios:
- `x` = entrar/atravesar
- `r` = listar

Cambiar permisos:

```bash
chmod 640 informe.txt
chmod 755 script.sh
chmod u+x script.sh
```

Cambiar propietario y grupo:

```bash
chown ana:dev informe.txt
chown -R ana:dev /ruta/directorio
```

Ver umask:

```bash
umask
```

Ver ACLs (si existen) y entender permisos “raros”:

```bash
getfacl fichero 2>/dev/null || true
```

---

## 7) Diagnóstico rápido: “Permission denied”

Método en 6 preguntas:

1. ¿Qué usuario ejecuta la acción? (`whoami` / `id`)
2. ¿Sobre qué ruta exacta falla? (ruta absoluta)
3. ¿Propietario y grupo del recurso? (`ls -l` / `ls -ld`)
4. ¿Permisos del recurso y del directorio padre? (**x para entrar**)
5. ¿Grupo efectivo del usuario? (`id`, `groups`)
6. ¿Afecta `umask` a ficheros nuevos?

Comandos (plantilla):

```bash
whoami
id
groups
namei -l /ruta/objetivo
ls -ld /ruta/objetivo
getfacl /ruta/objetivo 2>/dev/null || true
umask
```

Extra pro (para diagnosticar herencia de grupo en directorios compartidos):

```bash
ls -ld /srv/proyecto
# Si ves una "s" en el grupo (ej: drwxrws---), el directorio tiene setgid y hereda grupo.
```

---

## 8) Vim (lo imprescindible para admins)

Abrir fichero:

```bash
vim /etc/hosts
```

Modo supervivencia:
- `i` → insertar (escribir)
- `Esc` → volver a normal
- `:w` → guardar
- `:q` → salir
- `:wq` → guardar y salir
- `:q!` → salir sin guardar
- `/texto` → buscar
- `n` / `N` → siguiente / anterior

Chuleta rápida:

```text
Mover: h j k l
Inicio/fin línea: 0 / $
Borrar línea: dd
Deshacer/rehacer: u / Ctrl+r
Copiar/pegar: yy / p
Salir sin guardar: :q!
```

Alternativa “sin vim” (para admins muy junior, a veces salva):

```bash
nano /etc/hosts
```

---

## 9) Usuarios y grupos (base para permisos)

¿Quién soy?

```bash
whoami
id
```

Crear usuarios y password:

```bash
sudo useradd ana
sudo passwd ana
```

Crear grupo y añadir usuarios:

```bash
sudo groupadd dev
sudo usermod -aG dev ana
```

Validar:

```bash
id ana
getent passwd ana
getent group dev
```

Ficheros clave:
- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`

Formatos:

```text
# /etc/passwd
usuario:x:UID:GID:comentario:/home/usuario:/bin/bash

# /etc/group
grupo:x:GID:miembro1,miembro2
```

---

## 10) Lab Día 1 (comandos de referencia)

Objetivo:
- Crear usuarios `ana` y `bruno`
- Crear grupo `dev`
- Crear `/srv/proyecto` y asignarlo a `dev`
- Permisos: `dev` puede leer/escribir; otros no
- Usar **setgid** en el directorio para heredar grupo

Comandos:

```bash
sudo groupadd dev

sudo useradd ana
sudo useradd bruno

sudo usermod -aG dev ana
sudo usermod -aG dev bruno

sudo mkdir -p /srv/proyecto
sudo chown root:dev /srv/proyecto
sudo chmod 2770 /srv/proyecto
```

Validación (check rápido):

```bash
ls -ld /srv/proyecto
id ana
id bruno
```

Prueba funcional (simular acceso):

```bash
sudo -iu ana
touch /srv/proyecto/prueba_ana.txt
exit

sudo -iu bruno
touch /srv/proyecto/prueba_bruno.txt
exit
```

---

## 11) Primer vistazo a logs (solo orientación)

Listar logs clásicos:

```bash
ls -lah /var/log
```

Journald (vista general):

```bash
journalctl -n 50 --no-pager
```

---

## 12) Comandos “salvavidas” (orden recomendado)

Cuando algo falla, empieza por aquí:

```bash
uptime
df -h
free -h
ip a
journalctl -xe
```

🧠 Consejo final: en troubleshooting, **el orden importa**.  
Piensa siempre: **sistema → red → permisos → servicios → logs**.
