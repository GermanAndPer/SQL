# 🐬 Referencia Completa de MySQL
> MySQL 8.0+ / MariaDB 10.6+
> Organizada por categorías e importancia de uso

---

## 📊 Escala de Importancia

| Nivel | Descripción |
|-------|-------------|
| 🔴 **Crítico** | Uso diario, esencial en cualquier proyecto |
| 🟠 **Alto** | Frecuente en desarrollo real y administración |
| 🟡 **Medio** | Situaciones específicas o avanzadas |
| 🟢 **Avanzado** | DBA, tuning, replicación, alta disponibilidad |

---

## 1. ⚙️ Configuración y Entorno

```sql
-- Versión del servidor
SELECT VERSION();
SELECT @@version;

-- Variables de sistema
SHOW VARIABLES;
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE '%charset%';
SHOW VARIABLES LIKE '%innodb%';

-- Estado del servidor
SHOW STATUS;
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Queries';

-- Configuración de sesión
SET SESSION sql_mode = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION';
SET NAMES 'utf8mb4';                    -- Charset de la conexión
SET time_zone = 'America/Mexico_City';  -- Zona horaria de sesión
SET FOREIGN_KEY_CHECKS = 0;            -- ⚠️ Deshabilitar FK temporalmente
SET FOREIGN_KEY_CHECKS = 1;            -- Volver a habilitar

-- Variables globales (requiere SUPER o SYSTEM_VARIABLES_ADMIN)
SET GLOBAL max_connections   = 200;
SET GLOBAL slow_query_log    = ON;

-- Información del servidor
SELECT @@hostname, @@datadir, @@basedir, @@port;
SELECT DATABASE() AS BaseDatosActual;
SELECT USER()     AS UsuarioActual;
SELECT NOW()      AS FechaHoraActual;
```

---

## 2. 🗂️ DDL — Definición de Datos

### 2.1 Bases de Datos

| Importancia | Comando | Descripción |
|-------------|---------|-------------|
| 🔴 | `CREATE DATABASE nombre` | Crea una base de datos |
| 🔴 | `DROP DATABASE nombre` | Elimina una base de datos |
| 🔴 | `USE nombre` | Selecciona la base de datos activa |
| 🟠 | `SHOW DATABASES` | Lista todas las bases de datos |
| 🟠 | `SHOW CREATE DATABASE nombre` | Muestra el DDL de la BD |
| 🟡 | `ALTER DATABASE nombre ...` | Modifica propiedades de la BD |

```sql
-- Crear BD con charset y collation (recomendado MySQL 8+)
CREATE DATABASE tienda
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- Crear solo si no existe
CREATE DATABASE IF NOT EXISTS tienda
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- Modificar charset de una BD existente
ALTER DATABASE tienda
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;   -- MySQL 8 default

-- Ver todas las BDs con sus propiedades
SELECT schema_name, default_character_set_name, default_collation_name
FROM information_schema.schemata;

-- Eliminar si existe
DROP DATABASE IF EXISTS tienda;
```

---

### 2.2 Motores de Almacenamiento (Storage Engines)

| Motor | Transacciones | FK | Bloqueo | Uso recomendado |
|-------|--------------|-----|---------|-----------------|
| **InnoDB** ✅ | ✅ Sí | ✅ Sí | Fila | Producción general — **usar siempre** |
| **MyISAM** | ❌ No | ❌ No | Tabla | Legacy, lectura masiva (obsoleto) |
| **MEMORY** | ❌ No | ❌ No | Tabla | Tablas temporales en RAM |
| **ARCHIVE** | ❌ No | ❌ No | Fila | Logs históricos, solo lectura |
| **CSV** | ❌ No | ❌ No | Tabla | Intercambio de datos planos |

```sql
-- Ver engines disponibles
SHOW ENGINES;

-- Ver engine de cada tabla
SELECT table_name, engine
FROM information_schema.tables
WHERE table_schema = DATABASE();

-- Cambiar motor de una tabla
ALTER TABLE mi_tabla ENGINE = InnoDB;
```

---

### 2.3 Tablas

| Importancia | Comando | Descripción |
|-------------|---------|-------------|
| 🔴 | `CREATE TABLE` | Crea una tabla |
| 🔴 | `ALTER TABLE` | Modifica estructura |
| 🔴 | `DROP TABLE` | Elimina tabla |
| 🟠 | `TRUNCATE TABLE` | Vacía la tabla y reinicia AUTO_INCREMENT |
| 🟠 | `RENAME TABLE vieja TO nueva` | Renombra tabla |
| 🟠 | `CREATE TABLE nueva SELECT * FROM vieja` | Copia estructura + datos (sin constraints) |
| 🟡 | `CREATE TABLE nueva LIKE vieja` | Copia solo la estructura |
| 🟡 | `CREATE TEMPORARY TABLE` | Tabla temporal de sesión |
| 🟡 | `SHOW TABLES` | Lista tablas de la BD actual |
| 🟡 | `SHOW CREATE TABLE nombre` | Muestra DDL completo |
| 🟡 | `DESCRIBE nombre` / `DESC nombre` | Muestra columnas y tipos |

```sql
-- Creación completa de tabla
CREATE TABLE IF NOT EXISTS productos (
    producto_id     INT             NOT NULL AUTO_INCREMENT,
    codigo          VARCHAR(20)     NOT NULL,
    nombre          VARCHAR(200)    NOT NULL,
    descripcion     TEXT            NULL,
    precio          DECIMAL(12,2)   NOT NULL DEFAULT 0.00,
    stock           INT UNSIGNED    NOT NULL DEFAULT 0,
    activo          TINYINT(1)      NOT NULL DEFAULT 1,
    creado_en       DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP,
    modificado_en   DATETIME        NULL ON UPDATE CURRENT_TIMESTAMP,
    categoria_id    INT             NULL,

    CONSTRAINT pk_productos         PRIMARY KEY (producto_id),
    CONSTRAINT uq_productos_codigo  UNIQUE KEY  (codigo),
    CONSTRAINT fk_productos_cat     FOREIGN KEY (categoria_id)
                                    REFERENCES categorias(categoria_id)
                                    ON DELETE SET NULL
                                    ON UPDATE CASCADE,
    CONSTRAINT ck_precio            CHECK (precio >= 0),

    INDEX idx_categoria (categoria_id),
    INDEX idx_activo_precio (activo, precio)
)
ENGINE = InnoDB
DEFAULT CHARSET = utf8mb4
COLLATE = utf8mb4_unicode_ci
COMMENT = 'Catálogo de productos';

-- Tabla temporal (existe solo en la sesión actual)
CREATE TEMPORARY TABLE tmp_resultados (
    id    INT,
    valor DECIMAL(10,2)
);
```

---

### 2.4 Tipos de Datos

| Categoría | Tipo | Descripción | Rango / Notas |
|-----------|------|-------------|---------------|
| **Enteros** | `TINYINT` | 1 byte | -128/127 · UNSIGNED: 0-255 |
| | `SMALLINT` | 2 bytes | -32,768/32,767 |
| | `MEDIUMINT` | 3 bytes | -8.3M/8.3M |
| | `INT` / `INTEGER` | 4 bytes | -2.1B/2.1B |
| | `BIGINT` | 8 bytes | ±9.2 × 10¹⁸ |
| | `BOOLEAN` / `TINYINT(1)` | Booleano | 0 / 1 |
| **Decimales** | `DECIMAL(p,s)` / `NUMERIC` | Exacto | Hasta 65 dígitos ✅ Dinero |
| | `FLOAT` | Simple precisión | ~7 dígitos significativos |
| | `DOUBLE` | Doble precisión | ~15 dígitos significativos |
| **Texto** | `CHAR(n)` | Fijo (0-255) | Rellena con espacios |
| | `VARCHAR(n)` | Variable (0-65,535) | ✅ Más usado |
| | `TINYTEXT` | Hasta 255 bytes | |
| | `TEXT` | Hasta 64 KB | |
| | `MEDIUMTEXT` | Hasta 16 MB | |
| | `LONGTEXT` | Hasta 4 GB | |
| | `ENUM('a','b')` | Un valor de lista | Hasta 65,535 opciones |
| | `SET('a','b')` | Varios valores de lista | |
| **Fecha/Hora** | `DATE` | Solo fecha | 1000-01-01 – 9999-12-31 |
| | `TIME` | Solo hora | -838:59:59 – 838:59:59 |
| | `DATETIME` | Fecha + hora | 1000-01-01 – 9999-12-31 |
| | `TIMESTAMP` | Fecha + hora UTC | 1970-2038 ⚠️ Y2K38 |
| | `YEAR` | Solo año | 1901-2155 |
| **Binario** | `TINYBLOB`/`BLOB`/`MEDIUMBLOB`/`LONGBLOB` | Binario | Hasta 4 GB |
| | `BINARY(n)` / `VARBINARY(n)` | Binario fijo/variable | |
| **Especiales** | `JSON` | Documento JSON nativo | MySQL 5.7.8+ |
| | `GEOMETRY` | Datos espaciales | |

```sql
-- AUTO_INCREMENT
CREATE TABLE categorias (
    categoria_id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    nombre       VARCHAR(100) NOT NULL
);

-- Reiniciar contador
ALTER TABLE categorias AUTO_INCREMENT = 1;

-- Obtener último ID insertado
SELECT LAST_INSERT_ID();

-- UNSIGNED: solo positivos, duplica rango máximo
stock INT UNSIGNED NOT NULL DEFAULT 0,   -- 0 a 4,294,967,295
```

---

### 2.5 ALTER TABLE — Modificar Tablas

```sql
-- Agregar columna al final
ALTER TABLE productos ADD COLUMN peso_kg DECIMAL(8,3) NULL;

-- Agregar en posición específica
ALTER TABLE productos ADD COLUMN peso_kg DECIMAL(8,3) NULL AFTER precio;
ALTER TABLE productos ADD COLUMN sku VARCHAR(50) NOT NULL FIRST;

-- Modificar tipo de columna (conserva nombre)
ALTER TABLE productos MODIFY COLUMN nombre VARCHAR(500) NOT NULL;

-- Modificar nombre y tipo (CHANGE)
ALTER TABLE productos CHANGE COLUMN nombre nombre_producto VARCHAR(500) NOT NULL;

-- Renombrar columna (MySQL 8.0+)
ALTER TABLE productos RENAME COLUMN nombre TO nombre_producto;

-- Eliminar columna
ALTER TABLE productos DROP COLUMN peso_kg;

-- Renombrar tabla
RENAME TABLE productos TO articulos;
-- O mover entre bases de datos:
RENAME TABLE db1.tabla TO db2.tabla;

-- Agregar / eliminar índices
ALTER TABLE productos ADD INDEX idx_nombre (nombre);
ALTER TABLE productos ADD UNIQUE INDEX uq_sku (sku);
ALTER TABLE productos DROP INDEX idx_nombre;

-- Agregar / eliminar FK
ALTER TABLE productos
  ADD CONSTRAINT fk_proveedor
  FOREIGN KEY (proveedor_id) REFERENCES proveedores(proveedor_id)
  ON DELETE RESTRICT ON UPDATE CASCADE;

ALTER TABLE productos DROP FOREIGN KEY fk_proveedor;

-- Cambiar charset de la tabla
ALTER TABLE productos CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Múltiples cambios en una sola sentencia (más eficiente)
ALTER TABLE productos
    ADD COLUMN tags      JSON NULL,
    ADD COLUMN vistas    INT UNSIGNED NOT NULL DEFAULT 0,
    ADD INDEX idx_vistas (vistas),
    MODIFY COLUMN precio DECIMAL(14,2) NOT NULL DEFAULT 0.00;
```

---

## 3. 🔍 DQL — Consulta de Datos

### 3.1 SELECT Fundamental

```sql
-- Estructura completa de un SELECT
SELECT [DISTINCT | ALL]
    columnas | expresiones | *
FROM tabla [AS alias]
    [JOIN ...]
[WHERE condicion]
[GROUP BY columnas [WITH ROLLUP]]
[HAVING condicion_de_grupo]
[ORDER BY columnas [ASC | DESC]]
[LIMIT n [OFFSET m]];
```

```sql
-- Ejemplo completo
SELECT
    p.producto_id,
    p.nombre,
    p.precio,
    p.precio * 1.16                      AS precio_con_iva,
    c.nombre                             AS categoria,
    IFNULL(p.stock, 0)                   AS stock,
    DATE_FORMAT(p.creado_en, '%d/%m/%Y') AS fecha_alta,
    CASE
        WHEN p.precio < 500  THEN 'Económico'
        WHEN p.precio < 2000 THEN 'Medio'
        ELSE 'Premium'
    END                                  AS rango_precio
FROM productos p
INNER JOIN categorias c ON p.categoria_id = c.categoria_id
WHERE p.activo = 1
  AND p.stock > 0
ORDER BY p.precio DESC
LIMIT 50;
```

---

### 3.2 Paginación — LIMIT y OFFSET

```sql
-- Básico
SELECT * FROM productos ORDER BY nombre LIMIT 10;

-- Paginación: LIMIT cantidad OFFSET inicio
SELECT * FROM productos ORDER BY nombre LIMIT 10 OFFSET 20;  -- Página 3

-- Sintaxis corta equivalente: LIMIT inicio, cantidad
SELECT * FROM productos ORDER BY nombre LIMIT 20, 10;

-- Con variables de sesión
SET @pagina     = 3;
SET @tamano_pag = 10;

SELECT * FROM productos ORDER BY nombre
LIMIT 10 OFFSET ((@pagina - 1) * @tamano_pag);

-- Contar total real ignorando el LIMIT (para paginación frontend)
SELECT SQL_CALC_FOUND_ROWS producto_id, nombre
FROM productos WHERE activo = 1
LIMIT 10 OFFSET 20;

SELECT FOUND_ROWS() AS total_sin_limit;
```

---

### 3.3 Filtros — WHERE

| Importancia | Operador | Ejemplo |
|-------------|----------|---------|
| 🔴 | `=`, `<>`, `!=`, `<`, `>`, `<=`, `>=` | `WHERE precio > 100` |
| 🔴 | `AND`, `OR`, `NOT` | `WHERE activo = 1 AND stock > 0` |
| 🔴 | `IS NULL` / `IS NOT NULL` | `WHERE descripcion IS NULL` |
| 🔴 | `IN (...)` / `NOT IN (...)` | `WHERE categoria_id IN (1, 3, 5)` |
| 🔴 | `BETWEEN ... AND ...` | `WHERE precio BETWEEN 100 AND 999` |
| 🔴 | `LIKE` / `NOT LIKE` | `WHERE nombre LIKE 'Lap%'` |
| 🟠 | `REGEXP` / `RLIKE` | `WHERE nombre REGEXP '^[A-Z]'` |
| 🟠 | `EXISTS (subconsulta)` | `WHERE EXISTS (SELECT 1 FROM ...)` |
| 🟡 | `SOUNDS LIKE` | Búsqueda fonética (Soundex) |
| 🟡 | `MATCH() AGAINST()` | Búsqueda full-text |

```sql
-- LIKE: comodines
WHERE nombre LIKE 'A%'       -- Empieza con A
WHERE nombre LIKE '%pro%'    -- Contiene "pro"
WHERE nombre LIKE '_ata%'    -- Cualquier char + "ata" + cualquier cosa

-- REGEXP: expresiones regulares
WHERE nombre  REGEXP '^[A-Z]'         -- Empieza con mayúscula
WHERE email   REGEXP '@gmail\\.com$'  -- Termina en @gmail.com
WHERE codigo  REGEXP '^[0-9]{4}$'     -- Exactamente 4 dígitos

-- Búsqueda FULLTEXT
ALTER TABLE productos ADD FULLTEXT INDEX ft_idx (nombre, descripcion);

SELECT *, MATCH(nombre, descripcion) AGAINST('laptop gaming' IN BOOLEAN MODE) AS relevancia
FROM productos
WHERE MATCH(nombre, descripcion) AGAINST('laptop gaming' IN BOOLEAN MODE)
ORDER BY relevancia DESC;
```

---

### 3.4 JOINs

| Importancia | Tipo | Descripción |
|-------------|------|-------------|
| 🔴 | `INNER JOIN` | Solo coincidencias en ambas tablas |
| 🔴 | `LEFT JOIN` | Todas las filas izquierda + coincidencias |
| 🟠 | `RIGHT JOIN` | Todas las filas derecha + coincidencias |
| 🟠 | `CROSS JOIN` | Producto cartesiano |
| 🟡 | `SELF JOIN` | La tabla unida consigo misma |

> ⚠️ MySQL **no tiene** `FULL OUTER JOIN` nativo — se simula con `UNION`

```sql
-- INNER JOIN
SELECT p.nombre, c.nombre AS categoria
FROM productos p
INNER JOIN categorias c ON p.categoria_id = c.categoria_id;

-- LEFT JOIN: todos los productos, con o sin categoría
SELECT p.nombre, IFNULL(c.nombre, 'Sin categoría') AS categoria
FROM productos p
LEFT JOIN categorias c ON p.categoria_id = c.categoria_id;

-- Simular FULL OUTER JOIN con UNION
SELECT p.nombre, c.nombre AS categoria
FROM productos p LEFT JOIN categorias c ON p.categoria_id = c.categoria_id
UNION
SELECT p.nombre, c.nombre AS categoria
FROM productos p RIGHT JOIN categorias c ON p.categoria_id = c.categoria_id;

-- JOIN múltiple
SELECT p.nombre, c.nombre AS categoria, pr.nombre AS proveedor
FROM productos p
INNER JOIN categorias  c  ON p.categoria_id = c.categoria_id
INNER JOIN proveedores pr ON p.proveedor_id = pr.proveedor_id;

-- SELF JOIN: empleados con su jefe
SELECT e.nombre AS empleado, j.nombre AS jefe
FROM empleados e
LEFT JOIN empleados j ON e.jefe_id = j.empleado_id;
```

---

### 3.5 Agregación y GROUP BY

```sql
-- Resumen por categoría
SELECT
    c.nombre                        AS categoria,
    COUNT(p.producto_id)            AS total_productos,
    SUM(p.stock)                    AS stock_total,
    AVG(p.precio)                   AS precio_promedio,
    MIN(p.precio)                   AS precio_minimo,
    MAX(p.precio)                   AS precio_maximo,
    SUM(p.precio * p.stock)         AS valor_inventario
FROM categorias c
LEFT JOIN productos p ON p.categoria_id = c.categoria_id
WHERE p.activo = 1
GROUP BY c.categoria_id, c.nombre
HAVING COUNT(p.producto_id) >= 5
ORDER BY valor_inventario DESC;

-- ROLLUP: subtotales y total general (MySQL nativo)
SELECT
    IFNULL(c.nombre, 'TOTAL GENERAL') AS categoria,
    SUM(p.stock)                       AS stock_total
FROM productos p
JOIN categorias c ON p.categoria_id = c.categoria_id
GROUP BY c.nombre WITH ROLLUP;

-- GROUP_CONCAT: concatenar valores de un grupo en una cadena
SELECT
    c.nombre AS categoria,
    GROUP_CONCAT(p.nombre ORDER BY p.nombre SEPARATOR ', ') AS productos,
    COUNT(p.producto_id) AS total
FROM categorias c
JOIN productos p ON p.categoria_id = c.categoria_id
GROUP BY c.categoria_id, c.nombre;

-- Aumentar límite de GROUP_CONCAT (default 1024 bytes)
SET SESSION group_concat_max_len = 1000000;
```

---

### 3.6 CTEs y Subconsultas (MySQL 8.0+)

```sql
-- CTE simple
WITH ventas_mensuales AS (
    SELECT
        YEAR(fecha)  AS anio,
        MONTH(fecha) AS mes,
        SUM(total)   AS ingresos
    FROM pedidos
    GROUP BY YEAR(fecha), MONTH(fecha)
)
SELECT anio, mes, ingresos,
       SUM(ingresos) OVER (PARTITION BY anio ORDER BY mes) AS acumulado_anual
FROM ventas_mensuales
ORDER BY anio, mes;

-- CTEs múltiples
WITH
activos AS (SELECT * FROM productos WHERE activo = 1),
caros   AS (SELECT * FROM activos    WHERE precio > 5000)
SELECT * FROM caros ORDER BY precio DESC;

-- CTE Recursiva: jerarquía organizacional
WITH RECURSIVE jerarquia AS (
    -- Caso base: sin jefe
    SELECT empleado_id, nombre, jefe_id, 0 AS nivel,
           CAST(nombre AS CHAR(500)) AS ruta
    FROM empleados WHERE jefe_id IS NULL

    UNION ALL

    -- Caso recursivo
    SELECT e.empleado_id, e.nombre, e.jefe_id, j.nivel + 1,
           CONCAT(j.ruta, ' > ', e.nombre)
    FROM empleados e
    INNER JOIN jerarquia j ON e.jefe_id = j.empleado_id
)
SELECT nivel, nombre, ruta FROM jerarquia ORDER BY ruta;

-- Subconsulta correlacionada
SELECT p.nombre, p.precio,
       (SELECT AVG(precio) FROM productos WHERE categoria_id = p.categoria_id) AS avg_cat
FROM productos p;
```

---

### 3.7 Window Functions (MySQL 8.0+)

| Importancia | Función | Descripción |
|-------------|---------|-------------|
| 🟠 | `ROW_NUMBER()` | Número de fila único por partición |
| 🟠 | `RANK()` | Posición con huecos en empates |
| 🟠 | `DENSE_RANK()` | Posición sin huecos en empates |
| 🟠 | `SUM() OVER()` | Suma acumulada o por partición |
| 🟠 | `AVG() OVER()` | Promedio por ventana |
| 🟡 | `LAG(col, n, default)` | Valor de N filas anteriores |
| 🟡 | `LEAD(col, n, default)` | Valor de N filas siguientes |
| 🟡 | `FIRST_VALUE()` / `LAST_VALUE()` | Primer / último valor |
| 🟡 | `NTILE(n)` | Divide en N grupos iguales |
| 🟡 | `PERCENT_RANK()` | Rango como porcentaje (0.0 – 1.0) |
| 🟡 | `CUME_DIST()` | Distribución acumulada |

```sql
-- Análisis completo por categoría
SELECT
    nombre,
    categoria_id,
    precio,
    ROW_NUMBER()     OVER (PARTITION BY categoria_id ORDER BY precio DESC) AS posicion,
    RANK()           OVER (PARTITION BY categoria_id ORDER BY precio DESC) AS ranking,
    DENSE_RANK()     OVER (PARTITION BY categoria_id ORDER BY precio DESC) AS ranking_denso,
    NTILE(4)         OVER (PARTITION BY categoria_id ORDER BY precio DESC) AS cuartil,
    SUM(precio)      OVER (PARTITION BY categoria_id)                      AS total_cat,
    AVG(precio)      OVER (PARTITION BY categoria_id)                      AS prom_cat,
    LAG(precio,1,0)  OVER (PARTITION BY categoria_id ORDER BY precio)      AS precio_prev,
    LEAD(precio,1,0) OVER (PARTITION BY categoria_id ORDER BY precio)      AS precio_sig
FROM productos WHERE activo = 1;

-- Eliminar duplicados con ROW_NUMBER
DELETE FROM productos
WHERE producto_id IN (
    SELECT producto_id FROM (
        SELECT producto_id,
               ROW_NUMBER() OVER (PARTITION BY codigo ORDER BY creado_en DESC) AS rn
        FROM productos
    ) t
    WHERE rn > 1
);
```

---

## 4. ✏️ DML — Manipulación de Datos

### 4.1 INSERT

```sql
-- Insert simple
INSERT INTO productos (nombre, precio, stock, categoria_id)
VALUES ('Laptop Dell XPS', 25999.99, 15, 3);

-- Insert múltiple (más eficiente que múltiples INSERTs)
INSERT INTO productos (nombre, precio, stock) VALUES
    ('Mouse Logitech',   349.00, 100),
    ('Teclado Mecánico', 899.00,  75),
    ('Monitor 4K',      6499.00,  20);

-- Insert desde SELECT
INSERT INTO productos_archivo (nombre, precio, archivado_en)
SELECT nombre, precio, NOW()
FROM productos WHERE activo = 0;

-- INSERT IGNORE: ignora errores de duplicados silenciosamente
INSERT IGNORE INTO categorias (categoria_id, nombre)
VALUES (1, 'Electrónica');

-- ON DUPLICATE KEY UPDATE (Upsert)
INSERT INTO productos (producto_id, nombre, precio, stock)
VALUES (1, 'Laptop Pro', 17999.99, 20)
ON DUPLICATE KEY UPDATE
    precio        = VALUES(precio),
    stock         = VALUES(stock),
    modificado_en = NOW();

-- MySQL 8.0.20+: usar alias de fila en lugar de VALUES()
INSERT INTO productos (producto_id, nombre, precio) VALUES (1, 'Laptop Pro', 17999.99) AS nuevo
ON DUPLICATE KEY UPDATE precio = nuevo.precio, nombre = nuevo.nombre;

-- REPLACE INTO: elimina + inserta si existe (⚠️ borra la fila completa)
REPLACE INTO productos (producto_id, nombre, precio)
VALUES (1, 'Laptop Pro', 17999.99);

-- Obtener ID del último insert
SELECT LAST_INSERT_ID();
```

---

### 4.2 UPDATE

```sql
-- Update simple (⚠️ SIEMPRE usar WHERE)
UPDATE productos
SET precio        = precio * 1.10,
    modificado_en = NOW()
WHERE categoria_id = 3 AND activo = 1;

-- Update con JOIN (sintaxis propia de MySQL)
UPDATE productos p
INNER JOIN categorias c ON p.categoria_id = c.categoria_id
SET p.precio       = p.precio * 1.05,
    p.modificado_en = NOW()
WHERE c.nombre = 'Electrónica' AND p.activo = 1;

-- Update con CASE
UPDATE productos
SET descuento = CASE
    WHEN precio > 10000 THEN 0.15
    WHEN precio > 5000  THEN 0.10
    WHEN precio > 1000  THEN 0.05
    ELSE 0
END
WHERE activo = 1;

-- Update con LIMIT (útil para actualizar por lotes)
UPDATE productos SET vistas = vistas + 1
WHERE activo = 1
ORDER BY vistas ASC
LIMIT 100;

-- Update con subconsulta (MySQL requiere subquery derivada)
UPDATE productos p
SET precio = (
    SELECT promedio FROM (
        SELECT categoria_id, AVG(precio) AS promedio
        FROM productos GROUP BY categoria_id
    ) AS sub
    WHERE sub.categoria_id = p.categoria_id
) * 0.95
WHERE activo = 0;
```

---

### 4.3 DELETE y TRUNCATE

```sql
-- Delete con condición (⚠️ SIEMPRE usar WHERE)
DELETE FROM productos WHERE activo = 0 AND stock = 0;

-- Delete con JOIN
DELETE p
FROM productos p
INNER JOIN categorias c ON p.categoria_id = c.categoria_id
WHERE c.activo = 0;

-- Delete con LIMIT (borrar por lotes — mejor práctica para tablas grandes)
DELETE FROM logs
WHERE fecha < DATE_SUB(NOW(), INTERVAL 90 DAY)
ORDER BY fecha ASC
LIMIT 10000;
-- Repetir hasta que ROW_COUNT() = 0

-- Delete con subquery (MySQL no permite borrar de la misma tabla directamente)
DELETE FROM productos
WHERE producto_id IN (
    SELECT producto_id FROM (
        SELECT producto_id FROM productos
        WHERE precio < 1 AND creado_en < DATE_SUB(NOW(), INTERVAL 1 YEAR)
    ) AS tmp
);

-- TRUNCATE: vacía tabla y reinicia AUTO_INCREMENT
TRUNCATE TABLE logs_temporales;
-- ⚠️ No activa triggers, no se puede revertir con ROLLBACK (DDL implícito)

-- Filas afectadas por el último DML
SELECT ROW_COUNT();
```

---

## 5. 🔄 Transacciones (TCL)

```sql
-- Transacción básica (solo con InnoDB)
START TRANSACTION;
    UPDATE cuentas SET saldo = saldo - 1000 WHERE cuenta_id = 1;
    UPDATE cuentas SET saldo = saldo + 1000 WHERE cuenta_id = 2;
COMMIT;

-- Rollback si hay error
START TRANSACTION;
    INSERT INTO pedidos (cliente_id, total) VALUES (42, 1500.00);
    -- Si algo falla:
ROLLBACK;

-- SAVEPOINT: punto de retorno parcial
START TRANSACTION;
    INSERT INTO pedidos (cliente_id, total) VALUES (42, 1500.00);
    SAVEPOINT sp_pedido;

    INSERT INTO pedido_detalle (pedido_id, producto_id, cantidad)
    VALUES (LAST_INSERT_ID(), 7, 2);
    -- Si falla el detalle:
    ROLLBACK TO SAVEPOINT sp_pedido;
    -- O confirmar todo:
COMMIT;

RELEASE SAVEPOINT sp_pedido;  -- Liberar savepoint sin rollback

-- Autocommit
SET autocommit = 0;  -- Deshabilitar (iniciar transacción implícita)
SET autocommit = 1;  -- Volver a habilitar

-- Niveles de aislamiento
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;  -- Default MySQL
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Ver nivel actual
SELECT @@transaction_isolation;
```

| Nivel | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|---------------------|--------------|
| `READ UNCOMMITTED` | ✅ Posible | ✅ Posible | ✅ Posible |
| `READ COMMITTED` | ❌ No | ✅ Posible | ✅ Posible |
| `REPEATABLE READ` | ❌ No | ❌ No | ⚠️ Parcial (InnoDB usa MVCC) |
| `SERIALIZABLE` | ❌ No | ❌ No | ❌ No |

---

## 6. ⚡ Índices y Rendimiento

### 6.1 Tipos de Índices

| Importancia | Tipo | Descripción |
|-------------|------|-------------|
| 🔴 | `PRIMARY KEY` | Índice clustered único, no nulo |
| 🔴 | `INDEX` / `KEY` | Nonclustered estándar |
| 🟠 | `UNIQUE INDEX` | Garantiza unicidad de valores |
| 🟠 | Índice compuesto | Sobre múltiples columnas |
| 🟡 | `FULLTEXT INDEX` | Búsqueda de texto completo |
| 🟡 | `SPATIAL INDEX` | Datos geoespaciales |
| 🟢 | `INVISIBLE INDEX` | Ignorado por el optimizador (MySQL 8+) |
| 🟢 | `DESCENDING INDEX` | Orden descendente explícito (MySQL 8+) |

```sql
-- Crear índices
CREATE INDEX idx_categoria  ON productos(categoria_id);
CREATE UNIQUE INDEX uq_email ON clientes(email);
CREATE FULLTEXT INDEX ft_idx ON productos(nombre, descripcion);

-- Índice compuesto (el orden de columnas importa)
CREATE INDEX idx_activo_precio ON productos(activo, precio DESC);
CREATE INDEX idx_cliente_fecha ON pedidos(cliente_id, fecha DESC);

-- Prefijo de columna para TEXT/VARCHAR grandes
ALTER TABLE productos ADD INDEX idx_nombre (nombre(50));

-- Índice invisible (MySQL 8+)
ALTER TABLE productos ALTER INDEX idx_nombre INVISIBLE;
ALTER TABLE productos ALTER INDEX idx_nombre VISIBLE;

-- Ver índices de una tabla
SHOW INDEX FROM productos;
SHOW INDEXES FROM productos\G

-- Eliminar índice
DROP INDEX idx_categoria ON productos;
ALTER TABLE productos DROP INDEX idx_categoria;

-- Forzar/ignorar un índice en una consulta
SELECT * FROM productos USE INDEX (idx_categoria) WHERE categoria_id = 3;
SELECT * FROM productos IGNORE INDEX (idx_nombre) WHERE nombre LIKE 'L%';
SELECT * FROM productos FORCE INDEX (idx_activo_precio) WHERE activo = 1;
```

---

### 6.2 EXPLAIN y Diagnóstico

```sql
-- Plan de ejecución básico
EXPLAIN SELECT * FROM productos WHERE categoria_id = 3;

-- EXPLAIN extendido con formato JSON
EXPLAIN FORMAT=JSON SELECT * FROM productos WHERE categoria_id = 3;

-- EXPLAIN ANALYZE (MySQL 8.0.18+): ejecuta y muestra tiempos reales
EXPLAIN ANALYZE SELECT * FROM productos WHERE categoria_id = 3\G

-- Columnas clave de EXPLAIN:
-- type:  ALL (full scan ⚠️) | index | range | ref | eq_ref | const (✅ mejor)
-- key:   índice usado (NULL = sin índice ⚠️)
-- rows:  estimación de filas examinadas
-- Extra: "Using index" ✅ | "Using filesort" ⚠️ | "Using temporary" ⚠️

-- Slow Query Log
SET GLOBAL slow_query_log       = ON;
SET GLOBAL long_query_time      = 1;  -- Queries > 1 segundo
SET GLOBAL slow_query_log_file  = '/var/log/mysql/slow.log';
SHOW VARIABLES LIKE 'slow_query%';

-- Consultas más lentas (Performance Schema)
SELECT digest_text, count_star, avg_timer_wait/1000000000 AS avg_ms
FROM performance_schema.events_statements_summary_by_digest
ORDER BY avg_timer_wait DESC LIMIT 10;

-- Actualizar estadísticas del optimizador
ANALYZE TABLE productos;

-- Desfragmentar tabla
OPTIMIZE TABLE productos;

-- Estado del motor InnoDB
SHOW ENGINE INNODB STATUS\G
```

---

## 7. 🧩 Vistas (Views)

```sql
-- Vista simple
CREATE VIEW v_productos_activos AS
SELECT p.producto_id, p.nombre, p.precio, p.stock, c.nombre AS categoria
FROM productos p
JOIN categorias c ON p.categoria_id = c.categoria_id
WHERE p.activo = 1;

-- Crear o reemplazar
CREATE OR REPLACE VIEW v_productos_activos AS
SELECT p.producto_id, p.nombre, p.precio, p.stock,
       c.nombre AS categoria, pr.nombre AS proveedor
FROM productos p
JOIN categorias  c  ON p.categoria_id = c.categoria_id
JOIN proveedores pr ON p.proveedor_id = pr.proveedor_id
WHERE p.activo = 1;

-- WITH CHECK OPTION: restringe INSERT/UPDATE a filas visibles en la vista
CREATE VIEW v_productos_caros AS
SELECT * FROM productos WHERE precio > 5000
WITH CHECK OPTION;

-- Vista actualizable (sin GROUP BY, DISTINCT, subqueries, UNION)
CREATE VIEW v_productos_edit AS
SELECT producto_id, nombre, precio, stock, activo FROM productos;

-- DML sobre la vista
UPDATE v_productos_edit SET precio = 999 WHERE producto_id = 1;

-- Modificar / Eliminar vista
ALTER VIEW v_productos_activos AS SELECT ...;
DROP VIEW IF EXISTS v_productos_activos;

-- Listar vistas
SHOW FULL TABLES WHERE Table_type = 'VIEW';
SHOW CREATE VIEW v_productos_activos\G
```

---

## 8. ⚙️ Procedimientos Almacenados

```sql
DELIMITER $$

-- Procedimiento con IN, OUT e INOUT
CREATE PROCEDURE sp_obtener_productos(
    IN  p_categoria_id INT,
    IN  p_solo_activos TINYINT,
    IN  p_pagina       INT,
    IN  p_tamano_pag   INT,
    OUT p_total        INT
)
BEGIN
    DECLARE v_offset INT DEFAULT (p_pagina - 1) * p_tamano_pag;

    -- Contar total
    SELECT COUNT(*) INTO p_total
    FROM productos
    WHERE (p_categoria_id IS NULL OR categoria_id = p_categoria_id)
      AND (p_solo_activos = 0 OR activo = 1);

    -- Retornar página
    SELECT p.producto_id, p.nombre, p.precio, p.stock, c.nombre AS categoria
    FROM productos p
    JOIN categorias c ON p.categoria_id = c.categoria_id
    WHERE (p_categoria_id IS NULL OR p.categoria_id = p_categoria_id)
      AND (p_solo_activos = 0 OR p.activo = 1)
    ORDER BY p.nombre
    LIMIT p_tamano_pag OFFSET v_offset;
END$$

-- Procedimiento con manejo de errores
CREATE PROCEDURE sp_transferir_saldo(
    IN p_origen  INT,
    IN p_destino INT,
    IN p_monto   DECIMAL(12,2)
)
BEGIN
    DECLARE v_error     TINYINT DEFAULT 0;
    DECLARE v_msj_error VARCHAR(255);

    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
        SET v_error = 1;
        GET DIAGNOSTICS CONDITION 1 v_msj_error = MESSAGE_TEXT;
    END;

    START TRANSACTION;
        UPDATE cuentas SET saldo = saldo - p_monto WHERE cuenta_id = p_origen;
        UPDATE cuentas SET saldo = saldo + p_monto WHERE cuenta_id = p_destino;

        IF v_error THEN
            ROLLBACK;
            SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = v_msj_error;
        ELSE
            COMMIT;
        END IF;
END$$

DELIMITER ;

-- Ejecutar
CALL sp_obtener_productos(3, 1, 1, 20, @total);
SELECT @total AS total_registros;

-- Ver / Eliminar procedimientos
SHOW PROCEDURE STATUS WHERE Db = DATABASE();
SHOW CREATE PROCEDURE sp_obtener_productos\G
DROP PROCEDURE IF EXISTS sp_obtener_productos;
```

---

## 9. 🔧 Funciones

```sql
DELIMITER $$

-- Función escalar
CREATE FUNCTION fn_precio_con_iva(p_precio DECIMAL(12,2))
RETURNS DECIMAL(12,2)
DETERMINISTIC
BEGIN
    RETURN p_precio * 1.16;
END$$

-- Función con lógica condicional
CREATE FUNCTION fn_clasificar_precio(p_precio DECIMAL(12,2))
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
    RETURN CASE
        WHEN p_precio < 500   THEN 'Económico'
        WHEN p_precio < 2000  THEN 'Medio'
        WHEN p_precio < 10000 THEN 'Premium'
        ELSE 'Lujo'
    END;
END$$

-- Función que consulta datos (NOT DETERMINISTIC)
CREATE FUNCTION fn_stock_total_categoria(p_categoria_id INT)
RETURNS INT
READS SQL DATA
BEGIN
    DECLARE v_total INT DEFAULT 0;
    SELECT SUM(stock) INTO v_total
    FROM productos
    WHERE categoria_id = p_categoria_id AND activo = 1;
    RETURN IFNULL(v_total, 0);
END$$

DELIMITER ;

-- Uso
SELECT nombre, precio,
       fn_precio_con_iva(precio)    AS precio_iva,
       fn_clasificar_precio(precio) AS clasificacion
FROM productos;

-- Ver / Eliminar funciones
SHOW FUNCTION STATUS WHERE Db = DATABASE();
DROP FUNCTION IF EXISTS fn_precio_con_iva;
```

---

## 10. 🔔 Triggers

```sql
DELIMITER $$

-- AFTER UPDATE: auditoría de cambios de precio
CREATE TRIGGER trg_auditoria_precio
AFTER UPDATE ON productos
FOR EACH ROW
BEGIN
    IF OLD.precio <> NEW.precio THEN
        INSERT INTO auditoria_precios (
            producto_id, precio_anterior, precio_nuevo, usuario, fecha
        ) VALUES (OLD.producto_id, OLD.precio, NEW.precio, USER(), NOW());
    END IF;
END$$

-- BEFORE INSERT: validación y normalización
CREATE TRIGGER trg_before_insert_producto
BEFORE INSERT ON productos
FOR EACH ROW
BEGIN
    -- Normalizar
    SET NEW.nombre    = TRIM(NEW.nombre);
    SET NEW.creado_en = IFNULL(NEW.creado_en, NOW());

    -- Validar
    IF NEW.precio < 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'El precio no puede ser negativo';
    END IF;
END$$

-- BEFORE DELETE: respaldo antes de eliminar
CREATE TRIGGER trg_before_delete_producto
BEFORE DELETE ON productos
FOR EACH ROW
BEGIN
    INSERT INTO productos_eliminados (producto_id, nombre, precio, eliminado_en, eliminado_por)
    VALUES (OLD.producto_id, OLD.nombre, OLD.precio, NOW(), USER());
END$$

DELIMITER ;

-- Ver / Eliminar triggers
SHOW TRIGGERS FROM tienda;
SHOW TRIGGERS LIKE 'trg_%'\G
DROP TRIGGER IF EXISTS trg_auditoria_precio;
```

---

## 11. 🗓️ Eventos Programados (Event Scheduler)

```sql
-- Habilitar el scheduler
SET GLOBAL event_scheduler = ON;

-- Evento único (una sola vez)
CREATE EVENT ev_limpieza_unica
ON SCHEDULE AT '2026-12-31 23:59:00'
DO DELETE FROM logs WHERE fecha < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- Evento recurrente diario
CREATE EVENT ev_limpieza_logs
ON SCHEDULE EVERY 1 DAY
STARTS CURRENT_TIMESTAMP
DO DELETE FROM logs WHERE fecha < DATE_SUB(NOW(), INTERVAL 90 DAY);

-- Evento mensual con expiración
CREATE EVENT ev_resumen_mensual
ON SCHEDULE EVERY 1 MONTH
STARTS '2026-02-01 06:00:00'
ENDS   '2027-01-01 00:00:00'
ON COMPLETION PRESERVE
COMMENT 'Genera resumen mensual de ventas'
DO CALL sp_resumen_mensual();

-- Administrar eventos
SHOW EVENTS FROM tienda;
ALTER EVENT ev_limpieza_logs DISABLE;
ALTER EVENT ev_limpieza_logs ENABLE;
DROP EVENT IF EXISTS ev_limpieza_logs;
```

---

## 12. 📅 Funciones de Fecha y Hora

| Importancia | Función | Descripción |
|-------------|---------|-------------|
| 🔴 | `NOW()` | Fecha y hora actual |
| 🔴 | `CURDATE()` / `CURRENT_DATE` | Solo fecha |
| 🔴 | `CURTIME()` / `CURRENT_TIME` | Solo hora |
| 🟠 | `DATE_ADD(f, INTERVAL n u)` | Sumar a una fecha |
| 🟠 | `DATE_SUB(f, INTERVAL n u)` | Restar a una fecha |
| 🟠 | `DATEDIFF(f1, f2)` | Diferencia en días |
| 🟠 | `TIMESTAMPDIFF(u, f1, f2)` | Diferencia en unidad dada |
| 🟠 | `DATE_FORMAT(f, fmt)` | Formatear fecha |
| 🟠 | `YEAR()` / `MONTH()` / `DAY()` | Extraer partes |
| 🟡 | `LAST_DAY(fecha)` | Último día del mes |
| 🟡 | `WEEK(fecha)` / `WEEKDAY(fecha)` | Número de semana / día |
| 🟡 | `UNIX_TIMESTAMP()` / `FROM_UNIXTIME()` | Timestamp Unix |
| 🟡 | `STR_TO_DATE(str, fmt)` | Parsear fecha desde string |
| 🟡 | `CONVERT_TZ(f, from_tz, to_tz)` | Convertir zona horaria |

```sql
-- Unidades de INTERVAL: MICROSECOND, SECOND, MINUTE, HOUR,
--                        DAY, WEEK, MONTH, QUARTER, YEAR

SELECT
    NOW()                                        AS ahora,
    CURDATE()                                    AS solo_fecha,
    DATE_ADD(NOW(), INTERVAL 30 DAY)             AS en_30_dias,
    DATE_SUB(NOW(), INTERVAL 3 MONTH)            AS hace_3_meses,
    DATEDIFF(NOW(), '2026-01-01')                AS dias_desde_inicio,
    TIMESTAMPDIFF(MONTH, '2025-01-01', NOW())    AS meses_transcurridos,
    LAST_DAY(NOW())                              AS fin_de_mes,
    DATE_FORMAT(NOW(), '%d/%m/%Y %H:%i')         AS formateado,
    DAYNAME(NOW())                               AS nombre_dia,
    MONTHNAME(NOW())                             AS nombre_mes,
    QUARTER(NOW())                               AS trimestre,
    WEEK(NOW(), 3)                               AS semana_iso;

-- Filtros frecuentes
WHERE fecha >= DATE_SUB(NOW(), INTERVAL 30 DAY)    -- Últimos 30 días
WHERE YEAR(fecha) = 2026 AND MONTH(fecha) = 5       -- Mayo 2026
WHERE fecha BETWEEN '2026-01-01' AND '2026-12-31'   -- Todo 2026
WHERE DATE(fecha) = CURDATE()                       -- Hoy

-- Inicio del mes actual
SELECT DATE_FORMAT(NOW(), '%Y-%m-01') AS inicio_mes;

-- Inicio de la semana (lunes)
SELECT DATE_SUB(CURDATE(), INTERVAL WEEKDAY(CURDATE()) DAY) AS inicio_semana;

-- Parsear fecha desde string
SELECT STR_TO_DATE('03/05/2026', '%d/%m/%Y') AS fecha_parseada;
```

---

## 13. 📝 Funciones de Texto

| Importancia | Función | Descripción |
|-------------|---------|-------------|
| 🔴 | `CONCAT(a, b, ...)` | Concatenar cadenas |
| 🔴 | `CHAR_LENGTH(str)` | Longitud en caracteres (UTF-8 safe ✅) |
| 🔴 | `LENGTH(str)` | Longitud en bytes |
| 🔴 | `UPPER()` / `LOWER()` | Mayúsculas / minúsculas |
| 🔴 | `TRIM()` / `LTRIM()` / `RTRIM()` | Eliminar espacios |
| 🔴 | `SUBSTRING(str, pos, len)` | Extraer subcadena |
| 🟠 | `REPLACE(str, old, new)` | Reemplazar texto |
| 🟠 | `INSTR(str, sub)` | Posición de subcadena |
| 🟠 | `LEFT(str, n)` / `RIGHT(str, n)` | Primeros/últimos N chars |
| 🟠 | `LPAD(str, n, c)` / `RPAD(str, n, c)` | Rellenar izq/der |
| 🟠 | `GROUP_CONCAT(col)` | Agrupar valores en cadena (exclusivo MySQL) |
| 🟠 | `SUBSTRING_INDEX(str, delim, n)` | Extraer hasta N-ésimo delimitador |
| 🟡 | `REGEXP_REPLACE(str, pat, rep)` | Reemplazar con regex (MySQL 8+) |
| 🟡 | `REGEXP_SUBSTR(str, pat)` | Extraer con regex (MySQL 8+) |
| 🟡 | `FORMAT(n, d)` | Formato numérico con separadores de miles |
| 🟡 | `REVERSE(str)` | Invertir cadena |
| 🟡 | `REPEAT(str, n)` | Repetir cadena N veces |

```sql
SELECT
    CONCAT(nombre, ' - $', FORMAT(precio, 2))          AS descripcion,
    CHAR_LENGTH(nombre)                                 AS longitud,        -- chars
    LENGTH(nombre)                                      AS bytes,           -- bytes
    UPPER(LEFT(nombre, 1))                              AS inicial,
    TRIM(BOTH ' ' FROM '  hola  ')                     AS limpio,
    LPAD(CAST(producto_id AS CHAR), 6, '0')            AS id_padded,       -- '000042'
    REPLACE(email, '@', ' [at] ')                      AS email_ofuscado,
    SUBSTRING_INDEX(email, '@', -1)                    AS dominio,
    REGEXP_REPLACE(telefono, '[^0-9]', '')              AS tel_solo_num,
    FORMAT(precio, 2, 'es_MX')                         AS precio_formato;

-- SUBSTRING_INDEX: muy útil para dividir cadenas
SELECT SUBSTRING_INDEX('nombre@dominio.com', '@', 1)  AS usuario;    -- 'nombre'
SELECT SUBSTRING_INDEX('nombre@dominio.com', '@', -1) AS dominio;    -- 'dominio.com'
SELECT SUBSTRING_INDEX('a,b,c,d', ',', 2)             AS dos_primeros; -- 'a,b'

-- GROUP_CONCAT avanzado
SELECT categoria_id,
       GROUP_CONCAT(nombre ORDER BY nombre ASC SEPARATOR ' | ') AS productos
FROM productos GROUP BY categoria_id;
```

---

## 14. 🔢 Funciones Numéricas y Condicionales

```sql
-- Numéricas
SELECT
    ABS(-42)                     AS absoluto,
    ROUND(3.14159, 2)            AS redondeado,    -- 3.14
    CEIL(3.2)                    AS techo,          -- 4
    FLOOR(3.9)                   AS piso,           -- 3
    TRUNCATE(3.14159, 2)         AS truncado,       -- 3.14 (sin redondear)
    MOD(10, 3)                   AS modulo,         -- 1
    POWER(2, 10)                 AS potencia,       -- 1024
    SQRT(144)                    AS raiz,           -- 12
    PI()                         AS pi,
    RAND()                       AS aleatorio,
    GREATEST(3, 7, 2, 9)        AS mayor,          -- 9
    LEAST(3, 7, 2, 9)           AS menor,          -- 2
    SIGN(-5)                     AS signo,          -- -1
    DIV(10, 3)                   AS division_entera; -- 3

-- Condicionales
SELECT
    CASE
        WHEN precio < 500  THEN 'Económico'
        WHEN precio < 2000 THEN 'Medio'
        ELSE 'Premium'
    END                                              AS rango,

    IF(stock > 0, 'Disponible', 'Agotado')          AS disponibilidad,
    IFNULL(descripcion, 'Sin descripción')           AS desc_segura,
    NULLIF(descuento, 0)                             AS descuento_real,
    COALESCE(desc_corta, nombre, codigo, 'N/D')     AS etiqueta,

    -- ELT: seleccionar por índice (1-based)
    ELT(MONTH(NOW()), 'Ene','Feb','Mar','Abr','May','Jun',
                       'Jul','Ago','Sep','Oct','Nov','Dic') AS mes_abrev;
```

---

## 15. 📦 Manejo de JSON (MySQL 5.7.8+)

```sql
-- Insertar JSON
INSERT INTO productos (nombre, atributos)
VALUES ('Laptop', '{"color":"negro","ram":"16GB","disco":"512GB","garantia":2}');

-- Leer campo JSON
SELECT
    nombre,
    JSON_EXTRACT(atributos, '$.color')  AS color,
    atributos->>'$.ram'                 AS ram,       -- Operador shorthand (sin comillas)
    atributos->'$.garantia'             AS garantia;  -- Con comillas JSON

-- Modificar JSON
UPDATE productos
SET atributos = JSON_SET(atributos, '$.color', 'gris', '$.ram', '32GB')
WHERE producto_id = 1;

-- Agregar clave nueva
UPDATE productos
SET atributos = JSON_INSERT(atributos, '$.stock_minimo', 5)
WHERE producto_id = 1;

-- Eliminar clave
UPDATE productos
SET atributos = JSON_REMOVE(atributos, '$.garantia')
WHERE producto_id = 1;

-- Verificar si una clave existe
SELECT JSON_CONTAINS_PATH(atributos, 'one', '$.color') AS tiene_color FROM productos;

-- Buscar en JSON
SELECT * FROM productos WHERE atributos->>'$.color' = 'negro';
SELECT * FROM productos WHERE JSON_EXTRACT(atributos, '$.garantia') > 1;

-- JSON_TABLE: desanidar JSON en filas (MySQL 8.0+)
SELECT p.nombre, j.*
FROM productos p,
JSON_TABLE(p.atributos, '$' COLUMNS (
    color    VARCHAR(50) PATH '$.color',
    ram      VARCHAR(20) PATH '$.ram',
    garantia INT         PATH '$.garantia'
)) AS j;

-- Agregar JSON desde filas
SELECT
    JSON_ARRAYAGG(nombre)           AS array_nombres,
    JSON_OBJECTAGG(codigo, precio)  AS objeto_precios
FROM productos WHERE activo = 1;

-- Validar JSON
SELECT JSON_VALID('{"clave":"valor"}') AS es_valido;  -- 1 = válido
SELECT JSON_VALID('clave:valor')       AS invalido;   -- 0 = no es JSON
```

---

## 16. 🔒 DCL — Control de Acceso

```sql
-- Crear usuario
CREATE USER 'app_usuario'@'localhost'  IDENTIFIED BY 'C0ntraseña$egura!';
CREATE USER 'app_usuario'@'%'          IDENTIFIED BY 'Otra$lave!';  -- Desde cualquier host

-- Permisos por tabla
GRANT SELECT, INSERT, UPDATE, DELETE ON tienda.*          TO 'app_usuario'@'localhost';
GRANT SELECT                          ON tienda.productos  TO 'app_readonly'@'%';
GRANT ALL PRIVILEGES                  ON tienda.*          TO 'app_usuario'@'localhost';
GRANT EXECUTE ON PROCEDURE tienda.sp_obtener_productos    TO 'app_usuario'@'localhost';

-- Revocar permisos
REVOKE INSERT, UPDATE ON tienda.* FROM 'app_usuario'@'localhost';
REVOKE ALL PRIVILEGES ON tienda.* FROM 'app_usuario'@'localhost';

-- Aplicar cambios (necesario si se modifica mysql.user directamente)
FLUSH PRIVILEGES;

-- Ver permisos de un usuario
SHOW GRANTS FOR 'app_usuario'@'localhost';

-- Cambiar contraseña
ALTER USER 'app_usuario'@'localhost' IDENTIFIED BY 'NuevaPass$!';

-- Bloquear / desbloquear cuenta
ALTER USER 'app_usuario'@'localhost' ACCOUNT LOCK;
ALTER USER 'app_usuario'@'localhost' ACCOUNT UNLOCK;

-- Eliminar usuario
DROP USER IF EXISTS 'app_usuario'@'localhost';

-- Ver todos los usuarios
SELECT user, host, account_locked, password_expired
FROM mysql.user ORDER BY user;
```

---

## 17. 📋 Administración y Monitoreo

```sql
-- Listar tablas con detalles y tamaño
SELECT
    table_name                                    AS tabla,
    engine,
    table_rows                                    AS filas_est,
    ROUND(data_length  / 1024 / 1024, 2)          AS data_mb,
    ROUND(index_length / 1024 / 1024, 2)          AS index_mb,
    table_collation,
    create_time
FROM information_schema.tables
WHERE table_schema = DATABASE()
ORDER BY data_length DESC;

-- Ver procesos activos
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;  -- Query completa sin truncar

-- Terminar proceso
KILL 42;         -- Proceso y query
KILL QUERY 42;   -- Solo la query, conserva la conexión

-- Estado del servidor InnoDB
SHOW ENGINE INNODB STATUS\G

-- Verificar buffer pool hit ratio (debe ser > 99%)
SELECT
    ROUND((1 - (
        (SELECT variable_value FROM performance_schema.global_status WHERE variable_name = 'Innodb_buffer_pool_reads') /
        (SELECT variable_value FROM performance_schema.global_status WHERE variable_name = 'Innodb_buffer_pool_read_requests')
    )) * 100, 2) AS buffer_pool_hit_pct;

-- ═══════════════════════════════════════
-- BACKUP Y RESTORE (línea de comandos)
-- ═══════════════════════════════════════

-- Backup completo
-- mysqldump -u root -p tienda > tienda_backup.sql

-- Backup con opciones avanzadas
-- mysqldump -u root -p \
--     --single-transaction \   (consistente sin bloquear InnoDB)
--     --routines \             (SPs y funciones)
--     --triggers \             (triggers)
--     --events \               (eventos programados)
--     tienda > tienda_full.sql

-- Backup comprimido
-- mysqldump -u root -p tienda | gzip > tienda_$(date +%Y%m%d).sql.gz

-- Restore
-- mysql -u root -p tienda < tienda_backup.sql
-- gunzip -c tienda_20260503.sql.gz | mysql -u root -p tienda

-- Backup solo estructura (sin datos)
-- mysqldump -u root -p --no-data tienda > estructura.sql

-- Backup solo datos (sin estructura)
-- mysqldump -u root -p --no-create-info tienda > datos.sql
```

---

## 18. 🔀 Programación T-SQL en MySQL

### 18.1 Variables y Control de Flujo

```sql
-- Variables de sesión (@)
SET @nombre     = 'MySQL';
SET @precio     = 1299.99;
SET @fecha      = CURDATE();

-- Asignar desde query
SELECT MAX(precio) INTO @max_precio FROM productos;

-- Variables locales (solo dentro de BEGIN...END)
DECLARE v_contador INT         DEFAULT 0;
DECLARE v_nombre   VARCHAR(100);
DECLARE v_total    DECIMAL(12,2) NOT NULL DEFAULT 0.00;

-- IF / ELSEIF / ELSE
IF v_precio > 1000 THEN
    SET v_nombre = 'Caro';
ELSEIF v_precio > 500 THEN
    SET v_nombre = 'Medio';
ELSE
    SET v_nombre = 'Barato';
END IF;

-- WHILE
WHILE v_contador < 10 DO
    SET v_contador = v_contador + 1;
END WHILE;

-- REPEAT ... UNTIL (equivale a do-while)
REPEAT
    SET v_contador = v_contador + 1;
UNTIL v_contador >= 10 END REPEAT;

-- LOOP con etiqueta + LEAVE + ITERATE
mi_loop: LOOP
    SET v_contador = v_contador + 1;
    IF v_contador = 5 THEN ITERATE mi_loop; END IF;  -- continue
    IF v_contador >= 10 THEN LEAVE mi_loop; END IF;  -- break
END LOOP mi_loop;
```

---

### 18.2 Manejo de Errores con SIGNAL

```sql
DELIMITER $$

CREATE PROCEDURE sp_con_errores(IN p_id INT)
BEGIN
    DECLARE v_error     TINYINT DEFAULT 0;
    DECLARE v_msj_error VARCHAR(255);

    -- Handler general para errores SQL
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
        SET v_error = 1;
        GET DIAGNOSTICS CONDITION 1 v_msj_error = MESSAGE_TEXT;
    END;

    -- Handler para NOT FOUND
    DECLARE CONTINUE HANDLER FOR NOT FOUND
    BEGIN
        SET v_error = 1;
        SET v_msj_error = 'Registro no encontrado';
    END;

    START TRANSACTION;
        UPDATE productos SET stock = stock - 1 WHERE producto_id = p_id;

        IF v_error THEN
            ROLLBACK;
            SELECT v_msj_error AS error;
        ELSE
            COMMIT;
            SELECT 'OK' AS resultado;
        END IF;
END$$

-- SIGNAL: lanzar error personalizado
CREATE PROCEDURE sp_validar(IN p_precio DECIMAL(12,2))
BEGIN
    IF p_precio < 0 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'El precio no puede ser negativo',
                MYSQL_ERRNO  = 1644;
    END IF;
END$$

DELIMITER ;
```

---

### 18.3 SQL Dinámico — Prepared Statements

```sql
-- Prepared statement de sesión
SET @sql = 'SELECT COUNT(*) FROM productos WHERE categoria_id = ?';

PREPARE stmt FROM @sql;

SET @cat = 3;
EXECUTE stmt USING @cat;
DEALLOCATE PREPARE stmt;

-- Nombre de tabla dinámico (solo valores con ?)
SET @tabla = 'productos';
SET @sql   = CONCAT('SELECT COUNT(*) FROM ', @tabla);  -- ⚠️ validar @tabla
PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;

-- En procedimiento almacenado
DELIMITER $$
CREATE PROCEDURE sp_buscar(IN p_columna VARCHAR(100), IN p_valor VARCHAR(200))
BEGIN
    SET @q = CONCAT('SELECT * FROM productos WHERE ', p_columna, ' LIKE ?');
    SET @v = CONCAT('%', p_valor, '%');
    PREPARE stmt FROM @q;
    EXECUTE stmt USING @v;
    DEALLOCATE PREPARE stmt;
END$$
DELIMITER ;
```

---

## 19. ✅ Buenas Prácticas MySQL

```sql
-- 1. Siempre usar InnoDB
CREATE TABLE mi_tabla (...) ENGINE = InnoDB;

-- 2. Charset utf8mb4 (⚠️ utf8 en MySQL = utf8mb3, no soporta emojis)
CREATE TABLE mi_tabla (...) DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_unicode_ci;

-- 3. ON UPDATE CURRENT_TIMESTAMP para auditoría automática
modificado_en DATETIME NULL ON UPDATE CURRENT_TIMESTAMP

-- 4. SIEMPRE WHERE en UPDATE y DELETE
UPDATE productos SET activo = 0 WHERE producto_id = 42;  -- ✅
UPDATE productos SET activo = 0;                          -- ⚠️ Actualiza todo

-- 5. EXPLAIN para detectar full scans
EXPLAIN SELECT * FROM productos WHERE nombre LIKE '%laptop%';
-- type = ALL → agregar índice o refactorizar

-- 6. Filtros SARGable (que aprovechan índices)
WHERE fecha BETWEEN '2026-01-01' AND '2026-12-31' -- ✅ Usa índice
WHERE YEAR(fecha) = 2026                           -- ❌ Ignora índice (función en columna)
WHERE precio * 1.16 > 1000                         -- ❌ Ignora índice
WHERE precio > 1000 / 1.16                         -- ✅ Usa índice

-- 7. CHAR_LENGTH vs LENGTH con utf8mb4
WHERE CHAR_LENGTH(nombre) > 100   -- ✅ Cuenta caracteres Unicode
WHERE LENGTH(nombre) > 100        -- ⚠️ Cuenta bytes (emoji = 4 bytes)

-- 8. Borrar en lotes para tablas grandes
DELETE FROM logs WHERE fecha < DATE_SUB(NOW(), INTERVAL 90 DAY)
ORDER BY fecha ASC LIMIT 10000;
-- Repetir hasta ROW_COUNT() = 0

-- 9. LAST_INSERT_ID() inmediatamente después del INSERT
INSERT INTO pedidos (cliente_id, total) VALUES (1, 500.00);
SET @pedido_id = LAST_INSERT_ID();  -- Capturar de inmediato

-- 10. Usar prepared statements (previene SQL Injection)
-- En el driver/ORM: siempre usar parámetros binding
-- NUNCA: "SELECT * FROM users WHERE email = '" + email + "'"

-- 11. Índice compuesto: columna más selectiva primero
CREATE INDEX idx_activo_cat ON productos(activo, categoria_id);
-- Si 'activo' filtra más (ej. 95% inactivos), va primero

-- 12. DELIMITER solo en cliente de línea de comandos
-- En frameworks (Laravel, Django, etc.), enviar SP sin DELIMITER

-- 13. UUID vs AUTO_INCREMENT
SELECT UUID();        -- GUID completo (lento para PK clustered)
SELECT UUID_SHORT();  -- INT de 64 bits ordenado (mejor para PK)
```

---

## 20. 🗺️ Mapa Mental MySQL

```
MySQL 8.0+
├── DDL (Definición)
│   ├── DATABASE   → CREATE [IF NOT EXISTS], ALTER, DROP [IF EXISTS], USE, SHOW
│   ├── TABLE      → CREATE [IF NOT EXISTS], ALTER (ADD/MODIFY/CHANGE/RENAME/DROP)
│   │                DROP [IF EXISTS], TRUNCATE, RENAME, SHOW CREATE
│   ├── INDEX      → CREATE (INDEX/UNIQUE/FULLTEXT/SPATIAL), DROP, INVISIBLE (8+)
│   └── CONSTRAINTS → PK, FK (ON DELETE/UPDATE), UNIQUE, CHECK, DEFAULT
│                     ON UPDATE CURRENT_TIMESTAMP
│
├── DML (Manipulación)
│   ├── SELECT     → DISTINCT, JOIN (INNER/LEFT/RIGHT/CROSS/SELF)
│   │                WHERE, GROUP BY [WITH ROLLUP], HAVING, ORDER BY, LIMIT/OFFSET
│   │                CTEs [WITH RECURSIVE], Window Functions (8+), UNION
│   ├── INSERT     → VALUES, multiple rows, SELECT, INSERT IGNORE
│   │                ON DUPLICATE KEY UPDATE, REPLACE INTO
│   ├── UPDATE     → SET, JOIN (MySQL syntax), CASE, LIMIT
│   └── DELETE     → WHERE, JOIN, LIMIT + ORDER BY (borrado por lotes)
│
├── DCL (Control)
│   ├── CREATE USER / DROP USER / ALTER USER
│   ├── GRANT / REVOKE → por BD, tabla, columna, procedimiento
│   └── FLUSH PRIVILEGES
│
├── TCL (Transacciones)
│   ├── START TRANSACTION / COMMIT / ROLLBACK
│   ├── SAVEPOINT / ROLLBACK TO SAVEPOINT / RELEASE SAVEPOINT
│   ├── SET autocommit = 0|1
│   └── SET TRANSACTION ISOLATION LEVEL
│
├── Programación
│   ├── Variables de sesión → @variable
│   ├── Variables locales   → DECLARE (en SPs/funciones)
│   ├── Flujo               → IF/ELSEIF/ELSE, CASE, WHILE, REPEAT, LOOP
│   ├── Errores             → DECLARE HANDLER, SIGNAL, GET DIAGNOSTICS
│   ├── Cursores            → DECLARE CURSOR, OPEN, FETCH, CLOSE
│   └── SQL Dinámico        → PREPARE / EXECUTE / DEALLOCATE
│
└── Objetos
    ├── Vistas              → CREATE OR REPLACE VIEW, WITH CHECK OPTION
    ├── Procedimientos      → CALL, IN/OUT/INOUT, DELIMITER
    ├── Funciones           → DETERMINISTIC / READS SQL DATA
    ├── Triggers            → BEFORE/AFTER · INSERT/UPDATE/DELETE · OLD/NEW · SIGNAL
    └── Eventos             → ON SCHEDULE EVERY / AT, Event Scheduler
```

---

*📌 Referencia para MySQL 8.0+ / MariaDB 10.6+ · Actualización: 2026*
