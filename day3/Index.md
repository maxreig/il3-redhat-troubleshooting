# Dia 3 - Red, seguridad basica y lab integrado

## Indice del Dia 3 (material asociado)

- [`tema1-networkmanager-nmcli.md`](tema1-networkmanager-nmcli.md) -> configuracion y diagnostico de red con NetworkManager (`nmcli`) y alternativas segun entorno.
- [`tema2-seguridad-firewalld-selinux.md`](tema2-seguridad-firewalld-selinux.md) -> base de `firewalld` y nociones practicas de SELinux (modo basico).
- [`comandos-dia3.md`](comandos-dia3.md) -> chuleta de comandos clave del dia 3.
- [`labs-dia3.md`](labs-dia3.md) -> laboratorio guiado: disco nuevo, servicio habilitado y puerto abierto en firewall.
- Lab 5 recomendado dentro de `labs-dia3.md` -> diagnostico de puertos con `nmap` y `ncat` (TCP/UDP).

---

## Resumen ejecutivo del dia

1. Red: entender perfiles y estado de interfaces con `nmcli`.
1. Diagnostico de puertos: validar servicio vs firewall con `nmap`/`ncat`.
1. Seguridad de red: abrir solo lo necesario con `firewalld` (runtime y permanente).
1. Seguridad del sistema: reconocer estados y contexto SELinux para no romper servicios.
1. Practica integrada: preparar almacenamiento, publicar un servicio y validar acceso por puerto.

---

## Objetivos del alumno (al finalizar el dia 3)

- Diagnosticar conectividad y configuracion IP con `nmcli`.
- Aplicar cambios de red de forma controlada (perfil, DNS, gateway, reinicio de conexion).
- Gestionar reglas basicas de `firewalld` por zona y servicio/puerto.
- Entender lo minimo de SELinux para troubleshooting inicial (`getenforce`, contextos y puertos).
- Completar un lab de extremo a extremo con validaciones.

---

## Distribucion recomendada de la sesion (4 horas)

- Bloque 1 (1h 15m): teoria guiada de red y seguridad.
- Bloque 2 (2h 15m): lab integrado del dia 3.
- Bloque 3 (30m): revision de errores frecuentes y checklist final.

---

## Cierre y puente al dia 4

Antes de cerrar el dia 3, valida que puedes:

- identificar y corregir problemas basicos de red;
- exponer un servicio minimamente seguro con firewall;
- reconocer cuando SELinux puede estar bloqueando una operacion.

Este bloque deja la base para troubleshooting mas avanzado en servicios de aplicacion y diagnostico de incidentes.
