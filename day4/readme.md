# Dia 4 - Monitorizacion y troubleshooting integrado

## Indice del Dia 4 (material asociado)

- [`tema0-Heramienta_GUI.md`](tema0-Heramienta_GUI.md) -> equivalencia GUI/CLI para usuarios, permisos, storage/LVM, firewall y rendimiento.
- [`tema1-monitorizacion-basica.md`](tema1-monitorizacion-basica.md) -> monitorizacion operativa de CPU, RAM y disco (`top`, `free`, `df`, `iostat`).
- [`tema2-logs-journald.md`](tema2-logs-journald.md) -> uso practico de `journalctl`, jerarquia de logs y rutas clave en `/var/log`.
- [`tema3-diagnostico-problemas-comunes.md`](tema3-diagnostico-problemas-comunes.md) -> metodo guiado para conectividad, servicios caidos, falta de espacio y accesos.
- [`tema4-shell-script-bash.md`](tema4-shell-script-bash.md) -> nociones basicas de scripting en Bash para automatizar tareas operativas.
- [`comandos-dia4.md`](comandos-dia4.md) -> chuleta de comandos del dia 4.
- [`labs-dia4.md`](labs-dia4.md) -> lab final con escenario integrado de troubleshooting.

---

## Resumen ejecutivo del dia

1. Medir antes de tocar: estado real de CPU, memoria, disco e I/O.
1. Usar logs como fuente principal de evidencia, no como ultimo recurso.
1. Aplicar un flujo de diagnostico reproducible por tipo de incidente.
1. Correlacionar sintomas entre capas (recurso, servicio, red, permisos).
1. Cerrar con validacion y evidencia tecnica del arreglo.

---

## Objetivos del alumno (al finalizar el dia 4)

- Identificar cuellos de botella basicos con comandos de monitorizacion.
- Buscar eventos relevantes en `journald` y en `/var/log` por tiempo, unidad y severidad.
- Diagnosticar y corregir incidencias comunes en Linux de forma estructurada.
- Resolver un escenario de troubleshooting integrado con checklist de evidencias.

---

## Distribucion recomendada de la sesion (4 horas)

- Bloque 1 (1h 10m): monitorizacion basica y lectura operativa de sintomas.
- Bloque 2 (1h 00m): logs y journald (busqueda, filtros y timeline).
- Bloque 3 (1h 20m): diagnostico de problemas comunes (runbooks rapidos).
- Bloque 4 (30m): lab final integrado + cierre tecnico.

---

## Cierre del bloque formativo

Este dia consolida el metodo completo de troubleshooting del curso:

1. observar;
1. aislar hipotesis;
1. validar con evidencia;
1. corregir con riesgo controlado;
1. verificar resultado y documentar.
