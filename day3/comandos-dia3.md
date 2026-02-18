# Comandos Dia 3 - Red y seguridad basica

## 1) NetworkManager y nmcli

```bash
# Estado general
nmcli general status
nmcli device status
nmcli connection show

# Detalle de un perfil
nmcli connection show "System eth0"

# Activar/desactivar perfil
nmcli connection up "System eth0"
nmcli connection down "System eth0"

# Configurar IP estatica
nmcli connection modify "System eth0" \
  ipv4.addresses "192.168.56.20/24" \
  ipv4.gateway "192.168.56.1" \
  ipv4.dns "1.1.1.1 8.8.8.8" \
  ipv4.method manual
nmcli connection up "System eth0"

# Volver a DHCP
nmcli connection modify "System eth0" ipv4.method auto
nmcli connection up "System eth0"
```

## 2) Diagnostico de conectividad

```bash
ip a
ip route
cat /etc/resolv.conf
ping -c 3 <gateway>
ping -c 3 8.8.8.8
ping -c 3 redhat.com
ss -tulpen
```

## 3) Escaneo y prueba de puertos (nmap / ncat)

```bash
# Instalar utilidades
dnf install -y nmap nmap-ncat

# Descubrir hosts en red local (sin escaneo de puertos)
nmap -sn 192.168.56.0/24

# Escaneo de puertos TCP comunes
nmap <IP_OBJETIVO>

# Escaneo de puertos concretos
nmap -p 22,80,443,8080 <IP_OBJETIVO>

# Escucha TCP de prueba
ncat -l 8080

# Cliente TCP de prueba
ncat <IP_OBJETIVO> 8080

# Escucha UDP de prueba
ncat -u -l 9999

# Cliente UDP de prueba
echo "test-udp" | ncat -u <IP_OBJETIVO> 9999
```

## 4) firewalld

```bash
# Estado y zona activa
systemctl status firewalld
firewall-cmd --state
firewall-cmd --get-active-zones
firewall-cmd --list-all

# Abrir servicio HTTP
firewall-cmd --add-service=http
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

# Abrir puerto custom
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# Cerrar reglas
firewall-cmd --permanent --remove-service=http
firewall-cmd --permanent --remove-port=8080/tcp
firewall-cmd --reload
```

## 5) SELinux (basico)

```bash
getenforce
sestatus
ls -lZ /var/www/html
ps -eZ | grep httpd

# Restaurar contexto recomendado
restorecon -Rv /var/www/html

# Ver puertos asociados a httpd
semanage port -l | grep http_port_t
```

## 6) Servicio (systemctl) para el lab

```bash
# Instalar y habilitar Apache
dnf install -y httpd
systemctl enable --now httpd
systemctl status httpd

# Validar escucha local
ss -tulpen | grep :80
curl -I http://127.0.0.1
```

## 7) Disco y montaje para el lab

```bash
lsblk -f
parted /dev/sdb --script mklabel gpt mkpart primary xfs 1MiB 100%
mkfs.xfs /dev/sdb1
mkdir -p /datos
mount /dev/sdb1 /datos
blkid /dev/sdb1

# Persistencia en /etc/fstab (usar UUID real)
# UUID=xxxx /datos xfs defaults 0 0
mount -a
df -hT /datos
```
