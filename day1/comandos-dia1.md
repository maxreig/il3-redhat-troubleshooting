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

```bash
#### Pruebas rápidas de conectividad / DNS
ping -c 3 8.8.8.8
ping -c 3 google.com
curl -I https://www.google.com
```
