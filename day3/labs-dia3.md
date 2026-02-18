# Labs Dia 3 - Disco + servicio + firewall + diagnostico de puertos

## Escenario del laboratorio

Objetivo: dejar un servicio web funcionando en la VM con:

1. disco nuevo preparado y montado de forma persistente;
1. servicio `httpd` habilitado y activo;
1. puerto abierto en `firewalld`;
1. validaciones de red y seguridad basica.

Tiempo estimado alumno: 75-105 minutos.

---

## Requisitos previos

1. VM RHEL/Rocky operativa con acceso root o `sudo`.
1. Un disco adicional presentado desde el hipervisor (ejemplo: `/dev/sdb`).
1. Paquetes base disponibles por repositorio.

---

## Lab 1 - Anadir y preparar un disco

### Paso 1. Detectar el nuevo disco

```bash
lsblk -f
```

Resultado esperado: aparece un disco sin uso (ejemplo `/dev/sdb`).

### Paso 2. Crear particion y filesystem

```bash
parted /dev/sdb --script mklabel gpt mkpart primary xfs 1MiB 100%
mkfs.xfs /dev/sdb1
```

### Paso 3. Montar en `/datos` y hacer persistente

```bash
mkdir -p /datos
mount /dev/sdb1 /datos
blkid /dev/sdb1
```

Editar `/etc/fstab` y agregar una linea con UUID:

```text
UUID=<uuid-real> /datos xfs defaults 0 0
```

Validar:

```bash
mount -a
df -hT /datos
```

Checklist rapido:

1. `/datos` montado correctamente.
1. `mount -a` no devuelve errores.
1. El tipo de filesystem es `xfs`.

---

## Lab 2 - Habilitar un servicio (Apache)

### Paso 1. Instalar y arrancar servicio

```bash
dnf install -y httpd
systemctl enable --now httpd
systemctl status httpd
```

### Paso 2. Publicar pagina de prueba en el disco nuevo

```bash
cat > /datos/index.html <<'EOF'
<html><body><h1>Lab Dia 3 OK</h1></body></html>
EOF
cp /datos/index.html /var/www/html/index.html
```

### Paso 3. Validar localmente

```bash
ss -tulpen | grep :80
curl -I http://127.0.0.1
curl http://127.0.0.1
```

Checklist rapido:

1. `httpd` en estado `active (running)`.
1. Puerto `80/tcp` en escucha.
1. Respuesta HTTP local correcta.

---

## Lab 3 - Abrir puerto en firewalld

### Paso 1. Revisar estado y zona activa

```bash
systemctl enable --now firewalld
firewall-cmd --get-active-zones
firewall-cmd --list-all
```

### Paso 2. Abrir servicio HTTP de forma permanente

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-all
```

### Paso 3. Probar conectividad desde otra maquina

```bash
curl -I http://<IP_DE_LA_VM>
```

Checklist rapido:

1. Zona correcta con servicio `http` habilitado.
1. Acceso remoto a puerto 80 validado.

---

## Lab 4 - Nocion SELinux (comprobacion basica)

### Paso 1. Revisar estado y contexto

```bash
getenforce
ls -lZ /var/www/html/index.html
```

### Paso 2. Restaurar contexto por seguridad

```bash
restorecon -Rv /var/www/html
```

### Paso 3. Si usas puerto alternativo (opcional 8080)

```bash
semanage port -l | grep http_port_t
# Si no aparece 8080 y quieres usarlo:
semanage port -a -t http_port_t -p tcp 8080
```

---

## Lab 5 - Diagnostico de puertos con nmap y ncat (TCP/UDP)

### Paso 1. Instalar herramientas

```bash
dnf install -y nmap nmap-ncat
```

### Paso 2. Escanear puertos TCP del servidor

```bash
# Sustituir por la IP de tu VM objetivo
nmap -p 22,80,443,8080 <IP_DE_LA_VM>
```

Resultado esperado: `22/tcp` y `80/tcp` en estado coherente con la configuracion real.

### Paso 3. Probar puerto TCP con ncat (2 terminales)

En terminal A (servidor):

```bash
ncat -l 9090
```

En terminal B (cliente):

```bash
ncat <IP_DE_LA_VM> 9090
```

Escribe texto en el cliente y valida recepcion en el servidor.

### Paso 4. Probar puerto UDP con ncat

En terminal A (servidor UDP):

```bash
ncat -u -l 9999
```

En terminal B (cliente UDP):

```bash
echo "test-udp" | ncat -u <IP_DE_LA_VM> 9999
```

Validar que el texto llega al servidor UDP.

### Paso 5. Correlacionar con firewall

```bash
firewall-cmd --list-all
```

Si el puerto esta bloqueado, repetir prueba tras abrirlo y documentar diferencia.

Checklist rapido:

1. `nmap` refleja puertos abiertos/cerrados de forma coherente.
1. `ncat` TCP confirma conectividad extremo a extremo.
1. `ncat` UDP validado (con nota de que UDP no confirma sesion como TCP).
1. El alumno identifica si el bloqueo es de firewall o de servicio no escuchando.

---

## Entregable final del alumno

Evidencias minimas:

1. salida de `lsblk -f` y `df -hT /datos`;
1. salida de `systemctl status httpd` (resumen estado activo);
1. salida de `firewall-cmd --list-all` con `http`;
1. resultado de `curl` remoto contra la IP de la VM;
1. salida de `nmap -p 22,80,443,8080 <IP_DE_LA_VM>`;
1. evidencia de prueba `ncat` en TCP o UDP.

## Troubleshooting guiado (errores comunes)

1. Servicio activo pero no accesible:
1. revisar `firewalld` y zona.
1. No monta despues de reinicio:
1. revisar UUID en `/etc/fstab`.
1. Firewall correcto pero sigue fallando:
1. revisar SELinux (`getenforce`, contextos, puertos).
1. `nmap` marca `closed`:
1. revisar si el proceso escucha en ese puerto (`ss -tulpen`).
1. `nmap` marca `filtered`:
1. revisar reglas y zona activa de `firewalld`.
