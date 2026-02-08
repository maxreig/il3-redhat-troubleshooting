# Día 2 – Labs (software, almacenamiento y servicios)

Curso **Red Hat Enterprise Linux 8 – Diagnostics & Troubleshooting**  
Material de apoyo – Día 2

> Objetivo general del día: practicar un flujo realista de administración y troubleshooting con **paquetes/repos**, **almacenamiento (LVM + montajes persistentes)** y **servicios (systemd)**.

---

## ✅ Lab principal – “Filesystems persistentes con LVM + verificación de /etc/fstab”

### Objetivo
1. Presentar un **nuevo disco** a la VM (VirtualBox).
2. Detectar el disco desde Linux y validar que el sistema lo ve correctamente.
3. Crear una estructura **LVM** completa:
   - **PV** sobre el disco (o partición)
   - **VG** dedicado
   - **3 a 5 LV** (según tiempo)
4. Crear **filesystems** en cada LV.
5. Montar los FS y dejar los montajes **persistentes** en `/etc/fstab` usando **UUID**.
6. Validar con `mount -a` y (opcional) con reinicio.

---

### Paso 0 – Preparación (si aplica)
- Entra con un usuario con `sudo` o como `root`.
- (Recomendado) Crea un **snapshot** antes de tocar discos y fstab.

Sanity check:

```bash
cat /etc/redhat-release
hostnamectl
ip a
df -h
lsblk
```

Asegúrate de tener herramientas instaladas (sobre todo en instalación minimal):

```bash
sudo dnf -y install lvm2 xfsprogs util-linux
```

---

### Paso 1 – Presentar un disco nuevo en VirtualBox
En VirtualBox:
- Apaga la VM (si hace falta).
- Añade un **nuevo disco** (por ejemplo 20 GB).
- Arranca la VM.

---

### Paso 2 – Detectar el nuevo disco en Linux

```bash
lsblk
sudo fdisk -l
dmesg | tail -n 80
```

📌 Identifica el disco nuevo (ejemplo típico: `/dev/sdb`).  
⚠️ **Ojo**: no uses el disco del sistema (`/dev/sda`) por error.

---

### Paso 3 – (Opcional) Crear partición LVM
> Para simplificar el lab puedes usar **todo el disco como PV** y saltarte este paso.

Si quieres particionar (recomendado en entornos “más reales”):

```bash
sudo fdisk /dev/sdb
```

Dentro de `fdisk` (guía rápida):
- `g` → tabla GPT (opcional)
- `n` → nueva partición (usa todo el disco)
- `t` → tipo (si te lo pide, selecciona “Linux LVM” si está disponible)
- `w` → escribir y salir

Luego:

```bash
lsblk
```

Ejemplo esperado: `/dev/sdb1`

---

### Paso 4 – Crear LVM (PV → VG → LV)

#### 4.1 Crear el PV
Si usas disco entero:

```bash
sudo pvcreate /dev/sdb
```

Si has creado partición:

```bash
sudo pvcreate /dev/sdb1
```

#### 4.2 Crear el VG dedicado

```bash
sudo vgcreate vg_lab /dev/sdb
# o:
# sudo vgcreate vg_lab /dev/sdb1
```

Validación:

```bash
pvs
vgs
```

#### 4.3 Crear 3 a 5 LVs

Ejemplo con 3 LVs (rápido):

```bash
sudo lvcreate -n lv_data1 -L 3G vg_lab
sudo lvcreate -n lv_data2 -L 3G vg_lab
sudo lvcreate -n lv_data3 -L 3G vg_lab
```

Si quieres 5 LVs (más completo):

```bash
sudo lvcreate -n lv_data4 -L 2G vg_lab
sudo lvcreate -n lv_data5 -L 2G vg_lab
```

Validación:

```bash
lvs
```

---

### Paso 5 – Crear filesystems

Recomendación (por coherencia con RHEL): **XFS**.

```bash
sudo mkfs.xfs /dev/vg_lab/lv_data1
sudo mkfs.xfs /dev/vg_lab/lv_data2
sudo mkfs.xfs /dev/vg_lab/lv_data3
# opcional:
sudo mkfs.xfs /dev/vg_lab/lv_data4
sudo mkfs.xfs /dev/vg_lab/lv_data5
```

Validación:

```bash
lsblk -f | grep vg_lab
blkid | grep vg_lab
```

---

### Paso 6 – Crear puntos de montaje y montar

Crea una estructura simple:

```bash
sudo mkdir -p /data/data1 /data/data2 /data/data3
# opcional:
sudo mkdir -p /data/data4 /data/data5
```

Monta:

```bash
sudo mount /dev/vg_lab/lv_data1 /data/data1
sudo mount /dev/vg_lab/lv_data2 /data/data2
sudo mount /dev/vg_lab/lv_data3 /data/data3
# opcional:
sudo mount /dev/vg_lab/lv_data4 /data/data4
sudo mount /dev/vg_lab/lv_data5 /data/data5
```

Validación:

```bash
df -hT | grep -E "/data/data"
findmnt | grep -E "/data/data"
```

---

### Paso 7 – Persistencia en /etc/fstab (con UUID)

#### 7.1 Backup antes de tocar

```bash
sudo cp -a /etc/fstab /etc/fstab.bak
```

#### 7.2 Obtener UUID

```bash
lsblk -f | grep vg_lab
blkid | grep vg_lab
```

#### 7.3 Editar fstab

```bash
sudo vim /etc/fstab
```

Ejemplo (ajusta UUIDs reales y tipo `xfs`):

```text
UUID=XXXX-XXXX  /data/data1  xfs  defaults  0  0
UUID=YYYY-YYYY  /data/data2  xfs  defaults  0  0
UUID=ZZZZ-ZZZZ  /data/data3  xfs  defaults  0  0
# UUID=AAAA-AAAA  /data/data4  xfs  defaults  0  0
# UUID=BBBB-BBBB  /data/data5  xfs  defaults  0  0
```

> Consejo: usa UUID (más robusto que `/dev/sdX`).

---

### Paso 8 – Validación SIN reiniciar (la clave del lab)

Simula “post-reboot”:

```bash
sudo umount /data/data1 /data/data2 /data/data3
# opcional:
sudo umount /data/data4 /data/data5 2>/dev/null || true
```

Monta todo lo del fstab:

```bash
sudo mount -a
```

Comprueba:

```bash
findmnt | grep -E "/data/data"
df -hT | grep -E "/data/data"
```

📌 Si `mount -a` falla:
- revisa que existan los directorios (`/data/dataX`)
- revisa UUID (copiado mal)
- revisa tipo FS (`xfs` vs `ext4`)
- revisa que el dispositivo exista (`lsblk -f`)

---

### Paso 9 – (Opcional) Validación con reinicio

```bash
sudo reboot
```

Tras reiniciar:

```bash
findmnt | grep -E "/data/data"
df -hT | grep -E "/data/data"
```

---

## 🧪 Extra A – Gestión de software y repos (mini-taller)

### Objetivo
Confirmar que sabes **listar repos**, **limpiar cache**, **instalar** y **auditar** cambios (history).

```bash
sudo dnf repolist
sudo dnf repolist --all
sudo dnf clean all
sudo dnf makecache
sudo dnf -y install tree
tree --version || true
sudo dnf history | head
```

> Si estás en RHEL y no hay repos: revisa suscripción (`subscription-manager status`).

---

## 🧪 Extra B – systemctl: servicio común + logs

### Objetivo
Practicar el ciclo de vida de un servicio real y su troubleshooting básico.

Ejemplo con `chronyd` (tiempo) o `sshd` (acceso remoto):

```bash
sudo systemctl status chronyd || true
sudo systemctl enable --now chronyd
systemctl is-enabled chronyd
sudo systemctl restart chronyd
journalctl -u chronyd -n 50 --no-pager
```

Si prefieres SSH:

```bash
sudo systemctl status sshd
sudo systemctl enable --now sshd
journalctl -u sshd -n 50 --no-pager
```

---

## ✅ Cierre del día (checklist final)

Antes de terminar, confirma:
- [ ] Disco nuevo presentado y detectado (`lsblk`, `fdisk -l`)
- [ ] PV creado correctamente (`pvs`)
- [ ] VG creado (`vgs`)
- [ ] 3–5 LVs creados (`lvs`)
- [ ] Filesystems creados y visibles (`lsblk -f`, `blkid`)
- [ ] Montajes OK (`findmnt`, `df -hT`)
- [ ] `/etc/fstab` actualizado con **UUID** y verificado con `mount -a`
- [ ] (Opcional) Tras reinicio, sigue todo montado
- [ ] (Recomendado) Snapshot guardado

---

## 📌 Tarea opcional (entre sesiones)
- Leer: `man dnf`, `man lvm`, `man fstab`
- Practicar: romper algo controlado y arreglarlo:
  - cambia un UUID en fstab a propósito y recupera desde `/etc/fstab.bak`
