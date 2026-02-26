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

---

## Lab extra (30-45 min) - Bash scripting operativo basico (Tema 4)

### Objetivo

Crear un script Bash sencillo para automatizar una comprobacion de espacio en disco con validaciones, codigos de salida y mensajes para operador.

### Contexto

El equipo de soporte necesita una herramienta rapida para comprobar uso de disco en una ruta y devolver estado `OK/WARNING` sin ejecutar comandos manuales cada vez.

### Tarea principal: `check_disk.sh`

Crear un script `check_disk.sh` que:

1. Reciba una ruta como parametro (por defecto `/`).
1. Reciba un umbral opcional (por defecto `80`).
1. Valide que la ruta existe.
1. Muestre uso de disco de la ruta (`df -h`).
1. Extraiga el porcentaje de uso real.
1. Devuelva `WARNING` y `exit 1` si supera el umbral.
1. Devuelva `OK` y `exit 0` si esta por debajo o igual al umbral.

### Requisitos tecnicos (minimos)

1. Debe incluir shebang `#!/usr/bin/env bash`.
1. Debe usar comillas en variables (`"$ruta"`).
1. Debe validar parametros basicos.
1. Debe funcionar con `bash -n` (sin errores de sintaxis).

### Pista de comandos utiles

```bash
df -P / | tail -n 1
df -P / | awk 'NR==2 {print $5}'
tr -d '%'
```

### Plantilla de prueba (ejecucion)

```bash
chmod +x check_disk.sh
bash -n check_disk.sh
./check_disk.sh
./check_disk.sh /tmp
./check_disk.sh / 1
echo $?
```

### Evidencia esperada

1. Captura/salida de `bash -n check_disk.sh`.
1. Ejecucion con ruta por defecto.
1. Ejecucion con umbral bajo (para forzar `WARNING`).
1. Codigo de salida mostrado con `echo $?`.

### Extension opcional (nivel +1)

Añadir un segundo script `check_service.sh` que:

1. Reciba nombre de servicio (`sshd` por defecto).
1. Use `systemctl is-active --quiet`.
1. Muestre `OK`/`CRITICAL`.
1. Devuelva `exit 0` o `exit 2`.

Comandos de prueba:

```bash
./check_service.sh
./check_service.sh sshd
./check_service.sh servicio_inexistente
```

### Checklist de correccion (docente)

1. El script ejecuta sin error de sintaxis.
1. El alumno usa parametros y valores por defecto.
1. El porcentaje se parsea correctamente (sin `%`).
1. El codigo de salida coincide con el resultado.
1. El mensaje final es claro para operacion (`OK` o `WARNING`).
