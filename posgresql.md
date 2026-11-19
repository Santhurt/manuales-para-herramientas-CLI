# Manual de PostgreSQL en CLI

> Manual de referencia rápida para trabajar con PostgreSQL desde la terminal.

---

## Índice

1. [Chuleta rápida](#1-chuleta-rápida)
2. [Conexión con psql](#2-conexión-con-psql)
3. [Metacomandos de psql](#3-metacomandos-de-psql)
4. [Bases de datos](#4-bases-de-datos)
5. [Tablas y esquemas](#5-tablas-y-esquemas)
6. [Tipos de datos](#6-tipos-de-datos)
7. [Consultas (CRUD)](#7-consultas-crud)
8. [Índices y rendimiento](#8-índices-y-rendimiento)
9. [Usuarios, roles y permisos](#9-usuarios-roles-y-permisos)
10. [Backup y restauración](#10-backup-y-restauración)
11. [Importar y exportar datos](#11-importar-y-exportar-datos)
12. [Transacciones](#12-transacciones)
13. [Administración del servidor](#13-administración-del-servidor)
14. [Configuración de psql (.psqlrc)](#14-configuración-de-psql-psqlrc)
15. [Solución de problemas](#15-solución-de-problemas)

---

## 1. Chuleta rápida

Los comandos que se usan el 90% del tiempo.

Comando: `sudo -u postgres psql`

| Acción | Comando |
|---|---|
| Conectar | `psql -U usuario -d basededatos` |
| Listar bases de datos | `\l` |
| Cambiar de base de datos | `\c nombre_bd` |
| Listar tablas | `\dt` |
| Describir tabla | `\d nombre_tabla` |
| Listar usuarios/roles | `\du` |
| Ejecutar archivo SQL | `\i archivo.sql` |
| Historial de comandos | `\s` |
| Salida vertical (legible) | `\x` |
| Ayuda de SQL | `\h SELECT` |
| Ayuda de psql | `\?` |
| Salir | `\q` |

**Backup y restauración en una línea:**

```bash
pg_dump -U usuario basededatos > backup.sql        # Exportar
psql -U usuario -d basededatos < backup.sql        # Importar
```

---

## 2. Conexión con psql

### Sintaxis básica

```bash
psql -U usuario -d basededatos -h host -p 5432
```

| Flag | Descripción | Valor por defecto |
|---|---|---|
| `-U` | Usuario | Usuario del sistema |
| `-d` | Base de datos | Nombre del usuario |
| `-h` | Host | `localhost` / socket local |
| `-p` | Puerto | `5432` |
| `-W` | Forzar solicitud de contraseña | — |
| `-c` | Ejecutar un comando y salir | — |
| `-f` | Ejecutar un archivo SQL y salir | — |
| `-l` | Listar bases de datos y salir | — |
| `-A` | Salida sin alineación | — |
| `-t` | Solo filas, sin encabezados | — |

### Ejemplos

```bash
# Conexión local estándar
psql -U postgres

# Conexión remota
psql -U admin -h db.ejemplo.com -p 5432 -d produccion

# Ejecutar un comando sin entrar a la shell
psql -U postgres -d mydb -c "SELECT count(*) FROM usuarios;"

# Ejecutar un script
psql -U postgres -d mydb -f schema.sql

# Cadena de conexión (URI)
psql "postgresql://usuario:clave@host:5432/basededatos"

# Conexión con SSL obligatorio
psql "postgresql://usuario@host:5432/db?sslmode=require"
```

### Variables de entorno

Evitan repetir parámetros en cada comando.

```bash
export PGUSER=postgres
export PGPASSWORD=miclave      # Cuidado: queda en el historial del shell
export PGHOST=localhost
export PGPORT=5432
export PGDATABASE=mydb

psql   # Ya no necesita flags
```

### Archivo .pgpass (recomendado para contraseñas)

```bash
# Formato: host:puerto:basededatos:usuario:contraseña
echo "localhost:5432:*:postgres:miclave" >> ~/.pgpass
chmod 600 ~/.pgpass    # Obligatorio, si no PostgreSQL lo ignora
```

---

## 3. Metacomandos de psql

Los metacomandos empiezan con `\` y son propios de psql (no son SQL).

### Información

| Comando | Descripción |
|---|---|
| `\l` o `\l+` | Listar bases de datos (`+` = con tamaño) |
| `\dt` | Listar tablas |
| `\dt+` | Tablas con tamaño y descripción |
| `\dt esquema.*` | Tablas de un esquema específico |
| `\d tabla` | Estructura de una tabla |
| `\d+ tabla` | Estructura detallada |
| `\dv` | Listar vistas |
| `\di` | Listar índices |
| `\ds` | Listar secuencias |
| `\df` | Listar funciones |
| `\dn` | Listar esquemas |
| `\du` | Listar roles/usuarios |
| `\dp` o `\z` | Permisos de tablas |
| `\dx` | Extensiones instaladas |
| `\conninfo` | Info de la conexión actual |

### Navegación y ejecución

| Comando | Descripción |
|---|---|
| `\c bd [usuario]` | Cambiar de base de datos o usuario |
| `\i archivo.sql` | Ejecutar un archivo SQL |
| `\o archivo.txt` | Redirigir la salida a un archivo |
| `\o` | Volver a la salida por pantalla |
| `\e` | Abrir el editor para escribir la consulta |
| `\ef funcion` | Editar una función |
| `\g` | Reejecutar la última consulta |
| `\s` | Ver historial |
| `\s archivo` | Guardar historial a archivo |
| `\! comando` | Ejecutar un comando del shell |
| `\cd ruta` | Cambiar de directorio |
| `\q` | Salir |

### Formato de salida

| Comando | Descripción |
|---|---|
| `\x` | Alternar salida expandida (vertical) |
| `\x auto` | Expandida solo si no cabe en pantalla |
| `\a` | Alternar alineación |
| `\t` | Ocultar encabezados y pie |
| `\pset null 'NULL'` | Mostrar texto para valores nulos |
| `\pset border 2` | Bordes de la tabla (0, 1, 2) |
| `\pset format csv` | Formato de salida (`aligned`, `csv`, `html`, `json`) |
| `\timing` | Mostrar tiempo de ejecución de cada consulta |

> **Tip:** `\x auto` y `\timing` son los dos ajustes más útiles del día a día. Ponlos en tu `.psqlrc` (ver [sección 14](#14-configuración-de-psql-psqlrc)).

### Ayuda

```sql
\?              -- Todos los metacomandos
\h              -- Todos los comandos SQL
\h CREATE TABLE -- Sintaxis de un comando específico
```

---

## 4. Bases de datos

### Desde el shell

```bash
createdb -U postgres mi_base
createdb -U postgres -O propietario -E UTF8 mi_base
dropdb   -U postgres mi_base
```

### Desde SQL

```sql
CREATE DATABASE mi_base;

CREATE DATABASE mi_base
    WITH OWNER = mi_usuario
         ENCODING = 'UTF8'
         LC_COLLATE = 'es_ES.UTF-8'
         LC_CTYPE = 'es_ES.UTF-8'
         TEMPLATE = template0;

ALTER DATABASE mi_base RENAME TO nueva_base;
ALTER DATABASE mi_base OWNER TO otro_usuario;

DROP DATABASE mi_base;
DROP DATABASE IF EXISTS mi_base;
```

### Consultar tamaño

```sql
-- Tamaño de una base de datos
SELECT pg_size_pretty(pg_database_size('mi_base'));

-- Todas las bases ordenadas por tamaño
SELECT datname, pg_size_pretty(pg_database_size(datname)) AS tamano
FROM pg_database
ORDER BY pg_database_size(datname) DESC;
```

### Cerrar conexiones antes de eliminar

```sql
-- Necesario si "database is being accessed by other users"
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'mi_base' AND pid <> pg_backend_pid();
```

---

## 5. Tablas y esquemas

### Crear tablas

```sql
CREATE TABLE usuarios (
    id          SERIAL PRIMARY KEY,
    email       VARCHAR(255) NOT NULL UNIQUE,
    nombre      TEXT NOT NULL,
    edad        INTEGER CHECK (edad >= 0),
    activo      BOOLEAN DEFAULT true,
    creado_en   TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE pedidos (
    id          BIGSERIAL PRIMARY KEY,
    usuario_id  INTEGER NOT NULL REFERENCES usuarios(id) ON DELETE CASCADE,
    total       NUMERIC(10,2) NOT NULL,
    estado      TEXT DEFAULT 'pendiente',
    creado_en   TIMESTAMPTZ DEFAULT now()
);

-- Crear solo si no existe
CREATE TABLE IF NOT EXISTS logs (...);

-- Copiar estructura de otra tabla
CREATE TABLE usuarios_backup (LIKE usuarios INCLUDING ALL);

-- Crear a partir de una consulta
CREATE TABLE usuarios_activos AS SELECT * FROM usuarios WHERE activo;

-- Tabla temporal (se borra al cerrar la sesión)
CREATE TEMP TABLE tmp_datos (id INT, valor TEXT);
```

### Modificar tablas (ALTER TABLE)

```sql
-- Columnas
ALTER TABLE usuarios ADD COLUMN telefono VARCHAR(20);
ALTER TABLE usuarios DROP COLUMN telefono;
ALTER TABLE usuarios RENAME COLUMN nombre TO nombre_completo;
ALTER TABLE usuarios ALTER COLUMN edad TYPE BIGINT;
ALTER TABLE usuarios ALTER COLUMN nombre SET NOT NULL;
ALTER TABLE usuarios ALTER COLUMN nombre DROP NOT NULL;
ALTER TABLE usuarios ALTER COLUMN activo SET DEFAULT false;

-- Restricciones
ALTER TABLE usuarios ADD CONSTRAINT uq_email UNIQUE (email);
ALTER TABLE usuarios ADD CONSTRAINT ck_edad CHECK (edad BETWEEN 0 AND 130);
ALTER TABLE pedidos ADD CONSTRAINT fk_usuario
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id);
ALTER TABLE usuarios DROP CONSTRAINT uq_email;

-- Tabla
ALTER TABLE usuarios RENAME TO clientes;
ALTER TABLE usuarios OWNER TO nuevo_dueno;
ALTER TABLE usuarios SET SCHEMA ventas;
```

### Eliminar y vaciar

```sql
DROP TABLE usuarios;
DROP TABLE IF EXISTS usuarios CASCADE;   -- CASCADE elimina dependencias

TRUNCATE usuarios;                        -- Vacía, mucho más rápido que DELETE
TRUNCATE usuarios RESTART IDENTITY;       -- Reinicia los contadores SERIAL
TRUNCATE usuarios, pedidos CASCADE;
```

### Esquemas

```sql
CREATE SCHEMA ventas;
CREATE SCHEMA IF NOT EXISTS ventas AUTHORIZATION mi_usuario;

DROP SCHEMA ventas CASCADE;

-- Ver y cambiar el search_path
SHOW search_path;
SET search_path TO ventas, public;
```

### Tamaño de tablas

```sql
SELECT
    schemaname,
    relname AS tabla,
    pg_size_pretty(pg_total_relation_size(relid)) AS tamano_total
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 20;
```

---

## 6. Tipos de datos

### Los más usados

| Categoría | Tipo | Notas |
|---|---|---|
| **Enteros** | `SMALLINT` | 2 bytes |
| | `INTEGER` / `INT` | 4 bytes, el más común |
| | `BIGINT` | 8 bytes |
| | `SERIAL` / `BIGSERIAL` | Autoincremental |
| **Decimales** | `NUMERIC(p,s)` | Exacto — usar para dinero |
| | `REAL` / `DOUBLE PRECISION` | Coma flotante, inexacto |
| **Texto** | `TEXT` | Longitud ilimitada, sin penalización |
| | `VARCHAR(n)` | Con límite |
| | `CHAR(n)` | Rellena con espacios, evitar |
| **Fecha/hora** | `DATE` | Solo fecha |
| | `TIMESTAMP` | Fecha y hora sin zona |
| | `TIMESTAMPTZ` | **Con zona horaria — usar este** |
| | `INTERVAL` | Duración |
| **Otros** | `BOOLEAN` | `true` / `false` / `NULL` |
| | `UUID` | Identificador único |
| | `JSONB` | JSON binario indexable |
| | `TEXT[]` | Arreglos de cualquier tipo |
| | `INET` / `CIDR` | Direcciones de red |

> **Recomendaciones:** usa `TEXT` en vez de `VARCHAR(n)` salvo que necesites el límite real; usa `TIMESTAMPTZ` siempre en lugar de `TIMESTAMP`; usa `NUMERIC` para dinero, nunca `FLOAT`; usa `JSONB` en lugar de `JSON`.

### Identidad moderna (preferible a SERIAL)

```sql
CREATE TABLE productos (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nombre TEXT NOT NULL
);
```

### JSONB

```sql
CREATE TABLE eventos (
    id SERIAL PRIMARY KEY,
    datos JSONB
);

INSERT INTO eventos (datos) VALUES ('{"tipo":"login","user":{"id":42}}');

SELECT datos->>'tipo'         FROM eventos;   -- Texto
SELECT datos->'user'->>'id'   FROM eventos;   -- Anidado
SELECT datos #>> '{user,id}'  FROM eventos;   -- Ruta

SELECT * FROM eventos WHERE datos @> '{"tipo":"login"}';   -- Contiene
SELECT * FROM eventos WHERE datos ? 'tipo';                -- Tiene la clave

CREATE INDEX idx_eventos_datos ON eventos USING GIN (datos);
```

---

## 7. Consultas (CRUD)

### INSERT

```sql
INSERT INTO usuarios (email, nombre) VALUES ('a@b.com', 'Ana');

-- Múltiples filas
INSERT INTO usuarios (email, nombre) VALUES
    ('a@b.com', 'Ana'),
    ('c@d.com', 'Carlos');

-- Devolver la fila insertada
INSERT INTO usuarios (email, nombre) VALUES ('a@b.com', 'Ana')
RETURNING id, creado_en;

-- UPSERT: insertar o actualizar si hay conflicto
INSERT INTO usuarios (email, nombre) VALUES ('a@b.com', 'Ana María')
ON CONFLICT (email) DO UPDATE SET nombre = EXCLUDED.nombre;

-- Ignorar duplicados
INSERT INTO usuarios (email, nombre) VALUES ('a@b.com', 'Ana')
ON CONFLICT DO NOTHING;

-- Insertar desde otra consulta
INSERT INTO archivo_usuarios SELECT * FROM usuarios WHERE NOT activo;
```

### SELECT

```sql
SELECT * FROM usuarios;
SELECT nombre, email FROM usuarios WHERE activo = true;

-- Filtros
WHERE edad > 18 AND activo
WHERE nombre ILIKE '%ana%'          -- ILIKE = sin distinguir mayúsculas
WHERE estado IN ('activo','pausa')
WHERE creado_en BETWEEN '2024-01-01' AND '2024-12-31'
WHERE telefono IS NULL
WHERE email ~ '^[a-z]+@'            -- Expresión regular

-- Orden, límite y paginación
SELECT * FROM usuarios ORDER BY creado_en DESC NULLS LAST LIMIT 10 OFFSET 20;

-- Agregación
SELECT estado, count(*), avg(total), sum(total)
FROM pedidos
GROUP BY estado
HAVING count(*) > 5
ORDER BY count(*) DESC;

-- DISTINCT
SELECT DISTINCT estado FROM pedidos;
SELECT DISTINCT ON (usuario_id) * FROM pedidos ORDER BY usuario_id, creado_en DESC;
```

### JOINs

```sql
-- INNER: solo coincidencias
SELECT u.nombre, p.total
FROM usuarios u
JOIN pedidos p ON p.usuario_id = u.id;

-- LEFT: todos los de la izquierda
SELECT u.nombre, COALESCE(sum(p.total), 0) AS gastado
FROM usuarios u
LEFT JOIN pedidos p ON p.usuario_id = u.id
GROUP BY u.id, u.nombre;

-- Otros: RIGHT JOIN, FULL OUTER JOIN, CROSS JOIN
```

### UPDATE

```sql
UPDATE usuarios SET activo = false WHERE id = 5;

UPDATE usuarios
SET nombre = 'Ana', activo = true
WHERE email = 'a@b.com'
RETURNING *;

-- Con datos de otra tabla
UPDATE usuarios u
SET total_pedidos = sub.n
FROM (SELECT usuario_id, count(*) AS n FROM pedidos GROUP BY usuario_id) sub
WHERE u.id = sub.usuario_id;
```

> ⚠️ **Siempre verifica el `WHERE` antes de ejecutar un `UPDATE` o `DELETE`.** Un buen hábito: escribe primero el `SELECT` con el mismo `WHERE`.

### DELETE

```sql
DELETE FROM usuarios WHERE id = 5;
DELETE FROM usuarios WHERE creado_en < now() - INTERVAL '1 year' RETURNING id;
DELETE FROM usuarios;   -- Borra todo (usa TRUNCATE si es lo que quieres)
```

### CTEs y funciones de ventana

```sql
-- CTE: subconsulta con nombre
WITH ventas_mes AS (
    SELECT date_trunc('month', creado_en) AS mes, sum(total) AS total
    FROM pedidos
    GROUP BY 1
)
SELECT * FROM ventas_mes WHERE total > 10000;

-- Funciones de ventana
SELECT
    nombre,
    total,
    ROW_NUMBER() OVER (ORDER BY total DESC)          AS puesto,
    RANK()       OVER (PARTITION BY estado ORDER BY total DESC) AS rank_estado,
    sum(total)   OVER (ORDER BY creado_en)           AS acumulado,
    LAG(total)   OVER (ORDER BY creado_en)           AS anterior
FROM pedidos;
```

### Funciones útiles

```sql
-- Fechas
SELECT now(), current_date, current_time;
SELECT date_trunc('month', now());
SELECT extract(year FROM creado_en);
SELECT now() - INTERVAL '7 days';
SELECT age(fecha_nacimiento);
SELECT to_char(now(), 'YYYY-MM-DD HH24:MI');

-- Texto
SELECT upper(x), lower(x), length(x), trim(x);
SELECT concat(a, ' ', b), a || ' ' || b;
SELECT substring(x FROM 1 FOR 5), replace(x, 'a', 'b');
SELECT split_part('a,b,c', ',', 2);   -- 'b'

-- Nulos y condicionales
SELECT COALESCE(telefono, 'sin dato');
SELECT NULLIF(valor, 0);
SELECT CASE WHEN edad < 18 THEN 'menor' ELSE 'adulto' END;

-- Agregación en arreglos/texto
SELECT array_agg(nombre), string_agg(nombre, ', ') FROM usuarios;
```

---

## 8. Índices y rendimiento

### Crear índices

```sql
CREATE INDEX idx_usuarios_email ON usuarios(email);
CREATE UNIQUE INDEX idx_usuarios_email_uq ON usuarios(email);

-- Compuesto (el orden de las columnas importa)
CREATE INDEX idx_pedidos_usuario_fecha ON pedidos(usuario_id, creado_en DESC);

-- Parcial: solo indexa un subconjunto
CREATE INDEX idx_usuarios_activos ON usuarios(email) WHERE activo;

-- Sobre una expresión
CREATE INDEX idx_usuarios_lower_email ON usuarios(lower(email));

-- Sin bloquear la tabla (producción)
CREATE INDEX CONCURRENTLY idx_grande ON tabla_grande(columna);

DROP INDEX idx_usuarios_email;
REINDEX TABLE usuarios;
```

### Tipos de índice

| Tipo | Uso |
|---|---|
| `BTREE` | Por defecto. Igualdad, rangos, orden |
| `GIN` | JSONB, arreglos, búsqueda de texto completo |
| `GIST` | Datos geométricos, rangos |
| `HASH` | Solo igualdad, raramente necesario |
| `BRIN` | Tablas enormes con datos correlacionados físicamente |

### Analizar consultas

```sql
EXPLAIN SELECT * FROM usuarios WHERE email = 'a@b.com';

-- Ejecuta realmente y muestra tiempos reales
EXPLAIN ANALYZE SELECT * FROM usuarios WHERE email = 'a@b.com';

EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT ...;
```

**Qué buscar en el plan:**

- `Seq Scan` sobre una tabla grande → probablemente falta un índice
- `Index Scan` / `Index Only Scan` → bien
- Diferencia grande entre `rows` estimadas y reales → hace falta `ANALYZE`
- `Nested Loop` con muchas filas → puede indicar un join ineficiente

### Mantenimiento

```sql
ANALYZE usuarios;          -- Actualiza estadísticas del planificador
VACUUM usuarios;           -- Recupera espacio de filas muertas
VACUUM ANALYZE usuarios;   -- Ambos
VACUUM FULL usuarios;      -- Reescribe la tabla, BLOQUEA, libera espacio al SO
```

### Diagnóstico

```sql
-- Consultas en ejecución
SELECT pid, now() - query_start AS duracion, state, query
FROM pg_stat_activity
WHERE state <> 'idle'
ORDER BY duracion DESC;

-- Cancelar / matar una consulta
SELECT pg_cancel_backend(pid);      -- Suave
SELECT pg_terminate_backend(pid);   -- Forzado

-- Bloqueos
SELECT * FROM pg_locks WHERE NOT granted;

-- Índices que nunca se usan
SELECT relname, indexrelname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY relname;
```

---

## 9. Usuarios, roles y permisos

### Crear roles

```sql
CREATE ROLE lector LOGIN PASSWORD 'clave';
CREATE USER app_user WITH PASSWORD 'clave';   -- USER = ROLE con LOGIN

CREATE ROLE admin_bd WITH
    LOGIN
    PASSWORD 'clave'
    CREATEDB
    CREATEROLE
    VALID UNTIL '2027-01-01';

CREATE ROLE superadmin WITH LOGIN SUPERUSER PASSWORD 'clave';
```

### Modificar y eliminar

```sql
ALTER ROLE app_user WITH PASSWORD 'nueva_clave';
ALTER ROLE app_user WITH NOLOGIN;
ALTER ROLE app_user VALID UNTIL 'infinity';
ALTER ROLE app_user RENAME TO nuevo_nombre;

DROP ROLE app_user;   -- Falla si posee objetos
REASSIGN OWNED BY app_user TO postgres;
DROP OWNED BY app_user;
DROP ROLE app_user;
```

### Permisos

```sql
-- Base de datos y esquema
GRANT CONNECT ON DATABASE mi_base TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;

-- Tablas
GRANT SELECT ON usuarios TO lector;
GRANT SELECT, INSERT, UPDATE, DELETE ON usuarios TO app_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO app_user;

-- Secuencias (necesario si hay columnas SERIAL)
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_user;

-- Permisos para tablas creadas en el futuro
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO lector;

-- Revocar
REVOKE INSERT ON usuarios FROM app_user;
REVOKE ALL ON ALL TABLES IN SCHEMA public FROM app_user;

-- Herencia de roles
GRANT lector TO app_user;
```

### Receta: usuario de solo lectura

```sql
CREATE ROLE reportes LOGIN PASSWORD 'clave';
GRANT CONNECT ON DATABASE mi_base TO reportes;
GRANT USAGE ON SCHEMA public TO reportes;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO reportes;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO reportes;
```

### Inspeccionar

```sql
\du                      -- Listar roles
\dp usuarios             -- Permisos de una tabla
SELECT current_user, session_user;
```

---

## 10. Backup y restauración

### pg_dump (una base de datos)

```bash
# Formato SQL plano
pg_dump -U postgres mi_base > backup.sql

# Formato custom (comprimido, permite restauración selectiva) — RECOMENDADO
pg_dump -U postgres -Fc mi_base > backup.dump

# Formato directorio, en paralelo (rápido para bases grandes)
pg_dump -U postgres -Fd -j 4 mi_base -f backup_dir/

# Solo el esquema (sin datos)
pg_dump -U postgres --schema-only mi_base > schema.sql

# Solo los datos
pg_dump -U postgres --data-only mi_base > data.sql

# Solo tablas específicas
pg_dump -U postgres -t usuarios -t pedidos mi_base > tablas.sql

# Excluir tablas
pg_dump -U postgres -T 'logs*' mi_base > backup.sql

# Comprimido con gzip
pg_dump -U postgres mi_base | gzip > backup.sql.gz
```

### pg_dumpall (todo el clúster)

```bash
pg_dumpall -U postgres > cluster.sql

# Solo roles y permisos globales
pg_dumpall -U postgres --globals-only > roles.sql
```

### Restaurar

```bash
# Desde SQL plano
psql -U postgres -d mi_base < backup.sql
psql -U postgres -d mi_base -f backup.sql
gunzip -c backup.sql.gz | psql -U postgres -d mi_base

# Desde formato custom
pg_restore -U postgres -d mi_base backup.dump

# Restauración en paralelo
pg_restore -U postgres -d mi_base -j 4 backup.dump

# Recrear la base desde cero antes de restaurar
pg_restore -U postgres -d postgres --clean --create backup.dump

# Restaurar solo una tabla
pg_restore -U postgres -d mi_base -t usuarios backup.dump

# Ver el contenido del dump sin restaurar
pg_restore -l backup.dump
```

### Backup automático diario (cron)

```bash
# Script: /usr/local/bin/pg_backup.sh
#!/bin/bash
FECHA=$(date +%Y%m%d_%H%M%S)
DEST=/var/backups/postgres
pg_dump -U postgres -Fc mi_base > "$DEST/mi_base_$FECHA.dump"
find "$DEST" -name "*.dump" -mtime +7 -delete   # Retención de 7 días
```

```bash
# crontab -e  →  todos los días a las 2 AM
0 2 * * * /usr/local/bin/pg_backup.sh
```

> **Regla de oro:** un backup no probado no es un backup. Restáuralo periódicamente en un entorno de prueba.

---

## 11. Importar y exportar datos

### COPY (del lado del servidor, requiere superusuario)

```sql
-- Exportar
COPY usuarios TO '/tmp/usuarios.csv' WITH (FORMAT csv, HEADER true);
COPY (SELECT * FROM usuarios WHERE activo) TO '/tmp/act.csv' CSV HEADER;

-- Importar
COPY usuarios(email, nombre) FROM '/tmp/usuarios.csv' WITH (FORMAT csv, HEADER true);
```

### \copy (del lado del cliente, no requiere permisos especiales)

```sql
\copy usuarios TO 'usuarios.csv' WITH (FORMAT csv, HEADER true)
\copy usuarios(email, nombre) FROM 'datos.csv' WITH (FORMAT csv, HEADER true)

-- Delimitador personalizado
\copy usuarios FROM 'datos.tsv' WITH (FORMAT csv, DELIMITER E'\t', HEADER true)

-- Manejo de nulos
\copy usuarios FROM 'datos.csv' WITH (FORMAT csv, HEADER true, NULL 'NULL')
```

> `\copy` lee y escribe archivos **en tu máquina**; `COPY` lo hace en el servidor. En la mayoría de los casos querrás `\copy`.

### Exportar desde el shell

```bash
# CSV
psql -U postgres -d mi_base -c "\copy (SELECT * FROM usuarios) TO 'out.csv' CSV HEADER"

# Texto plano sin formato
psql -U postgres -d mi_base -A -t -F',' -c "SELECT * FROM usuarios" > out.csv

# JSON
psql -U postgres -d mi_base -t -A -c \
  "SELECT json_agg(row_to_json(u)) FROM usuarios u" > out.json
```

---

## 12. Transacciones

```sql
BEGIN;
    UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
    UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;
COMMIT;

-- Deshacer
BEGIN;
    DELETE FROM usuarios;
ROLLBACK;

-- Puntos de guardado
BEGIN;
    INSERT INTO usuarios (email) VALUES ('a@b.com');
    SAVEPOINT sp1;
    INSERT INTO usuarios (email) VALUES ('malo');
    ROLLBACK TO sp1;    -- Deshace solo lo posterior al savepoint
COMMIT;
```

### Niveles de aislamiento

```sql
BEGIN ISOLATION LEVEL READ COMMITTED;   -- Por defecto
BEGIN ISOLATION LEVEL REPEATABLE READ;
BEGIN ISOLATION LEVEL SERIALIZABLE;     -- El más estricto
```

### Bloqueos explícitos

```sql
SELECT * FROM cuentas WHERE id = 1 FOR UPDATE;         -- Bloquea las filas
SELECT * FROM cuentas WHERE id = 1 FOR UPDATE NOWAIT;  -- Falla si está ocupada
SELECT * FROM cuentas FOR UPDATE SKIP LOCKED;          -- Útil para colas
```

> **Tip de seguridad:** ejecuta psql con `\set AUTOCOMMIT off` en producción para que cada sentencia requiera un `COMMIT` explícito.

---

## 13. Administración del servidor

### Control del servicio

```bash
# systemd (Ubuntu, Debian, RHEL)
sudo systemctl start postgresql
sudo systemctl stop postgresql
sudo systemctl restart postgresql
sudo systemctl reload postgresql     # Recarga configuración sin cortar conexiones
sudo systemctl status postgresql

# pg_ctl (directo)
pg_ctl -D /var/lib/postgresql/data start
pg_ctl -D /var/lib/postgresql/data stop -m fast
pg_ctl -D /var/lib/postgresql/data reload

# macOS con Homebrew
brew services start postgresql@16
brew services restart postgresql@16
```

### Archivos de configuración

| Archivo | Propósito |
|---|---|
| `postgresql.conf` | Configuración general del servidor |
| `pg_hba.conf` | Reglas de autenticación por host |
| `pg_ident.conf` | Mapeo de usuarios del SO a roles |

```sql
-- Encontrar su ubicación
SHOW config_file;
SHOW hba_file;
SHOW data_directory;
```

### Parámetros comunes

```sql
SHOW ALL;
SHOW max_connections;
SHOW shared_buffers;
SHOW work_mem;

-- Cambiar en caliente (persiste, requiere reload o restart según el parámetro)
ALTER SYSTEM SET max_connections = 200;
SELECT pg_reload_conf();

-- Solo para la sesión actual
SET work_mem = '256MB';
```

| Parámetro | Descripción | Valor típico |
|---|---|---|
| `max_connections` | Conexiones simultáneas | 100–200 |
| `shared_buffers` | Caché de PostgreSQL | 25% de la RAM |
| `effective_cache_size` | Estimación de caché total del SO | 50–75% de la RAM |
| `work_mem` | Memoria por operación de orden/hash | 4–64 MB |
| `maintenance_work_mem` | Memoria para VACUUM, CREATE INDEX | 512 MB – 2 GB |

### pg_hba.conf

```
# TIPO   BD      USUARIO   DIRECCIÓN        MÉTODO
local    all     all                        peer
host     all     all       127.0.0.1/32     scram-sha-256
host     mi_base app_user  10.0.0.0/24      scram-sha-256
hostssl  all     all       0.0.0.0/0        scram-sha-256
```

Tras editarlo: `sudo systemctl reload postgresql`.

### Monitoreo

```sql
SELECT version();
SELECT pg_postmaster_start_time();
SELECT count(*) FROM pg_stat_activity;                    -- Conexiones activas

-- Actividad por base de datos
SELECT datname, numbackends, xact_commit, xact_rollback, blks_hit, blks_read
FROM pg_stat_database;

-- Tasa de aciertos de caché (idealmente > 0.99)
SELECT sum(blks_hit)::float / nullif(sum(blks_hit) + sum(blks_read), 0)
FROM pg_stat_database;

-- Estado de la replicación
SELECT * FROM pg_stat_replication;
```

### Logs

```bash
# Ubicaciones habituales
/var/log/postgresql/postgresql-16-main.log     # Debian/Ubuntu
/var/lib/pgsql/16/data/log/                    # RHEL/CentOS

tail -f /var/log/postgresql/postgresql-16-main.log
journalctl -u postgresql -f
```

---

## 14. Configuración de psql (.psqlrc)

Crea `~/.psqlrc` para tener psql configurado a tu gusto en cada sesión:

```sql
-- Silenciar la salida de este archivo al iniciar
\set QUIET 1

-- Formato
\x auto
\timing on
\pset null '∅'
\pset linestyle unicode
\pset border 2

-- Historial: uno por base de datos, más largo
\set HISTFILE ~/.psql_history-:DBNAME
\set HISTSIZE 5000
\set HISTCONTROL ignoredups

-- Prompt informativo: usuario@basededatos
\set PROMPT1 '%[%033[1;32m%]%n@%/%[%033[0m%]%R%# '
\set PROMPT2 '  ... %R%# '

-- Autocompletado en mayúsculas
\set COMP_KEYWORD_CASE upper

-- Mostrar errores con detalle
\set VERBOSITY verbose

-- Atajos personalizados
\set version 'SELECT version();'
\set actividad 'SELECT pid, now() - query_start AS duracion, state, query FROM pg_stat_activity WHERE state <> \'idle\' ORDER BY duracion DESC;'
\set tamanos 'SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) FROM pg_catalog.pg_statio_user_tables ORDER BY pg_total_relation_size(relid) DESC LIMIT 20;'

\unset QUIET
```

Uso de los atajos: escribe `:actividad` y presiona Enter.

### Atajos de teclado en psql

| Atajo | Acción |
|---|---|
| `Ctrl+R` | Buscar en el historial |
| `Ctrl+A` / `Ctrl+E` | Inicio / fin de línea |
| `Ctrl+U` | Borrar la línea |
| `Ctrl+L` | Limpiar pantalla |
| `Ctrl+C` | Cancelar la consulta actual |
| `Ctrl+D` | Salir |
| `Tab` | Autocompletar |

---

## 15. Solución de problemas

### `psql: could not connect to server`

El servidor no está corriendo o no escucha donde crees.

```bash
sudo systemctl status postgresql
sudo ss -lntp | grep 5432
```

Verifica en `postgresql.conf` que `listen_addresses = '*'` (o la IP correcta) y `port = 5432`.

### `FATAL: role "usuario" does not exist`

```bash
sudo -u postgres createuser --interactive
sudo -u postgres psql -c "CREATE ROLE usuario LOGIN PASSWORD 'clave';"
```

### `FATAL: Peer authentication failed`

El método `peer` exige que el usuario del sistema coincida con el rol.

```bash
sudo -u postgres psql          # Solución rápida
psql -U usuario -h localhost   # Fuerza TCP, que usa scram-sha-256
```

O cambia `peer` por `scram-sha-256` en `pg_hba.conf` y recarga.

### `database "x" is being accessed by other users`

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity
WHERE datname = 'x' AND pid <> pg_backend_pid();
```

### `too many connections`

```sql
SELECT count(*) FROM pg_stat_activity;
ALTER SYSTEM SET max_connections = 200;
-- Requiere reiniciar. Mejor solución de fondo: un pooler como PgBouncer.
```

### `permission denied for table`

```sql
GRANT SELECT ON tabla TO usuario;
GRANT USAGE ON SCHEMA public TO usuario;
```

### `disk full` / la base crece sin control

```sql
VACUUM FULL;   -- Bloquea la tabla; usar en ventana de mantenimiento

-- Detectar bloat por filas muertas
SELECT relname, n_dead_tup, last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC LIMIT 20;
```

### Consulta que nunca termina

```sql
SELECT pid, now() - query_start AS duracion, query
FROM pg_stat_activity WHERE state = 'active' ORDER BY duracion DESC;

SELECT pg_cancel_backend(PID);
SELECT pg_terminate_backend(PID);   -- Si cancel no funciona
```

### Olvidé la contraseña de postgres

1. Edita `pg_hba.conf` y cambia el método a `trust` para conexiones locales.
2. `sudo systemctl reload postgresql`
3. `psql -U postgres -c "ALTER USER postgres PASSWORD 'nueva';"`
4. Revierte `pg_hba.conf` a `scram-sha-256` y recarga.

---

## Recursos

- Documentación oficial: <https://www.postgresql.org/docs/current/>
- Referencia de psql: <https://www.postgresql.org/docs/current/app-psql.html>
- Wiki de la comunidad: <https://wiki.postgresql.org/>
- Explicador de planes: <https://explain.dalibo.com/>
