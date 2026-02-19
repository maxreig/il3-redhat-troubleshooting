# Labs Dia 3 - Permisos, usuarios y networking basico

## Enfoque del dia

Este bloque es 100% practico. El guion te marca pasos, pero cada alumno ejecuta, verifica y corrige.

## Reglas de trabajo

1. Ejecuta los comandos tu mismo/a; no copies sin leer.
1. Tras cada bloque, valida salida esperada.
1. Si algo falla, documenta que comando fallo y que mensaje devolvio.

---

## Requisitos previos

1. VM RHEL/Rocky operativa con acceso root o `sudo`.
1. Snapshot recomendado antes de empezar.
1. Para Lab 0: anadir un disco nuevo de 20GB en VirtualBox.

---

## Lab 0 (35 min) - Ampliar `/tmp` a 20GB (arrastre desde dia anterior)

Objetivo: dejar `/tmp` con tamano de **20GB**, anadiendo un disco nuevo porque el VG actual no tiene espacio libre.

Tiempo estimado alumno: 35 min.

### Paso 0. Anadir disco en VirtualBox

1. Apagar la VM.
1. En VirtualBox: `Settings -> Storage -> Add Hard Disk -> 20GB`.
1. Arrancar la VM.

### Paso 1. Tomar evidencia inicial

```bash
lsblk -f
df -hT /tmp
findmnt /tmp
vgs
lvs
```

Resultado esperado:

1. `/tmp` identificado (idealmente en un LV dedicado).
1. El VG del LV de `/tmp` sin espacio libre suficiente.
1. Nuevo disco visible (ejemplo `/dev/sdb`).

### Paso 2. Presentar y preparar el disco nuevo para LVM

```bash
# Si no aparece de inmediato, fuerza rescan SCSI
for h in /sys/class/scsi_host/host*; do echo "- - -" > "$h/scan"; done

lsblk -f
pvcreate /dev/sdb
```

### Paso 3. Extender el VG donde vive `/tmp`

1. Localiza el LV de `/tmp`:

```bash
findmnt -no SOURCE /tmp
```

2. Localiza su VG:

```bash
lvs -o lv_name,vg_name,lv_path
```

3. Extiende el VG correcto (sustituye `<VG_TMP>`):

```bash
vgextend <VG_TMP> /dev/sdb
vgs
```

### Paso 4. Ampliar el LV de `/tmp` a 20GB

```bash
# Sustituye <LV_TMP_PATH> por la ruta real, ejemplo /dev/rhel/tmp
lvextend -L 20G -r <LV_TMP_PATH>
```

### Paso 5. Verificacion final

```bash
df -hT /tmp
lvs
vgs
```

Checklist rapido:

1. `/tmp` muestra 20GB.
1. El filesystem sigue montado correctamente.
1. No hay errores de LVM en el proceso.

Nota docente:

- Si `/tmp` no esta en un LV dedicado (por ejemplo cuelga de `/`), no improvisar en caliente. Avisar al instructor para decidir migracion controlada de `/tmp`.

---

## Lab 1 (30 min) - Usuarios y grupos (crear y customizar)

Objetivo: crear usuarios reales de trabajo y validar pertenencia a grupos.

Tiempo estimado alumno: 30 min.

### Paso 1. Crear grupos del laboratorio

```bash
groupadd proyecto
groupadd soporte
getent group proyecto soporte
```

### Paso 2. Crear usuarios

```bash
useradd -m -s /bin/bash -c "Ana Proyecto" ana
useradd -m -s /bin/bash -c "Bruno Soporte" bruno
passwd ana
passwd bruno
```

### Paso 3. Asignar grupos

```bash
usermod -aG proyecto ana
usermod -aG proyecto,soporte bruno
id ana
id bruno
```

### Paso 4. Customizar usuario (perfil shell)

```bash
cat >> /home/ana/.bashrc <<'EOB'
# Lab custom prompt
export PS1='[LAB-ANA \u@\h \W]\\$ '
alias ll='ls -lah --color=auto'
EOB
chown ana:ana /home/ana/.bashrc
```

### Paso 5. Verificar login y customizacion

```bash
su - ana -c 'echo $PS1; alias ll; id'
su - bruno -c 'id'
```

Checklist rapido:

1. Usuarios creados con home y shell.
1. Grupos aplicados correctamente.
1. `ana` entra con prompt y alias custom.

---

## Lab 2 (25 min) - Permisos con enfoque educativo

Objetivo: entender como cambian accesos reales con `chmod`, `chown`, `sgid` y sticky bit.

Tiempo estimado alumno: 25 min.

### Paso 1. Preparar entorno de practica

```bash
mkdir -p /lab/permisos/{proyecto,intercambio}
touch /lab/permisos/proyecto/plan.txt
chmod 770 /lab/permisos/proyecto
chmod 1777 /lab/permisos/intercambio
ls -ld /lab/permisos/proyecto /lab/permisos/intercambio
```

### Paso 2. Probar permisos por modo octal y simbolico

```bash
chmod 640 /lab/permisos/proyecto/plan.txt
ls -l /lab/permisos/proyecto/plan.txt
chmod g+w,o-r /lab/permisos/proyecto/plan.txt
ls -l /lab/permisos/proyecto/plan.txt
```

### Paso 3. Explicar herencia de grupo con `sgid`

```bash
chgrp root /lab/permisos/proyecto
chmod 2770 /lab/permisos/proyecto
ls -ld /lab/permisos/proyecto
```

### Paso 4. Comprobar sticky bit en intercambio

```bash
ls -ld /lab/permisos/intercambio
```

Interpretacion esperada:

1. `drwxrws---` en `proyecto` implica `sgid` (hereda grupo del directorio).
1. `drwxrwxrwt` en `intercambio` implica sticky bit (no borrado entre usuarios sin permisos).

---

## Lab 3 (20 min) - Umask basico en usuario

Objetivo: ver como `umask` afecta permisos por defecto y dejarlo persistente para un usuario.

Tiempo estimado alumno: 20 min.

### Paso 1. Comprobar umask actual

```bash
su - ana -c 'umask'
```

### Paso 2. Probar efecto con umask temporal

```bash
su - ana -c 'umask 027; touch /home/ana/fichero_027; mkdir /home/ana/carpeta_027; ls -ld /home/ana/fichero_027 /home/ana/carpeta_027'
```

### Paso 3. Configurar umask persistente para `ana`

```bash
echo 'umask 027' >> /home/ana/.bashrc
chown ana:ana /home/ana/.bashrc
su - ana -c 'umask; touch /home/ana/fichero_persistente; ls -l /home/ana/fichero_persistente'
```

### Paso 4. Comparacion rapida

```bash
su - ana -c 'umask 022; touch /home/ana/fichero_022; ls -l /home/ana/fichero_022'
su - ana -c 'umask 027; touch /home/ana/fichero_027b; ls -l /home/ana/fichero_027b'
```

Resultado esperado:

1. Con `022`, archivo suele nacer `644`.
1. Con `027`, archivo suele nacer `640`.

---

## Lab 4 (40 min) - Networking basico con validacion de puertos

Objetivo: validar conectividad y comprobar diferencia entre servicio escuchando y puerto permitido en firewall.

Tiempo estimado alumno: 40 min.

### Paso 1. Evidencia inicial de red

```bash
nmcli device status
ip -4 a
ip route
firewall-cmd --get-active-zones
firewall-cmd --list-all
```

### Paso 2. Levantar un puerto de prueba

```bash
# Si no tienes ncat: dnf install -y nmap-ncat
ncat -l 9090
```

Deja esta terminal abierta.

### Paso 3. Desde otro equipo o segunda VM, probar puerto

```bash
nmap -p 9090 <IP_DE_LA_VM>
# alternativa puntual
nc -zv <IP_DE_LA_VM> 9090
```

### Paso 4. Abrir puerto en firewall y repetir

```bash
firewall-cmd --permanent --add-port=9090/tcp
firewall-cmd --reload
firewall-cmd --list-ports
```

Repite `nmap` o `nc` desde cliente y compara resultado.

### Paso 5. Cerrar puerto y validar rollback

```bash
firewall-cmd --permanent --remove-port=9090/tcp
firewall-cmd --reload
firewall-cmd --list-ports
```

Checklist rapido:

1. El alumno diferencia `listening` local (`ss`/`ncat`) de accesible remoto.
1. Verifica impacto real de abrir/cerrar puerto en `firewalld`.
1. Documenta evidencia antes y despues.

---

## Entregable del alumno (evidencias minimas)

1. Captura/salida de `df -hT /tmp` con 20GB.
1. Captura/salida de `lvs` y `vgs` tras ampliar.
1. Evidencia de permisos de `proyecto` y `intercambio` (`ls -ld`).
1. Evidencia de `id ana` e `id bruno`.
1. Evidencia de umask y permisos de archivo creado con `022` vs `027`.
1. Evidencia de `nmap`/`nc` antes y despues de abrir puerto 9090.

## Errores frecuentes (guia de debugging)

1. Disco nuevo no aparece: falta rescan o disco no presentado en hipervisor.
1. VG extendido en grupo incorrecto: revisar `findmnt` y `lvs` antes de tocar LVM.
1. Cambiar permisos sin validar propietario/grupo: revisar `ls -l` y `id`.
1. Confundir `umask` temporal con persistente: revisar `.bashrc` y nueva sesion.
1. Abrir puerto en firewall pero no tener proceso escuchando: revisar `ss -tulpen`.
