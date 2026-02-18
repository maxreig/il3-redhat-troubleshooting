# Tema 1: Instalacion RHEL 8 (modo grafico y texto)

## Objetivo

Instalar un sistema base funcional (RHEL 8 o equivalente Rocky/Alma) y validar primer arranque para empezar troubleshooting desde una base estable.

![Infografia instalacion y primer arranque](images/tema1-instalacion-flujo.svg)

## Modos de instalacion

1. Modo grafico (Anaconda GUI): recomendado para formacion y labs.
1. Modo texto (TUI): util en servidores sin entorno grafico o consola limitada.

Ambos usan el mismo instalador y mismos conceptos clave:

1. origen de instalacion;
1. destino de instalacion (disco/particionado);
1. red y hostname;
1. usuarios y contrasena de `root`.

## Decisiones minimas recomendadas para el curso

1. Perfil de software:
1. instalacion minima si priorizas CLI;
1. "Server with GUI" si quieres escritorio para demo.
1. Particionado:
1. automatico para lab inicial;
1. manual cuando practiques almacenamiento avanzado.
1. Red:
1. activar interfaz en instalacion;
1. definir hostname coherente.

## Primer arranque: checklist tecnico

```bash
cat /etc/redhat-release
hostnamectl
ip a
ip route
free -h
df -h
timedatectl
```

Resultado esperado:

1. sistema identificado correctamente;
1. red operativa (IP + ruta);
1. disco y memoria coherentes;
1. fecha/hora correctas.

## Entorno de escritorio (si aplica)

Si instalas GUI, valida:

1. gestor grafico activo;
1. login operativo;
1. terminal funcional.

```bash
systemctl get-default
systemctl status gdm --no-pager
```

## Errores frecuentes

1. instalar sin red activa y no poder actualizar paquetes;
1. asignar poco disco y bloquear `/` o `/var` muy pronto;
1. olvidar validar DNS tras primer arranque.
