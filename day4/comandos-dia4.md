# Comandos Dia 4 - Monitorizacion y troubleshooting

## 0) Snapshot rapido del sistema

```bash
date
hostnamectl
uptime
cat /etc/redhat-release
```

## 1) Monitorizacion basica (CPU/RAM/disco)

```bash
# CPU y procesos
uptime
top
top -b -n 1 | head -n 25
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head

# Memoria
free -h
vmstat 1 5
cat /proc/meminfo | head -n 20

# Capacidad de disco e inodos
df -hT
df -i

# Top de uso en /var
du -xh /var | sort -h | tail -n 20
find /var/log -type f -size +100M -ls | head
```

## 2) I/O de disco (iostat)

```bash
# Instalar si falta
dnf install -y sysstat

# Metrica de I/O extendida
iostat -xz 1 5
```

## 3) Logs con journalctl

```bash
# Basico
journalctl -xe
journalctl -p err -b
journalctl --since "30 min ago"

# Por servicio
journalctl -u sshd -b -n 100 --no-pager
journalctl -u httpd --since "1 hour ago"

# Arranques
journalctl -b
journalctl -b -1

# Tiempo real
journalctl -f
```

## 4) Logs clasicos en /var/log

```bash
tail -n 100 /var/log/messages
tail -n 100 /var/log/secure
tail -n 100 /var/log/cron
grep -iE "error|fail|denied|timeout" /var/log/messages | tail -n 50
```

## 5) Diagnostico de conectividad

```bash
nmcli device status
ip -4 a
ip route
cat /etc/resolv.conf
ping -c 3 <gateway>
ping -c 3 8.8.8.8
getent hosts redhat.com
ss -tulpen
```

## 6) Servicios que no arrancan

```bash
systemctl status <servicio>
systemctl is-enabled <servicio>
journalctl -u <servicio> -b -n 120 --no-pager
systemctl cat <servicio>
```

## 7) Falta de espacio

```bash
df -hT
df -i
du -xh / | sort -h | tail -n 30
find / -xdev -type f -size +500M -ls | head
```

## 8) Accesos y permisos

```bash
id <usuario>
getent passwd <usuario>
getent group <grupo>
namei -l <ruta>
ls -ld <ruta>
ls -l <archivo>
```

## 9) SELinux (si aplica al incidente)

```bash
getenforce
sestatus
ls -lZ <ruta>
ausearch -m avc -ts recent
```

## 10) Evidencia final del troubleshooting

```bash
# Guardar resumen de estado tras correccion
date
systemctl status <servicio> --no-pager
journalctl -u <servicio> --since "10 min ago" --no-pager
df -hT
ss -tulpen
```
