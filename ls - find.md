# 🐧 Guía de Referencia: `ls` y `find` en Linux

> Guía de referencia rápida para los comandos `ls` y `find` en sistemas Linux/Unix.

---

## Tabla de Contenidos

- [🐧 Guía de Referencia: `ls` y `find` en Linux](#-gu%C3%ADa-de-referencia-ls-y-find-en-linux)
    - [Tabla de Contenidos](#tabla-de-contenidos)
    - [ls — Listar contenido de directorios](#ls--listar-contenido-de-directorios)
        - [Sintaxis](#sintaxis)
        - [Opciones más usadas](#opciones-m%C3%A1s-usadas)
        - [Ejemplos prácticos de ls](#ejemplos-pr%C3%A1cticos-de-ls)
        - [Combinaciones útiles de ls](#combinaciones-%C3%BAtiles-de-ls)
        - [Formato de salida en modo largo (-l)](#formato-de-salida-en-modo-largo--l)
    - [find — Buscar archivos y directorios](#find--buscar-archivos-y-directorios)
        - [Sintaxis](#sintaxis-1)
        - [Opciones de búsqueda por nombre](#opciones-de-b%C3%BAsqueda-por-nombre)
        - [Opciones de búsqueda por tipo](#opciones-de-b%C3%BAsqueda-por-tipo)
        - [Opciones de búsqueda por tamaño](#opciones-de-b%C3%BAsqueda-por-tama%C3%B1o)
        - [Opciones de búsqueda por tiempo](#opciones-de-b%C3%BAsqueda-por-tiempo)
        - [Opciones de búsqueda por permisos y propietario](#opciones-de-b%C3%BAsqueda-por-permisos-y-propietario)
        - [Acciones de find](#acciones-de-find)
        - [Operadores lógicos en find](#operadores-l%C3%B3gicos-en-find)
        - [Ejemplos prácticos de find](#ejemplos-pr%C3%A1cticos-de-find)
    - [Combinando ls y find](#combinando-ls-y-find)
    - [Tabla resumen de opciones](#tabla-resumen-de-opciones)
    - [Tips y buenas prácticas](#tips-y-buenas-pr%C3%A1cticas)

---

## ls — Listar contenido de directorios

### Sintaxis

```bash
ls [OPCIONES] [ARCHIVO/DIRECTORIO...]
```

### Opciones más usadas

|Opción|Descripción|
|---|---|
|`-l`|Formato largo (permisos, propietario, tamaño, fecha)|
|`-a`|Muestra archivos ocultos (los que empiezan con `.`)|
|`-A`|Como `-a` pero sin mostrar `.` y `..`|
|`-h`|Tamaños legibles para humanos (KB, MB, GB) — usar con `-l`|
|`-R`|Listado recursivo de subdirectorios|
|`-r`|Orden inverso|
|`-S`|Ordenar por tamaño (mayor primero)|
|`-t`|Ordenar por fecha de modificación (más reciente primero)|
|`-X`|Ordenar por extensión|
|`-1`|Un archivo por línea|
|`-d`|Muestra el directorio en sí, no su contenido|
|`-i`|Muestra el número de inodo|
|`-n`|Como `-l` pero muestra UID y GID numéricos|
|`--color`|Colorea la salida según tipo de archivo|
|`-F`|Añade indicador al nombre (`/` dir, `*` ejecutable, `@` enlace)|

### Ejemplos prácticos de ls

```bash
# Listado simple del directorio actual
ls

# Listado largo con tamaños legibles
ls -lh

# Mostrar todos los archivos incluyendo ocultos
ls -la

# Listar un directorio específico
ls -l /etc

# Ordenar por tamaño (mayor primero)
ls -lhS

# Ordenar por fecha de modificación (más reciente primero)
ls -lt

# Ordenar por fecha, orden inverso (más antiguo primero)
ls -ltr

# Listar solo directorios
ls -d */

# Ver info de un directorio sin ver su contenido
ls -ld /home/usuario

# Listado recursivo
ls -R /var/log

# Un archivo por línea (útil para scripts)
ls -1

# Listar con indicadores de tipo
ls -F

# Listar archivos .log en /var/log
ls -lh /var/log/*.log
```

### Combinaciones útiles de ls

```bash
# Los 10 archivos más grandes
ls -lhS | head -n 11

# Archivos modificados hoy (combinado con find, ver más abajo)
ls -lt | head

# Contar archivos en el directorio actual
ls -1 | wc -l

# Listar solo archivos (sin directorios)
ls -p | grep -v /

# Listar solo directorios
ls -p | grep /
```

### Formato de salida en modo largo (-l)

```
permisos  enlaces  propietario  grupo  tamaño  fecha        nombre
drwxr-xr-x  2     root         root   4096    Apr 20 10:30  etc/
-rw-r--r--  1     user         user   1234    May  1 09:15  archivo.txt
lrwxrwxrwx  1     root         root      7    Jan 15 08:00  lib -> usr/lib
│           │     │            │      │       │             │
│           │     │            │      │       └── Nombre del archivo
│           │     │            │      └────────── Tamaño en bytes
│           │     │            └───────────────── Grupo propietario
│           │     └────────────────────────────── Usuario propietario
│           └──────────────────────────────────── Número de enlaces duros
└──────────────────────────────────────────────── Tipo y permisos
```

**Tipos de archivo (primer carácter):**

|Carácter|Tipo|
|---|---|
|`-`|Archivo regular|
|`d`|Directorio|
|`l`|Enlace simbólico|
|`c`|Dispositivo de caracteres|
|`b`|Dispositivo de bloques|
|`p`|Tubería (pipe)|
|`s`|Socket|

---

## find — Buscar archivos y directorios

### Sintaxis

```bash
find [RUTA...] [EXPRESIÓN]
```

> Si no se especifica ruta, `find` usa el directorio actual (`.`) por defecto.

### Opciones de búsqueda por nombre

|Opción|Descripción|
|---|---|
|`-name PATRÓN`|Busca por nombre exacto (sensible a mayúsculas)|
|`-iname PATRÓN`|Busca por nombre (insensible a mayúsculas)|
|`-path PATRÓN`|Busca por ruta completa|
|`-ipath PATRÓN`|Busca por ruta completa (insensible a mayúsculas)|
|`-regex PATRÓN`|Busca usando expresión regular|

```bash
# Buscar por nombre exacto
find /home -name "archivo.txt"

# Buscar todos los .log (wildcard)
find /var/log -name "*.log"

# Búsqueda insensible a mayúsculas
find / -iname "readme*"

# Buscar por ruta
find / -path "*/config/*.conf"
```

### Opciones de búsqueda por tipo

|Valor|Tipo de archivo|
|---|---|
|`-type f`|Archivo regular|
|`-type d`|Directorio|
|`-type l`|Enlace simbólico|
|`-type c`|Dispositivo de caracteres|
|`-type b`|Dispositivo de bloques|
|`-type p`|Tubería (pipe)|
|`-type s`|Socket|

```bash
# Solo archivos regulares
find . -type f

# Solo directorios
find /home -type d

# Solo enlaces simbólicos
find /usr -type l
```

### Opciones de búsqueda por tamaño

```bash
find RUTA -size [+|-]N[UNIDAD]
```

|Unidad|Descripción|
|---|---|
|`c`|Bytes|
|`k`|Kilobytes (1024 bytes)|
|`M`|Megabytes|
|`G`|Gigabytes|

```bash
# Archivos de exactamente 100k
find . -size 100k

# Archivos mayores a 100MB
find / -size +100M

# Archivos menores a 1KB
find . -size -1k

# Archivos entre 1MB y 100MB
find . -size +1M -size -100M
```

### Opciones de búsqueda por tiempo

|Opción|Descripción|
|---|---|
|`-mtime N`|Modificado hace N días|
|`-atime N`|Accedido hace N días|
|`-ctime N`|Cambiado (metadatos) hace N días|
|`-mmin N`|Modificado hace N minutos|
|`-amin N`|Accedido hace N minutos|
|`-newer ARCHIVO`|Más reciente que el archivo indicado|

> Usar `+N` para "más de N días" y `-N` para "menos de N días".

```bash
# Modificados en las últimas 24 horas
find . -mtime -1

# Modificados hace más de 30 días
find /tmp -mtime +30

# Modificados en los últimos 60 minutos
find . -mmin -60

# Más recientes que un archivo de referencia
find . -newer referencia.txt

# Archivos no accedidos en más de 90 días
find /home -atime +90
```

### Opciones de búsqueda por permisos y propietario

```bash
# Por propietario (usuario)
find /home -user juan

# Por grupo
find /var -group www-data

# Por permisos exactos
find . -perm 644

# Archivos con al menos estos permisos
find . -perm -644

# Archivos ejecutables por cualquiera
find /usr/bin -perm /111

# Archivos con setuid activo (potencial riesgo de seguridad)
find / -perm -4000 -type f

# Archivos con setgid activo
find / -perm -2000 -type f

# Archivos sin propietario (huérfanos)
find / -nouser

# Archivos sin grupo asignado
find / -nogroup
```

### Acciones de find

|Acción|Descripción|
|---|---|
|`-print`|Imprime la ruta (comportamiento por defecto)|
|`-print0`|Imprime separado por null (para `xargs -0`)|
|`-ls`|Lista detallada (como `ls -l`)|
|`-delete`|Elimina los archivos encontrados ⚠️|
|`-exec CMD {} \;`|Ejecuta un comando por cada resultado|
|`-exec CMD {} +`|Ejecuta un comando pasando todos los resultados juntos|
|`-ok CMD {} \;`|Como `-exec` pero pide confirmación|

```bash
# Listar detalladamente los resultados
find /var/log -name "*.log" -ls

# Eliminar archivos .tmp (¡cuidado!)
find /tmp -name "*.tmp" -delete

# Ejecutar comando por cada archivo encontrado
find . -name "*.sh" -exec chmod +x {} \;

# Mostrar contenido de cada archivo encontrado
find . -name "*.conf" -exec cat {} \;

# Más eficiente: pasar todos los resultados a xargs
find . -name "*.txt" -print0 | xargs -0 grep "patron"

# Pedir confirmación antes de borrar
find . -name "*.bak" -ok rm {} \;

# Copiar archivos encontrados a otro directorio
find . -name "*.jpg" -exec cp {} /backup/fotos/ \;
```

### Operadores lógicos en find

|Operador|Descripción|
|---|---|
|`-and` / `-a`|Y lógico (implícito entre condiciones)|
|`-or` / `-o`|O lógico|
|`-not` / `!`|Negación|
|`\( \)`|Agrupación de condiciones|

```bash
# AND implícito: archivos .txt modificados en últimas 24h
find . -name "*.txt" -mtime -1

# OR: archivos .jpg o .png
find . -name "*.jpg" -o -name "*.png"

# NOT: todo excepto archivos .log
find . -not -name "*.log"

# Agrupación: (.jpg o .png) Y mayor a 1MB
find . \( -name "*.jpg" -o -name "*.png" \) -size +1M

# Archivos de usuario juan que no sean .tmp
find /home/juan -user juan -not -name "*.tmp"
```

### Ejemplos prácticos de find

```bash
# Encontrar y comprimir logs viejos
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;

# Limpiar archivos temporales viejos
find /tmp -type f -mtime +3 -delete

# Buscar archivos duplicados por nombre
find . -name "*.mp3" | sort

# Ver los 10 archivos más grandes
find . -type f -exec du -h {} + | sort -rh | head -10

# Buscar archivos de configuración editados recientemente
find /etc -name "*.conf" -mtime -7 -ls

# Buscar y reemplazar texto en múltiples archivos
find . -name "*.txt" -exec sed -i 's/viejo/nuevo/g' {} +

# Respaldar archivos modificados hoy
find . -mtime -1 -type f -exec cp {} /backup/ \;

# Buscar archivos vacíos
find . -type f -empty

# Buscar directorios vacíos
find . -type d -empty

# Contar archivos por extensión
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn

# Buscar archivos con permisos de escritura para todos (riesgo de seguridad)
find / -type f -perm -o=w 2>/dev/null
```

---

## Combinando ls y find

```bash
# Listar detalladamente los archivos encontrados con find
find /var/log -name "*.log" -exec ls -lh {} \;

# Ver el tamaño de todos los archivos .conf en /etc
find /etc -name "*.conf" -exec ls -lh {} \; | awk '{print $5, $9}'

# Buscar archivos grandes y listarlos con formato legible
find ~ -size +50M -exec ls -lh {} \; 2>/dev/null

# Encontrar los .sh más recientes y listarlos
find . -name "*.sh" -mtime -7 -exec ls -lt {} +

# Listar archivos encontrados con número de inodo
find /usr/bin -type f -name "py*" -exec ls -i {} \;
```

---

## Tabla resumen de opciones

### ls — Resumen

|Categoría|Opciones|
|---|---|
|**Formato**|`-l`, `-1`, `-F`, `-h`, `--color`|
|**Visibilidad**|`-a`, `-A`, `-d`, `-R`|
|**Ordenamiento**|`-S`, `-t`, `-r`, `-X`|
|**Info extra**|`-i`, `-n`|

### find — Resumen

|Categoría|Opciones|
|---|---|
|**Nombre**|`-name`, `-iname`, `-path`, `-regex`|
|**Tipo**|`-type f/d/l/c/b/p/s`|
|**Tamaño**|`-size [+\|-]N[c\|k\|M\|G]`|
|**Tiempo**|`-mtime`, `-atime`, `-ctime`, `-mmin`, `-newer`|
|**Permisos**|`-perm`, `-user`, `-group`, `-nouser`|
|**Acciones**|`-print`, `-ls`, `-delete`, `-exec`, `-ok`|
|**Lógica**|`-and`, `-or`, `-not`, `\( \)`|

---

## Tips y buenas prácticas

### Para ls

- Crea un alias permanente en tu `~/.bashrc` o `~/.zshrc`:
    
    ```bash
    alias ll='ls -lhA --color=auto'
    alias lt='ls -lhtr'
    alias lS='ls -lhS'
    ```
    
- Usa `ls --color=auto` para distinguir visualmente tipos de archivos.
    
- Para scripts, prefiere `ls -1` o directamente globbing de shell (`for f in *.txt`).
    

### Para find

- Redirige errores de permisos con `2>/dev/null` para salidas más limpias:
    
    ```bash
    find / -name "archivo" 2>/dev/null
    ```
    
- Usa `-maxdepth` para limitar la profundidad de búsqueda y acelerar la ejecución:
    
    ```bash
    find . -maxdepth 2 -name "*.conf"
    ```
    
- Usa `-mindepth` para evitar el directorio raíz de la búsqueda:
    
    ```bash
    find . -mindepth 1 -maxdepth 1 -type d
    ```
    
- **Antes de usar `-delete`**, prueba primero sin esa opción para ver qué se eliminaría:
    
    ```bash
    # Primero verificar:
    find /tmp -name "*.tmp" -mtime +7
    # Luego borrar:
    find /tmp -name "*.tmp" -mtime +7 -delete
    ```
    
- Para manejar nombres de archivo con espacios, usa `-print0` con `xargs -0`:
    
    ```bash
    find . -name "*.txt" -print0 | xargs -0 wc -l
    ```
    
- Usa `-prune` para excluir directorios de la búsqueda:
    
    ```bash
    find . -path ./node_modules -prune -o -name "*.js" -print
    ```
    

---

> **Nota:** Algunos comportamientos pueden variar ligeramente entre distribuciones Linux y macOS (BSD). En macOS, `find` no soporta `-printf` y algunas opciones difieren. Se recomienda instalar `findutils` de GNU en macOS vía Homebrew para comportamiento consistente.