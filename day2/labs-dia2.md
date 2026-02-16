# Día 2 – Labs (almacenamiento, swap y servicios)

Curso **Red Hat Enterprise Linux 8/9 – Diagnostics & Troubleshooting**  
Material práctico – Día 2

Objetivo del bloque práctico: ejecutar 3 laboratorios reales de operación en RHEL.

---

## Requisitos previos

- Acceso con usuario con permisos `sudo`.
- VM en VirtualBox.
- Snapshot recomendado antes de empezar.
- Paquetes base instalados:

```bash
sudo dnf -y install lvm2 xfsprogs util-linux
```

Verificación rápida inicial:

```bash
cat /etc/redhat-release
hostnamectl
lsblk
df -hT
free -h
```

---

## Lab 0 – Gestión mínima de software con `dnf`

**Tiempo estimado alumno:** 20–30 minutos.

### Objetivo
Practicar lo mínimo necesario de gestión de software: comprobar si un paquete está instalado, buscar paquetes en repositorios e instalar uno nuevo.

### Paso 1 – Comprobar repositorios y caché

Ver repos habilitados:

```bash
sudo dnf repolist
```

Refrescar metadatos:

```bash
sudo dnf makecache
```

### Paso 2 – Verificar si un paquete está instalado

Ejemplo con `tree`:

```bash
rpm -q tree
```

Si no está instalado, el comando indicará que no existe el paquete instalado.

### Paso 3 – Buscar paquetes en repositorios

Buscar por nombre:

```bash
dnf search tree
```

Ver información del paquete:

```bash
dnf info tree
```

### Paso 4 – Instalar paquete

```bash
sudo dnf -y install tree
```

Validar instalación:

```bash
rpm -q tree
tree --version
```

### Paso 5 – Consultar historial básico de `dnf`

```bash
sudo dnf history | head
```

---

## Lab 1 – Filesystems persistentes con LVM

**Tiempo estimado alumno:** 50–70 minutos.

### Objetivo
Crear LVM sobre un disco nuevo, montar varios filesystems y dejarlos persistentes en `/etc/fstab`.

### Paso 1 – Presentar disco en VirtualBox

- Añade un disco nuevo (ejemplo: 20 GB) a la VM.
- Inicia la VM.

### Paso 2 – Detectar el disco en Linux

```bash
lsblk
sudo fdisk -l
dmesg | tail -n 80
```

Identifica el disco nuevo (ejemplo: `/dev/sdb`). No uses el disco del sistema por error.

### Paso 3 – Crear estructura LVM (PV -> VG -> LV)

Crear PV (usando el disco completo):

```bash
sudo pvcreate /dev/sdb
```

Crear VG:

```bash
sudo vgcreate vg_lab /dev/sdb
```

Crear LVs de ejemplo:

```bash
sudo lvcreate -n lv_data1 -L 3G vg_lab
sudo lvcreate -n lv_data2 -L 3G vg_lab
sudo lvcreate -n lv_data3 -L 3G vg_lab
```

Validar:

```bash
pvs
vgs
lvs
```

### Paso 4 – Crear filesystems y montar

Formatear en XFS:

```bash
sudo mkfs.xfs /dev/vg_lab/lv_data1
sudo mkfs.xfs /dev/vg_lab/lv_data2
sudo mkfs.xfs /dev/vg_lab/lv_data3
```

Crear puntos de montaje:

```bash
sudo mkdir -p /data/data1 /data/data2 /data/data3
```

Montar:

```bash
sudo mount /dev/vg_lab/lv_data1 /data/data1
sudo mount /dev/vg_lab/lv_data2 /data/data2
sudo mount /dev/vg_lab/lv_data3 /data/data3
```

Validar:

```bash
findmnt | grep -E '/data/data'
df -hT | grep -E '/data/data'
```

### Paso 5 – Persistencia en `/etc/fstab`

Backup de seguridad:

```bash
sudo cp -a /etc/fstab /etc/fstab.bak
```

Obtener UUID:

```bash
sudo blkid /dev/vg_lab/lv_data1 /dev/vg_lab/lv_data2 /dev/vg_lab/lv_data3
```

Editar `/etc/fstab`:

```bash
sudo vi /etc/fstab
```

Añadir líneas (sustituyendo UUID reales):

```text
UUID=<UUID_LV_DATA1>  /data/data1  xfs  defaults  0  0
UUID=<UUID_LV_DATA2>  /data/data2  xfs  defaults  0  0
UUID=<UUID_LV_DATA3>  /data/data3  xfs  defaults  0  0
```

Validar sin reiniciar:

```bash
sudo umount /data/data1 /data/data2 /data/data3
sudo mount -a
findmnt | grep -E '/data/data'
```

---

## Lab 2 – Ampliar SWAP con un disco nuevo (VirtualBox)

**Tiempo estimado alumno:** 30–45 minutos.

### Objetivo
Añadir un nuevo disco y ampliar la swap del sistema de forma persistente.

### Paso 1 – Presentar segundo disco

- Añade otro disco a la VM (ejemplo: 4 GB).
- Inicia la VM.

Detectar disco (ejemplo: `/dev/sdc`):

```bash
lsblk
sudo fdisk -l
```

### Paso 2 – Crear swap en el nuevo disco

Inicializar área swap:

```bash
sudo mkswap /dev/sdc
```

Activar swap:

```bash
sudo swapon /dev/sdc
```

Validar estado:

```bash
swapon --show
free -h
```

### Paso 3 – Hacer persistente en `/etc/fstab`

Backup de seguridad (si no existe ya):

```bash
sudo cp -a /etc/fstab /etc/fstab.bak.swap
```

Obtener UUID del dispositivo swap:

```bash
sudo blkid /dev/sdc
```

Editar `/etc/fstab`:

```bash
sudo vi /etc/fstab
```

Añadir línea (sustituye UUID real):

```text
UUID=<UUID_SWAP_NUEVA>  none  swap  defaults  0  0
```

Probar configuración:

```bash
sudo swapoff /dev/sdc
sudo swapon -a
swapon --show
```

---

## Lab 3 – Crear un servicio `systemd` desde un script simple

**Tiempo estimado alumno:** 25–35 minutos.

### Objetivo
Crear un script básico de verificación de filesystems y ejecutarlo como servicio `systemd`.

### Paso 1 – Crear script de chequeo

```bash
sudo tee /usr/local/bin/check-fs.sh > /dev/null <<'SCRIPT'
#!/usr/bin/env bash
set -euo pipefail
DATE="$(date '+%F %T')"
echo "[$DATE] Estado de filesystems:"
df -hT | sed -n '1,12p'
SCRIPT
```

Dar permisos:

```bash
sudo chmod +x /usr/local/bin/check-fs.sh
```

Prueba manual:

```bash
/usr/local/bin/check-fs.sh
```

### Paso 2 – Crear unidad `systemd`

```bash
sudo tee /etc/systemd/system/check-fs.service > /dev/null <<'UNIT'
[Unit]
Description=Chequeo simple de filesystems
After=local-fs.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/check-fs.sh
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
UNIT
```

Recargar unidades:

```bash
sudo systemctl daemon-reload
```

### Paso 3 – Ejecutar y validar el servicio

Lanzar servicio:

```bash
sudo systemctl start check-fs.service
```

Revisar estado:

```bash
sudo systemctl status check-fs.service --no-pager
```

Revisar logs:

```bash
journalctl -u check-fs.service -n 30 --no-pager
```

Habilitar para que quede disponible en arranque:

```bash
sudo systemctl enable check-fs.service
systemctl is-enabled check-fs.service
```

---

## Checklist final del Día 2

- [ ] Lab 1 completado: LVM creado, montado y persistente en `/etc/fstab`.
- [ ] Lab 2 completado: swap nueva activa y persistente.
- [ ] Lab 3 completado: servicio `check-fs.service` ejecuta y registra en journal.
- [ ] Validaciones ejecutadas con `lsblk`, `findmnt`, `swapon --show`, `systemctl status`, `journalctl`.
