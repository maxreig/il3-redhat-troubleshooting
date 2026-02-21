# Tema 3: Diagnostico de problemas comunes

## Objetivo

Aplicar un metodo repetible para resolver incidencias frecuentes en Linux: conectividad, servicios que no arrancan, falta de espacio y accesos.

## Marco operativo

Flujo recomendado en cualquier incidente:

1. Confirmar sintoma con evidencia minima.
1. Delimitar alcance (un host, un servicio, toda la red).
1. Validar capas basicas (recurso, proceso, red, permisos).
1. Corregir con cambio minimo y reversible.
1. Verificar resultado y documentar.

## Referencias oficiales Red Hat (capturas reales)

Estas capturas son de documentacion oficial de Red Hat Web Console (Cockpit).  
Visualmente son de RHEL 7, pero el enfoque de troubleshooting aplica igual para RHEL 8/9.

Vista general del host (punto de partida):

![Red Hat Cockpit dashboard (oficial)](images/real/tema3-redhat-cockpit-dashboard-official.png)

Diagnostico de conectividad por interfaz:

![Red Hat Cockpit network interfaces (oficial)](images/real/tema3-redhat-cockpit-network-interfaces-official.png)

Validacion de zonas activas de firewall:

![Red Hat Cockpit firewall zones active (oficial)](images/real/tema3-redhat-cockpit-fw-zones-active-official.png)

Documentacion oficial recomendada (RHEL 9):

1. [Configuring and Managing Networking](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/index)
1. [Using systemd unit files](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_systemd_unit_files_to_customize_and_optimize_your_system/index)
1. [Security hardening](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/security_hardening/index)
1. [Managing systems using the RHEL web console (capturas)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html-single/managing_systems_using_the_rhel_7_web_console/index)

## 1) Conectividad

Checklist corto:

```bash
ip -4 a
ip route
ping -c 3 <gateway>
ping -c 3 8.8.8.8
getent hosts redhat.com
ss -tulpen
```

Interpretacion rapida:

1. Falla gateway -> problema local de red/interfaz/ruta.
1. Funciona IP pero falla nombre -> problema DNS.
1. Puerto cerrado/filtered -> firewall o servicio no escuchando.

## 2) Servicios que no arrancan

Checklist corto:

```bash
systemctl status <servicio>
journalctl -u <servicio> -b -n 100 --no-pager
systemctl cat <servicio>
```

Causas comunes:

1. Configuracion invalida.
1. Dependencia no disponible.
1. Puerto ocupado por otro proceso.
1. Permisos/contexto incorrecto en rutas de trabajo.

Validaciones utiles:

```bash
ss -tulpen | grep -E ':80|:443|:<puerto>'
ls -ld <ruta>
```

## 3) Falta de espacio

Checklist corto:

```bash
df -hT
df -i
du -xh /var | sort -h | tail -n 20
find /var/log -type f -size +200M -ls | head
```

Accion segura:

1. Identificar que crecio y por que.
1. Rotar/limpiar de forma controlada.
1. Validar recuperacion de servicio.
1. Si aplica, ampliar volumen con LVM.

## 4) Accesos y permisos

Checklist corto:

```bash
id <usuario>
namei -l <ruta>
ls -ld <ruta>
ls -l <archivo>
```

Causas comunes:

1. Usuario fuera del grupo esperado.
1. Permisos insuficientes en un directorio padre.
1. `umask` no alineada al caso de uso.
1. Control adicional (ACL/SELinux) bloqueando.

Validacion SELinux (si aplica):

```bash
getenforce
ls -lZ <ruta>
ausearch -m avc -ts recent
```

## Runbook minimo por incidente

1. Sintoma: que falla y desde cuando.
1. Evidencia inicial: 3-5 comandos base.
1. Hipotesis principal: causa mas probable.
1. Accion aplicada: comando/cambio exacto.
1. Verificacion: prueba antes/despues.
1. Leccion aprendida: como prevenir recurrencia.

## Errores frecuentes

1. Saltar directo a reiniciar sin diagnostico.
1. Cambiar varias cosas a la vez (sin aislar causa).
1. No medir antes/despues del cambio.
1. Ignorar logs y basarse solo en intuicion.
1. Cerrar incidente sin evidencia final.
