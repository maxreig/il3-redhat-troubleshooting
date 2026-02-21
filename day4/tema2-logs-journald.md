# Tema 2: Logs y journald para troubleshooting

## Objetivo

Usar `journalctl` y rutas clave de `/var/log` para reconstruir incidentes por tiempo, servicio y severidad, con evidencia clara.

## Resumen para pizarra

1. Si no hay timeline, no hay diagnostico solido.
1. `journalctl` acelera analisis por unidad, prioridad y ventana temporal.
1. `/var/log` sigue siendo clave para componentes especificos.
1. Buscar primero error real, luego sintomas secundarios.
1. Documentar comando, timestamp y evento encontrado.

## Referencias visuales oficiales (Red Hat)

Estas capturas vienen de la documentacion oficial de Red Hat sobre Web Console (Cockpit).  
Aunque el estilo visual cambie entre versiones, el flujo operativo de diagnostico sigue siendo equivalente en RHEL modernos.

Login en Cockpit:

![Red Hat Cockpit login (oficial)](images/real/tema2-redhat-cockpit-login-official.png)

Vista general (dashboard):

![Red Hat Cockpit dashboard (oficial)](images/real/tema2-redhat-cockpit-dashboard-official.png)

Alta de servicio en firewall (para pruebas de conectividad/logs):

![Red Hat Cockpit add service (oficial)](images/real/tema2-redhat-cockpit-add-service-official.png)

Fuente oficial:  
[Red Hat Documentation - Managing systems using the RHEL 7 web console](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html-single/managing_systems_using_the_rhel_7_web_console/index)

## 1) journald: consulta operativa

Comandos base:

```bash
journalctl -xe
journalctl -p err -b
journalctl --since "30 min ago"
journalctl --since "2026-02-19 08:00:00" --until "2026-02-19 10:00:00"
```

Filtros utiles:

```bash
# Por servicio systemd
journalctl -u sshd
journalctl -u httpd --since "1 hour ago"

# Solo arranque actual / arranque previo
journalctl -b
journalctl -b -1

# Seguir en tiempo real
journalctl -f

# Prioridad: emerg..debug
journalctl -p warning -b
```

Buenas practicas:

1. Arranca con rango temporal corto y amplia si hace falta.
1. Filtra por unidad (`-u`) antes de leer miles de lineas.
1. Si hay reboot, compara `-b` y `-b -1`.

## 2) Persistencia de journald

Comprobacion:

```bash
ls -ld /var/log/journal
grep -E '^Storage=' /etc/systemd/journald.conf
```

Nota operativa:

1. Sin persistencia, parte de logs puede quedar solo en memoria.
1. En servidores productivos conviene persistencia para postmortem.

## 3) Logs clasicos en /var/log

Rutas frecuentes en RHEL/Rocky:

1. `/var/log/messages` -> eventos generales (segun configuracion de rsyslog).
1. `/var/log/secure` -> autenticacion y acceso.
1. `/var/log/cron` -> tareas programadas.
1. `/var/log/dmesg` -> mensajes de kernel al arranque.
1. `/var/log/audit/audit.log` -> eventos de auditoria/SELinux.

Comandos base:

```bash
tail -n 100 /var/log/messages
tail -n 100 /var/log/secure
grep -iE "error|fail|denied|timeout" /var/log/messages | tail -n 50
```

## 4) Metodo rapido de analisis de logs

1. Define ventana de tiempo exacta del incidente.
1. Filtra por servicio afectado.
1. Extrae 3-5 eventos relevantes (causa, impacto, recuperacion).
1. Corrobora con estado de servicio y recursos del host.
1. Guarda evidencia en ticket o informe tecnico.

## 5) Casos tipicos

### Servicio no arranca

```bash
systemctl status httpd
journalctl -u httpd -b -n 80 --no-pager
```

### Fallo de login por SSH

```bash
journalctl -u sshd --since "20 min ago"
tail -n 100 /var/log/secure
```

### Reinicio inesperado

```bash
journalctl -b -1 -p err --no-pager
last -x | head
```

## Errores frecuentes

1. Leer logs sin acotar tiempo.
1. Revisar solo `systemctl status` sin `journalctl`.
1. Quedarse con el ultimo error y no revisar contexto previo.
1. No distinguir warning de error critico.
1. No guardar evidencia del hallazgo.
