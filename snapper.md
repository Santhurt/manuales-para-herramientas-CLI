# Snapper — Guía de referencia

> Gestión de snapshots con Btrfs. Snapper funciona de forma óptima con sistemas de archivos **Btrfs**, disponible por defecto en instalaciones recientes de Fedora.

---

## Tabla de contenidos

1. [Requisitos previos](#1-requisitos-previos)
2. [Instalación](#2-instalación)
3. [Configuración](#3-configuración)
4. [Gestión de snapshots](#4-gestión-de-snapshots)
5. [Ver diferencias entre snapshots](#5-ver-diferencias-entre-snapshots)
6. [Restauración](#6-restauración)
7. [Snapshots automáticos (Timeline)](#7-snapshots-automáticos-timeline)
8. [Gestión de espacio en disco](#8-gestión-de-espacio-en-disco)
9. [Referencia rápida de comandos](#9-referencia-rápida-de-comandos)

---

## 1. Requisitos previos

Verifica que tu sistema usa Btrfs antes de continuar.

```bash
# Verificar el sistema de archivos de la raíz
findmnt -T /

# Listar particiones y sus sistemas de archivos
lsblk -f
```

Busca `btrfs` en la columna `FSTYPE`. Si aparece `ext4` o `xfs`, snapper tendrá funcionalidad muy limitada (sin snapshots instantáneos ni rollback real).

---

## 2. Instalación

```bash
# Instalar snapper
sudo dnf install snapper

# Plugin de DNF: crea snapshots automáticos antes/después de cada operación dnf
sudo dnf install python3-dnf-plugin-snapper

# Instalar grub-btrfs para ver snapshots en el menú de GRUB (recomendado)
sudo dnf install grub-btrfs

# Regenerar configuración de GRUB después de instalar grub-btrfs
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

> **Plugin DNF:** con `python3-dnf-plugin-snapper` instalado, cada `dnf install`, `dnf remove` o `dnf update` crea automáticamente un par de snapshots `pre` y `post`, permitiéndote revertir los cambios causados por una actualización.

---

## 3. Configuración

### Crear configuraciones

Debes crear una configuración por cada subvolumen que quieras gestionar.

```bash
# Configuración para el subvolumen raíz
sudo snapper -c root create-config /

# Configuración para /home (requiere que sea un subvolumen Btrfs independiente)
sudo snapper -c home create-config /home

# Listar configuraciones existentes
snapper list-configs
```

### Editar parámetros

```bash
# Editar el archivo de configuración directamente
sudo nano /etc/snapper/configs/root

# O cambiar un parámetro puntual sin abrir el archivo
sudo snapper -c root set-config "NUMBER_LIMIT=20"
```

### Parámetros principales del archivo de configuración

```ini
# Habilitar snapshots automáticos por tiempo
TIMELINE_CREATE="yes"

# Habilitar limpieza automática de timeline
TIMELINE_CLEANUP="yes"

# Habilitar limpieza por número total
NUMBER_CLEANUP="yes"

# Límite máximo de snapshots numerados (manuales + DNF)
NUMBER_LIMIT="50"

# Cuántos snapshots conservar por período de tiempo
TIMELINE_LIMIT_HOURLY="5"
TIMELINE_LIMIT_DAILY="7"
TIMELINE_LIMIT_WEEKLY="4"
TIMELINE_LIMIT_MONTHLY="6"
TIMELINE_LIMIT_YEARLY="2"
```

---

## 4. Gestión de snapshots

### Crear snapshots

```bash
# Snapshot manual simple con descripción
sudo snapper -c root create --description "antes de cambios manuales"

# Snapshot tipo "pre" (antes de una operación)
# --print-number imprime el número asignado; guárdalo para crear el "post"
sudo snapper -c root create --type pre --print-number --description "pre-instalacion"

# Snapshot tipo "post" vinculado al "pre" número 5
sudo snapper -c root create --type post --pre-number 5 --description "post-instalacion"
```

### Listar snapshots

```bash
# Listar snapshots de la configuración "root"
snapper -c root list

# Listar snapshots de todas las configuraciones
snapper list --all
```

La columna `#0` siempre representa el estado actual del sistema de archivos, no un snapshot real.

Tipos de snapshot:

| Tipo     | Descripción                                      |
|----------|--------------------------------------------------|
| `single` | Snapshot independiente (creado manualmente)      |
| `pre`    | Snapshot tomado antes de una operación           |
| `post`   | Snapshot tomado después, vinculado a un `pre`    |
| `timeline` | Snapshot automático por tiempo                 |

### Modificar y eliminar

```bash
# Cambiar la descripción del snapshot número 3
sudo snapper -c root modify -d "nueva descripcion" 3

# Eliminar el snapshot número 3
sudo snapper -c root delete 3

# Eliminar un rango de snapshots (del 3 al 7)
sudo snapper -c root delete 3-7

# Eliminar múltiples snapshots no contiguos
sudo snapper -c root delete 3 5 8
```

---

## 5. Ver diferencias entre snapshots

```bash
# Listar archivos modificados entre el snapshot 2 y el 4
sudo snapper -c root status 2..4

# Comparar el estado actual (0) con el snapshot 3
sudo snapper -c root status 0..3

# Ver el diff real (estilo git) entre los snapshots 2 y 4
sudo snapper -c root diff 2..4

# Ver el diff de un archivo específico entre snapshots
sudo snapper -c root diff 2..4 -- /etc/fstab
```

### Prefijos del comando `status`

| Prefijo | Significado              |
|---------|--------------------------|
| `+`     | Archivo creado           |
| `-`     | Archivo eliminado        |
| `c`     | Contenido modificado     |
| `t`     | Tipo de archivo cambiado |
| `p`     | Permisos cambiados       |
| `u`     | Propietario cambiado     |
| `g`     | Grupo cambiado           |

---

## 6. Restauración

### Restaurar archivos individuales (sin reiniciar)

```bash
# Restaurar un archivo específico al estado del snapshot 2
# El orden 2..0 significa: "deshace cambios desde snapshot 2 hasta el estado actual"
sudo snapper -c root undochange 2..0 /etc/nginx/nginx.conf

# Restaurar múltiples archivos
sudo snapper -c root undochange 2..0 /etc/nginx/nginx.conf /etc/hosts

# Revertir TODOS los archivos modificados entre el snapshot 2 y el estado actual
sudo snapper -c root undochange 2..0
```

> Esta operación es segura y no requiere reinicio. Es la forma recomendada de revertir cambios causados por un par pre/post de DNF que salió mal.

### Restauración completa del sistema (rollback Btrfs)

> ⚠️ **Importante:** la restauración completa no puede hacerse con el sistema en ejecución. Requiere arrancar desde un snapshot (via GRUB) o desde un Live USB.

**Flujo de rollback completo:**

```
Reiniciar → GRUB → Btrfs Snapshots → Seleccionar snapshot → Arrancar en modo solo lectura → Ejecutar rollback → Reiniciar
```

```bash
# Desde una sesión de solo lectura en el snapshot, hacer rollback
sudo snapper -c root rollback 5

# Reiniciar para que los cambios tomen efecto
sudo reboot
```

El comando `rollback` crea un snapshot de seguridad del estado actual y establece el snapshot indicado como el nuevo subvolumen activo.

---

## 7. Snapshots automáticos (Timeline)

```bash
# Activar el timer que crea snapshots automáticos (cada hora)
sudo systemctl enable --now snapper-timeline.timer

# Activar el timer de limpieza automática (se ejecuta dos veces al día)
sudo systemctl enable --now snapper-cleanup.timer

# Verificar el estado de los timers
systemctl status snapper-timeline.timer
systemctl status snapper-cleanup.timer

# Ver cuándo se ejecutará el próximo snapshot automático
systemctl list-timers snapper*
```

> Los snapshots de timeline se conservan según los límites `TIMELINE_LIMIT_*` definidos en la configuración. La limpieza automática elimina los que superan esos límites.

---

## 8. Gestión de espacio en disco

Los snapshots Btrfs son copy-on-write: solo ocupan espacio proporcional a los archivos que han cambiado desde que se tomó el snapshot.

```bash
# Ver lista de snapshots con espacio usado por cada uno
snapper -c root list

# Ver uso total del sistema de archivos Btrfs
btrfs filesystem usage /

# Listar todos los subvolúmenes Btrfs (incluye snapshots)
sudo btrfs subvolume list /

# Limpieza manual inmediata (elimina snapshots que superan los límites)
sudo snapper -c root cleanup number

# Limpieza manual del timeline
sudo snapper -c root cleanup timeline
```

---

## 9. Referencia rápida de comandos

| Acción | Comando |
|--------|---------|
| Verificar sistema de archivos | `findmnt -T /` |
| Listar configuraciones | `snapper list-configs` |
| Crear configuración | `sudo snapper -c root create-config /` |
| Listar snapshots | `snapper -c root list` |
| Crear snapshot manual | `sudo snapper -c root create --description "texto"` |
| Eliminar snapshot | `sudo snapper -c root delete N` |
| Ver archivos modificados | `sudo snapper -c root status N..M` |
| Ver diff entre snapshots | `sudo snapper -c root diff N..M` |
| Restaurar archivos | `sudo snapper -c root undochange N..0 /ruta/archivo` |
| Restaurar todo el rango | `sudo snapper -c root undochange N..0` |
| Rollback completo | `sudo snapper -c root rollback N` |
| Activar timeline | `sudo systemctl enable --now snapper-timeline.timer` |
| Activar limpieza | `sudo systemctl enable --now snapper-cleanup.timer` |
| Limpieza manual | `sudo snapper -c root cleanup number` |
| Uso del disco Btrfs | `btrfs filesystem usage /` |
| Regenerar GRUB | `sudo grub2-mkconfig -o /boot/grub2/grub.cfg` |

---

*Referencia generada para Fedora Linux con Btrfs y Snapper.*
