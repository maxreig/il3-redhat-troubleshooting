# Labs Dia 4 - Escenario integrado de troubleshooting guiado

## Enfoque del dia

El objetivo no es "adivinar", sino aplicar metodo:

1. observar sintomas;
1. recoger evidencia;
1. aislar causa;
1. corregir;
1. validar.

## Reglas de trabajo

1. No ejecutar cambios sin evidencia previa.
1. Documentar cada comando clave y su salida relevante.
1. Aplicar cambios minimos, uno por vez.
1. Verificar resultado antes de cerrar.

---

## Requisitos previos

1. VM RHEL/Rocky operativa con acceso root o `sudo`.
1. Paquetes recomendados instalados: `sysstat`, `nmap-ncat`.
1. Snapshot recomendado antes de empezar.

---

## Lab final (120 min) - Escenario integrado

### Contexto

Se reporta que "la VM esta lenta, un servicio falla de forma intermitente y algunos usuarios no pueden acceder a recursos".  
El alumno debe diagnosticar por capas y dejar evidencia tecnica.

---

## Fase 1 (20 min) - Foto inicial del sistema

```bash
date
hostnamectl
uptime
top -b -n 1 | head -n 25
free -h
df -hT
df -i
iostat -xz 1 3
```

Evidencia esperada:

1. Estado de CPU, RAM, capacidad e I/O.
1. Hipotesis inicial de riesgo principal (recurso/servicio/red/acceso).

---

## Fase 2 (25 min) - Servicio degradado o caido

1. Selecciona un servicio de prueba (`httpd`, `sshd` o similar).
1. Fuerza fallo controlado (ejemplo: parametro invalido en config de test, o parar servicio).
1. Diagnostica y recupera.

Comandos sugeridos:

```bash
systemctl status <servicio>
journalctl -u <servicio> -b -n 100 --no-pager
systemctl restart <servicio>
systemctl is-active <servicio>
```

Checklist:

1. Detecta causa concreta del fallo.
1. Corrige configuracion o dependencia.
1. Deja servicio en estado `active`.

---

## Fase 3 (25 min) - Incidencia de conectividad

1. Simula fallo puntual de conectividad (DNS o firewall).
1. Demuestra diferencia entre "sin IP", "sin DNS" y "puerto cerrado".
1. Restaura estado operativo.

Comandos sugeridos:

```bash
ip -4 a
ip route
ping -c 3 <gateway>
ping -c 3 8.8.8.8
getent hosts redhat.com
ss -tulpen
firewall-cmd --list-all
ncat -l -k 9090
```

Desde cliente/segunda VM:

```bash
nmap -p 9090 <IP_DE_LA_VM>
ncat <IP_DE_LA_VM> 9090
```

Checklist:

1. Identifica si el bloqueo era red, DNS o firewall.
1. Recupera conectividad esperada.
1. Aporta prueba remota antes y despues.

---

## Fase 4 (25 min) - Espacio o inodos

1. Genera ocupacion controlada en ruta de pruebas (`/tmp/lab4`).
1. Detecta crecimiento y recupera espacio.
1. Verifica estado final.

Comandos sugeridos:

```bash
mkdir -p /tmp/lab4
dd if=/dev/zero of=/tmp/lab4/relleno.bin bs=1M count=300
df -hT /tmp
du -xh /tmp/lab4
rm -f /tmp/lab4/relleno.bin
df -hT /tmp
```

Checklist:

1. Detecta ruta que mas crece.
1. Aplica limpieza controlada.
1. Confirma recuperacion de capacidad.

---

## Fase 5 (25 min) - Accesos y permisos

1. Crea un recurso compartido con permisos incorrectos.
1. Valida fallo de acceso con usuario de pruebas.
1. Corrige propietario/grupo/permisos y verifica.

Comandos sugeridos:

```bash
mkdir -p /lab4/compartido
touch /lab4/compartido/dato.txt
chmod 700 /lab4/compartido
ls -ld /lab4/compartido
id <usuario_prueba>
namei -l /lab4/compartido/dato.txt
```

Checklist:

1. Explica por que fallaba el acceso.
1. Corrige con permisos minimos necesarios.
1. Valida acceso final del usuario objetivo.

---

## Entregable final del alumno

Evidencias minimas:

1. Snapshot de monitorizacion (`uptime`, `free -h`, `df -hT`, `iostat`).
1. Diagnostico de un servicio (error encontrado + correccion + estado final).
1. Prueba de conectividad antes/despues (IP, DNS y puerto).
1. Evidencia de gestion de capacidad (espacio o inodos).
1. Evidencia de correccion de permisos/acceso.
1. Resumen final: causa raiz de cada incidencia y accion aplicada.

## Errores frecuentes (guia de debugging)

1. Reiniciar todo sin aislar causa.
1. Corregir sin evidencia previa.
1. Cambiar varias variables a la vez.
1. Olvidar validar desde cliente en problemas de red.
1. Cerrar incidente sin test final de servicio.
