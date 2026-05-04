# 🗄️ Referencia Completa de Comandos SQL
> Organizada por categorías e importancia · Con equivalentes para MySQL, PostgreSQL y SQL Server

---

## 📊 Escala de Importancia

| Nivel | Descripción |
|-------|-------------|
| 🔴 **Crítico** | Uso diario, esencial en cualquier proyecto con base de datos |
| 🟠 **Alto** | Frecuente en desarrollo real y administración |
| 🟡 **Medio** | Casos específicos, optimización y administración avanzada |
| 🟢 **Avanzado** | DBA, tuning, replicación y casos edge |

## 🏷️ Leyenda de Compatibilidad

| Ícono | Motor |
|-------|-------|
| 🐬 | MySQL / MariaDB |
| 🐘 | PostgreSQL |
| 🪟 | SQL Server (T-SQL) |
| ✅ | Compatible (igual o muy similar) |
| ⚠️ | Diferencia importante |
| ❌ | No soportado / alternativa diferente |

---

## 1. 🗂️ DDL — Definición de Datos (Data Definition Language)

### 1.1 Bases de Datos

| Importancia | Comando SQL Estándar | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|----------------------|----------|---------------|---------------|
| 🔴 | `CREATE DATABASE nombre` | ✅ Igual | ✅ Igual | ✅ Igual |
| 🔴 | `DROP DATABASE nombre` | ✅ Igual | ✅ Igual | ✅ Igual |
| 🟠 | `USE nombre_db` | ✅ Igual | ⚠️ `\c nombre_db` (psql) | ✅ Igual |
| 🟡 | `SHOW DATABASES` | ✅ Igual | ⚠️ `\l` o `SELECT datname FROM pg_database` | ⚠️ `SELECT name FROM sys.databases` |
| 🟡 | `ALTER DATABASE nombre ...` | ✅ Igual | ✅ Igual | ✅ Igual |

```sql
-- Crear base de datos con charset
-- 🐬 MySQL
CREATE DATABASE tienda CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 🐘 PostgreSQL
CREATE DATABASE tienda ENCODING 'UTF8' LC_COLLATE 'es_MX.UTF-8';

-- 🪟 SQL Server
CREATE DATABASE tienda COLLATE Modern_Spanish_CI_AS;
```

---

### 1.2 Tablas

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🔴 | Crear tabla | ✅ Igual | ✅ Igual | ✅ Igual |
| 🔴 | Eliminar tabla | ✅ Igual | ✅ Igual | ✅ Igual |
| 🔴 | Modificar tabla | ✅ Igual | ✅ Igual | ✅ Igual |
| 🟠 | Renombrar tabla | `RENAME TABLE viejo TO nuevo` | `ALTER TABLE viejo RENAME TO nuevo` | `EXEC sp_rename 'viejo', 'nuevo'` |
| 🟠 | Ver estructura | `DESCRIBE tabla` / `SHOW COLUMNS` | `\d tabla` o `SELECT column_name...` | `EXEC sp_help 'tabla'` |
| 🟡 | Copiar tabla | `CREATE TABLE nueva SELECT * FROM vieja` | `CREATE TABLE nueva AS SELECT * FROM vieja` | `SELECT * INTO nueva FROM vieja` |
| 🟡 | Tabla temporal | `CREATE TEMPORARY TABLE tmp (...)` | `CREATE TEMP TABLE tmp (...)` | `CREATE TABLE #tmp (...)` |

```sql
-- Crear tabla completa con constraints
-- ✅ Compatible en los tres motores (con variaciones en tipos)
CREATE TABLE productos (
    id          INT             NOT NULL,
    nombre      VARCHAR(150)    NOT NULL,
    precio      DECIMAL(10,2)   NOT NULL DEFAULT 0.00,
    stock       INT             NOT NULL DEFAULT 0,
    activo      BOOLEAN         NOT NULL DEFAULT TRUE,
    creado_en   TIMESTAMP       NOT NULL,
    categoria_id INT,
    CONSTRAINT pk_productos PRIMARY KEY (id),
    CONSTRAINT fk_categoria FOREIGN KEY (categoria_id)
        REFERENCES categorias(id) ON DELETE SET NULL
);

-- Eliminar tabla si existe
-- 🐬 MySQL / 🐘 PostgreSQL
DROP TABLE IF EXISTS productos;

-- 🪟 SQL Server
IF OBJECT_ID('productos', 'U') IS NOT NULL DROP TABLE productos;
```

---

### 1.3 Tipos de Datos por Motor

| Categoría | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-----------|----------|---------------|---------------|
| Entero pequeño | `TINYINT` / `SMALLINT` | `SMALLINT` | `TINYINT` / `SMALLINT` |
| Entero estándar | `INT` | `INTEGER` / `INT` | `INT` |
| Entero grande | `BIGINT` | `BIGINT` | `BIGINT` |
| Decimal | `DECIMAL(p,s)` | `NUMERIC(p,s)` | `DECIMAL(p,s)` |
| Flotante | `FLOAT` / `DOUBLE` | `REAL` / `DOUBLE PRECISION` | `FLOAT` / `REAL` |
| Texto corto | `VARCHAR(n)` | `VARCHAR(n)` | `VARCHAR(n)` / `NVARCHAR(n)` |
| Texto largo | `TEXT` / `LONGTEXT` | `TEXT` | `VARCHAR(MAX)` |
| Fecha | `DATE` | `DATE` | `DATE` |
| Fecha y hora | `DATETIME` / `TIMESTAMP` | `TIMESTAMP` / `TIMESTAMPTZ` | `DATETIME` / `DATETIME2` |
| Booleano | `TINYINT(1)` / `BOOLEAN` | `BOOLEAN` | `BIT` |
| UUID | `CHAR(36)` / `VARCHAR(36)` | `UUID` | `UNIQUEIDENTIFIER` |
| JSON | `JSON` | `JSON` / `JSONB` | `NVARCHAR(MAX)` + `ISJSON()` |
| Auto-incremental | `AUTO_INCREMENT` | `SERIAL` / `GENERATED ALWAYS AS IDENTITY` | `IDENTITY(1,1)` |
| Binario | `BLOB` / `LONGBLOB` | `BYTEA` | `VARBINARY(MAX)` |

```sql
-- Auto-incremento (clave primaria)
-- 🐬 MySQL
CREATE TABLE usuarios (id INT AUTO_INCREMENT PRIMARY KEY, nombre VARCHAR(100));

-- 🐘 PostgreSQL
CREATE TABLE usuarios (id SERIAL PRIMARY KEY, nombre VARCHAR(100));
-- Moderno:
CREATE TABLE usuarios (id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, nombre VARCHAR(100));

-- 🪟 SQL Server
CREATE TABLE usuarios (id INT IDENTITY(1,1) PRIMARY KEY, nombre VARCHAR(100));
```

---

### 1.4 ALTER TABLE — Modificar Tablas

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🔴 | Agregar columna | `ALTER TABLE t ADD COLUMN col tipo` | ✅ Igual | `ALTER TABLE t ADD col tipo` |
| 🟠 | Eliminar columna | `ALTER TABLE t DROP COLUMN col` | ✅ Igual | ✅ Igual |
| 🟠 | Modificar tipo | `ALTER TABLE t MODIFY col nuevo_tipo` | `ALTER TABLE t ALTER COLUMN col TYPE nuevo_tipo` | `ALTER TABLE t ALTER COLUMN col nuevo_tipo` |
| 🟠 | Renombrar columna | `ALTER TABLE t RENAME COLUMN viejo TO nuevo` | ✅ Igual | `EXEC sp_rename 'tabla.col_vieja', 'col_nueva', 'COLUMN'` |
| 🟡 | Agregar constraint | `ALTER TABLE t ADD CONSTRAINT nombre ...` | ✅ Igual | ✅ Igual |
| 🟡 | Eliminar constraint | `ALTER TABLE t DROP FOREIGN KEY nombre` | `ALTER TABLE t DROP CONSTRAINT nombre` | `ALTER TABLE t DROP CONSTRAINT nombre` |

---

## 2. 🔍 DQL — Consulta de Datos (Data Query Language)

### 2.1 SELECT Básico

| Importancia | Comando | Compatibilidad |
|-------------|---------|----------------|
| 🔴 | `SELECT * FROM tabla` | ✅ Los tres motores |
| 🔴 | `SELECT col1, col2 FROM tabla` | ✅ Los tres motores |
| 🔴 | `SELECT col AS alias FROM tabla` | ✅ Los tres motores |
| 🔴 | `SELECT DISTINCT col FROM tabla` | ✅ Los tres motores |
| 🔴 | `SELECT ... WHERE condición` | ✅ Los tres motores |
| 🔴 | `SELECT ... ORDER BY col ASC/DESC` | ✅ Los tres motores |

```sql
-- SELECT completo con alias y condición
SELECT
    p.id,
    p.nombre                    AS producto,
    p.precio,
    c.nombre                    AS categoria,
    p.precio * 1.16             AS precio_con_iva
FROM productos p
INNER JOIN categorias c ON p.categoria_id = c.id
WHERE p.activo = TRUE
  AND p.precio BETWEEN 100 AND 5000
ORDER BY p.precio DESC;
```

---

### 2.2 Filtros — WHERE y Operadores

| Importancia | Operador / Cláusula | Descripción | Compatibilidad |
|-------------|---------------------|-------------|----------------|
| 🔴 | `=`, `<>`, `!=`, `<`, `>`, `<=`, `>=` | Comparación básica | ✅ Los tres |
| 🔴 | `AND`, `OR`, `NOT` | Operadores lógicos | ✅ Los tres |
| 🔴 | `IS NULL` / `IS NOT NULL` | Verificar nulos | ✅ Los tres |
| 🔴 | `IN (val1, val2, ...)` | Valores en lista | ✅ Los tres |
| 🔴 | `BETWEEN val1 AND val2` | Rango inclusivo | ✅ Los tres |
| 🔴 | `LIKE 'patrón%'` | Búsqueda con comodín | ✅ Los tres |
| 🟠 | `ILIKE 'patrón%'` | LIKE sin distinción de mayúsculas | ❌ No existe | ✅ Nativo | ⚠️ `LIKE` con collation CI |
| 🟠 | `NOT IN (...)` | Excluir valores | ✅ Los tres |
| 🟡 | `EXISTS (subconsulta)` | Verifica si subquery retorna filas | ✅ Los tres |
| 🟡 | `REGEXP` / `~` | Búsqueda con expresión regular | `REGEXP` | `~` (sensible) / `~*` (insensible) | `LIKE` o `PATINDEX` |

---

### 2.3 LIMIT / TOP / FETCH

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🔴 | Limitar filas | `LIMIT n` | `LIMIT n` | `TOP n` (antes del FROM) |
| 🟠 | Paginación | `LIMIT n OFFSET m` | `LIMIT n OFFSET m` | `OFFSET m ROWS FETCH NEXT n ROWS ONLY` |

```sql
-- Paginación: página 3, 10 registros por página

-- 🐬 MySQL / 🐘 PostgreSQL
SELECT * FROM productos ORDER BY id LIMIT 10 OFFSET 20;

-- 🪟 SQL Server
SELECT * FROM productos ORDER BY id OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;

-- TOP (SQL Server, solo primeras N filas)
SELECT TOP 10 * FROM productos ORDER BY precio DESC;
```

---

### 2.4 JOINs — Unión de Tablas

| Importancia | Tipo de JOIN | Descripción | Compatibilidad |
|-------------|--------------|-------------|----------------|
| 🔴 | `INNER JOIN` | Solo filas con coincidencia en ambas tablas | ✅ Los tres |
| 🔴 | `LEFT JOIN` | Todas las filas de la izquierda + coincidencias | ✅ Los tres |
| 🟠 | `RIGHT JOIN` | Todas las filas de la derecha + coincidencias | ✅ Los tres |
| 🟠 | `FULL OUTER JOIN` | Todas las filas de ambas tablas | ⚠️ No nativo (simular con UNION) | ✅ Nativo | ✅ Nativo |
| 🟡 | `CROSS JOIN` | Producto cartesiano | ✅ Los tres |
| 🟡 | `SELF JOIN` | La tabla se une consigo misma | ✅ Los tres |

```sql
-- FULL OUTER JOIN simulado en MySQL
SELECT p.*, c.nombre AS categoria
FROM productos p
LEFT JOIN categorias c ON p.categoria_id = c.id
UNION
SELECT p.*, c.nombre AS categoria
FROM productos p
RIGHT JOIN categorias c ON p.categoria_id = c.id;

-- FULL OUTER JOIN nativo (PostgreSQL / SQL Server)
SELECT p.*, c.nombre AS categoria
FROM productos p
FULL OUTER JOIN categorias c ON p.categoria_id = c.id;
```

---

### 2.5 Agregación y Agrupación

| Importancia | Función / Cláusula | Descripción | Compatibilidad |
|-------------|-------------------|-------------|----------------|
| 🔴 | `COUNT(*)` / `COUNT(col)` | Cuenta filas | ✅ Los tres |
| 🔴 | `SUM(col)` | Suma valores | ✅ Los tres |
| 🔴 | `AVG(col)` | Promedio | ✅ Los tres |
| 🔴 | `MAX(col)` / `MIN(col)` | Máximo y mínimo | ✅ Los tres |
| 🔴 | `GROUP BY col` | Agrupa resultados | ✅ Los tres |
| 🔴 | `HAVING condición` | Filtra grupos | ✅ Los tres |
| 🟠 | `GROUP_CONCAT(col)` | Concatena valores de grupo | ✅ MySQL | `STRING_AGG(col, ',')` | `STRING_AGG(col, ',')` |
| 🟡 | `ROLLUP` | Totales jerárquicos | ✅ Los tres (sintaxis varía) |
| 🟡 | `CUBE` | Totales por todas las combinaciones | ⚠️ Limitado | ✅ | ✅ |

```sql
-- Ventas por categoría con filtro de mínimo
SELECT
    c.nombre        AS categoria,
    COUNT(p.id)     AS total_productos,
    AVG(p.precio)   AS precio_promedio,
    SUM(p.stock)    AS stock_total
FROM categorias c
LEFT JOIN productos p ON p.categoria_id = c.id
GROUP BY c.id, c.nombre
HAVING COUNT(p.id) > 5
ORDER BY precio_promedio DESC;
```

---

### 2.6 Subconsultas y CTEs

| Importancia | Técnica | Descripción | Compatibilidad |
|-------------|---------|-------------|----------------|
| 🟠 | Subconsulta en WHERE | `WHERE id IN (SELECT ...)` | ✅ Los tres |
| 🟠 | Subconsulta correlacionada | Referencia a la consulta externa | ✅ Los tres |
| 🟠 | CTE — `WITH` | Common Table Expression | ✅ Los tres |
| 🟡 | CTE Recursiva | `WITH RECURSIVE` | `WITH RECURSIVE` | `WITH RECURSIVE` | `WITH` (recursión automática) |
| 🟡 | Subquery en FROM | Tabla derivada | ✅ Los tres |

```sql
-- CTE básica (compatible en los tres)
WITH ventas_mensuales AS (
    SELECT
        EXTRACT(MONTH FROM fecha) AS mes,
        SUM(total)                AS ingresos
    FROM pedidos
    WHERE EXTRACT(YEAR FROM fecha) = 2026
    GROUP BY EXTRACT(MONTH FROM fecha)
)
SELECT mes, ingresos,
       SUM(ingresos) OVER (ORDER BY mes) AS ingresos_acumulados
FROM ventas_mensuales;

-- CTE Recursiva: jerarquía de empleados
-- 🐬 MySQL 8+ / 🐘 PostgreSQL
WITH RECURSIVE jerarquia AS (
    SELECT id, nombre, jefe_id, 1 AS nivel
    FROM empleados WHERE jefe_id IS NULL
    UNION ALL
    SELECT e.id, e.nombre, e.jefe_id, j.nivel + 1
    FROM empleados e
    INNER JOIN jerarquia j ON e.jefe_id = j.id
)
SELECT * FROM jerarquia ORDER BY nivel;

-- 🪟 SQL Server (sin RECURSIVE)
WITH jerarquia AS (
    SELECT id, nombre, jefe_id, 1 AS nivel
    FROM empleados WHERE jefe_id IS NULL
    UNION ALL
    SELECT e.id, e.nombre, e.jefe_id, j.nivel + 1
    FROM empleados e
    INNER JOIN jerarquia j ON e.jefe_id = j.id
)
SELECT * FROM jerarquia ORDER BY nivel;
```

---

### 2.7 Funciones de Ventana (Window Functions)

| Importancia | Función | Descripción | Compatibilidad |
|-------------|---------|-------------|----------------|
| 🟠 | `ROW_NUMBER()` | Número de fila por partición | ✅ Los tres |
| 🟠 | `RANK()` / `DENSE_RANK()` | Posición con/sin huecos | ✅ Los tres |
| 🟠 | `SUM() OVER (...)` | Suma acumulada | ✅ Los tres |
| 🟠 | `AVG() OVER (...)` | Promedio por ventana | ✅ Los tres |
| 🟡 | `LAG(col, n)` / `LEAD(col, n)` | Valor de fila anterior/siguiente | ✅ Los tres |
| 🟡 | `FIRST_VALUE()` / `LAST_VALUE()` | Primer/último valor de ventana | ✅ Los tres |
| 🟡 | `NTILE(n)` | Divide en N grupos iguales | ✅ Los tres |
| 🟡 | `PERCENT_RANK()` | Rango como porcentaje | ✅ Los tres |

```sql
-- Ranking de productos más vendidos por categoría
SELECT
    nombre,
    categoria_id,
    ventas,
    ROW_NUMBER()  OVER (PARTITION BY categoria_id ORDER BY ventas DESC) AS posicion,
    SUM(ventas)   OVER (PARTITION BY categoria_id)                      AS total_categoria,
    LAG(ventas)   OVER (PARTITION BY categoria_id ORDER BY ventas DESC) AS ventas_anterior
FROM productos;
```

---

## 3. ✏️ DML — Manipulación de Datos (Data Manipulation Language)

### 3.1 INSERT

| Importancia | Variante | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|----------|----------|---------------|---------------|
| 🔴 | Insert simple | ✅ Igual | ✅ Igual | ✅ Igual |
| 🔴 | Insert múltiple | ✅ Igual | ✅ Igual | ✅ Igual |
| 🟠 | Insert desde SELECT | ✅ Igual | ✅ Igual | ✅ Igual |
| 🟠 | Insert o ignorar duplicado | `INSERT IGNORE INTO ...` | `INSERT ... ON CONFLICT DO NOTHING` | `INSERT ... WHERE NOT EXISTS (...)` |
| 🟠 | Upsert (insert o actualizar) | `INSERT ... ON DUPLICATE KEY UPDATE` | `INSERT ... ON CONFLICT DO UPDATE SET` | `MERGE INTO ...` |

```sql
-- Insert simple
INSERT INTO productos (nombre, precio, stock) VALUES ('Laptop', 15999.99, 50);

-- Insert múltiple (✅ compatible)
INSERT INTO productos (nombre, precio, stock) VALUES
    ('Mouse', 299.00, 200),
    ('Teclado', 599.00, 150),
    ('Monitor', 4500.00, 30);

-- UPSERT
-- 🐬 MySQL
INSERT INTO productos (id, nombre, precio) VALUES (1, 'Laptop Pro', 17999.99)
ON DUPLICATE KEY UPDATE precio = VALUES(precio), nombre = VALUES(nombre);

-- 🐘 PostgreSQL
INSERT INTO productos (id, nombre, precio) VALUES (1, 'Laptop Pro', 17999.99)
ON CONFLICT (id) DO UPDATE SET precio = EXCLUDED.precio, nombre = EXCLUDED.nombre;

-- 🪟 SQL Server (MERGE)
MERGE INTO productos AS destino
USING (VALUES (1, 'Laptop Pro', 17999.99)) AS origen(id, nombre, precio)
    ON destino.id = origen.id
WHEN MATCHED     THEN UPDATE SET destino.precio = origen.precio
WHEN NOT MATCHED THEN INSERT (id, nombre, precio) VALUES (origen.id, origen.nombre, origen.precio);
```

---

### 3.2 UPDATE

| Importancia | Variante | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|----------|----------|---------------|---------------|
| 🔴 | Update simple | ✅ Igual | ✅ Igual | ✅ Igual |
| 🟠 | Update con JOIN | `UPDATE t1 JOIN t2 ON ... SET ...` | `UPDATE t1 SET ... FROM t2 WHERE ...` | `UPDATE t1 SET ... FROM t1 JOIN t2 ON ...` |
| 🟡 | Update con subconsulta | ✅ Los tres (sintaxis varía) | | |

```sql
-- Update con JOIN
-- 🐬 MySQL
UPDATE productos p
JOIN categorias c ON p.categoria_id = c.id
SET p.precio = p.precio * 1.10
WHERE c.nombre = 'Electrónica';

-- 🐘 PostgreSQL
UPDATE productos p
SET precio = precio * 1.10
FROM categorias c
WHERE p.categoria_id = c.id AND c.nombre = 'Electrónica';

-- 🪟 SQL Server
UPDATE p
SET p.precio = p.precio * 1.10
FROM productos p
JOIN categorias c ON p.categoria_id = c.id
WHERE c.nombre = 'Electrónica';
```

---

### 3.3 DELETE

| Importancia | Variante | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|----------|----------|---------------|---------------|
| 🔴 | Delete con condición | ✅ Igual | ✅ Igual | ✅ Igual |
| 🟠 | Delete con JOIN | `DELETE t1 FROM t1 JOIN t2 ...` | `DELETE FROM t1 USING t2 WHERE ...` | `DELETE t1 FROM t1 JOIN t2 ...` |
| 🟠 | Vaciar tabla | `TRUNCATE TABLE nombre` | ✅ Igual | ✅ Igual |
| 🟡 | Delete con retorno | ❌ | `DELETE FROM t WHERE ... RETURNING *` | `DELETE FROM t OUTPUT DELETED.* WHERE ...` |

```sql
-- ⚠️ SIEMPRE usar WHERE en DELETE
DELETE FROM productos WHERE stock = 0 AND activo = FALSE;

-- TRUNCATE (elimina todo, más rápido que DELETE sin WHERE)
TRUNCATE TABLE logs;                        -- 🐬 MySQL / 🐘 PostgreSQL
TRUNCATE TABLE logs;                        -- 🪟 SQL Server

-- Delete con retorno de filas eliminadas
-- 🐘 PostgreSQL
DELETE FROM productos WHERE activo = FALSE RETURNING id, nombre;

-- 🪟 SQL Server
DELETE FROM productos OUTPUT DELETED.id, DELETED.nombre WHERE activo = 0;
```

---

## 4. 🔒 DCL — Control de Acceso (Data Control Language)

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🟠 | Crear usuario | `CREATE USER 'user'@'host' IDENTIFIED BY 'pass'` | `CREATE USER user WITH PASSWORD 'pass'` | `CREATE LOGIN user WITH PASSWORD='pass'` |
| 🟠 | Otorgar permiso | `GRANT SELECT ON db.tabla TO 'user'@'host'` | `GRANT SELECT ON tabla TO user` | `GRANT SELECT ON tabla TO user` |
| 🟠 | Revocar permiso | `REVOKE SELECT ON db.tabla FROM 'user'@'host'` | `REVOKE SELECT ON tabla FROM user` | `REVOKE SELECT ON tabla FROM user` |
| 🟠 | Eliminar usuario | `DROP USER 'user'@'host'` | `DROP USER user` | `DROP LOGIN user` |
| 🟡 | Permisos sobre schema | `GRANT ALL ON db.* TO 'user'@'host'` | `GRANT ALL ON SCHEMA schema TO user` | `GRANT CONTROL ON SCHEMA::schema TO user` |
| 🟡 | Aplicar permisos | `FLUSH PRIVILEGES` | ✅ Automático | ✅ Automático |

---

## 5. 🔄 TCL — Control de Transacciones (Transaction Control Language)

| Importancia | Comando | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|---------|----------|---------------|---------------|
| 🔴 | Iniciar transacción | `START TRANSACTION` | `BEGIN` | `BEGIN TRANSACTION` |
| 🔴 | Confirmar cambios | `COMMIT` | ✅ Igual | ✅ Igual |
| 🔴 | Revertir cambios | `ROLLBACK` | ✅ Igual | ✅ Igual |
| 🟠 | Punto de guardado | `SAVEPOINT nombre` | ✅ Igual | `SAVE TRANSACTION nombre` |
| 🟠 | Rollback a punto | `ROLLBACK TO SAVEPOINT nombre` | ✅ Igual | `ROLLBACK TRANSACTION nombre` |
| 🟡 | Nivel de aislamiento | `SET TRANSACTION ISOLATION LEVEL ...` | ✅ Igual | ✅ Igual |

```sql
-- Transacción completa con manejo de errores

-- 🐬 MySQL
START TRANSACTION;
    UPDATE cuentas SET saldo = saldo - 500 WHERE id = 1;
    UPDATE cuentas SET saldo = saldo + 500 WHERE id = 2;
COMMIT;

-- 🐘 PostgreSQL
BEGIN;
    UPDATE cuentas SET saldo = saldo - 500 WHERE id = 1;
    UPDATE cuentas SET saldo = saldo + 500 WHERE id = 2;
COMMIT;

-- 🪟 SQL Server con manejo de error
BEGIN TRANSACTION;
    BEGIN TRY
        UPDATE cuentas SET saldo = saldo - 500 WHERE id = 1;
        UPDATE cuentas SET saldo = saldo + 500 WHERE id = 2;
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;
        THROW;
    END CATCH;
```

### Niveles de Aislamiento

| Nivel | Dirty Read | Non-Repeatable Read | Phantom Read | Compatibilidad |
|-------|-----------|--------------------|--------------|----|
| `READ UNCOMMITTED` | ✅ Posible | ✅ Posible | ✅ Posible | ✅ Los tres |
| `READ COMMITTED` | ❌ No | ✅ Posible | ✅ Posible | ✅ Los tres (default PG/SS) |
| `REPEATABLE READ` | ❌ No | ❌ No | ✅ Posible | ✅ Los tres (default MySQL) |
| `SERIALIZABLE` | ❌ No | ❌ No | ❌ No | ✅ Los tres |
| `SNAPSHOT` | ❌ No | ❌ No | ❌ No | ❌ | ⚠️ Vía MVCC | ✅ Nativo |

---

## 6. ⚡ Índices y Performance

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🔴 | Crear índice | `CREATE INDEX idx ON tabla(col)` | ✅ Igual | ✅ Igual |
| 🔴 | Índice único | `CREATE UNIQUE INDEX idx ON tabla(col)` | ✅ Igual | ✅ Igual |
| 🟠 | Eliminar índice | `DROP INDEX idx ON tabla` | `DROP INDEX idx` | `DROP INDEX tabla.idx` |
| 🟠 | Índice compuesto | `CREATE INDEX idx ON tabla(col1, col2)` | ✅ Igual | ✅ Igual |
| 🟠 | Ver plan de ejecución | `EXPLAIN SELECT ...` | `EXPLAIN ANALYZE SELECT ...` | `SET STATISTICS IO ON` + ejecución |
| 🟡 | Índice de texto completo | `FULLTEXT INDEX` | `GIN / GiST + tsvector` | `FULLTEXT INDEX` / `CONTAINS()` |
| 🟡 | Índice parcial/filtrado | ❌ | `CREATE INDEX idx ON t(col) WHERE condición` | `CREATE INDEX idx ON t(col) WHERE condición` |
| 🟢 | Índice de cobertura | `USING INDEX (col1, col2, col3)` | Incluir columnas extra | `INCLUDE (col2, col3)` en SQL Server |

```sql
-- Índice compuesto para consultas frecuentes
CREATE INDEX idx_prod_cat_precio ON productos(categoria_id, precio DESC);

-- Ver plan de ejecución
-- 🐬 MySQL
EXPLAIN SELECT * FROM productos WHERE categoria_id = 5;

-- 🐘 PostgreSQL (con estadísticas reales)
EXPLAIN ANALYZE SELECT * FROM productos WHERE categoria_id = 5;

-- 🪟 SQL Server
SET STATISTICS IO ON;
SELECT * FROM productos WHERE categoria_id = 5;
```

---

## 7. 🧩 Vistas (Views)

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🟠 | Crear vista | `CREATE VIEW nombre AS SELECT ...` | ✅ Igual | ✅ Igual |
| 🟠 | Reemplazar vista | `CREATE OR REPLACE VIEW nombre AS ...` | ✅ Igual | ⚠️ `ALTER VIEW nombre AS ...` |
| 🟠 | Eliminar vista | `DROP VIEW nombre` | ✅ Igual | ✅ Igual |
| 🟡 | Vista materializada | ❌ (simular con tabla) | `CREATE MATERIALIZED VIEW` | `CREATE INDEX VIEW` (indexed view) |
| 🟡 | Refrescar vista mat. | N/A | `REFRESH MATERIALIZED VIEW nombre` | Se actualiza automáticamente |

```sql
-- Vista de resumen de ventas
CREATE OR REPLACE VIEW v_resumen_ventas AS
SELECT
    c.nombre        AS categoria,
    COUNT(p.id)     AS total_productos,
    SUM(p.precio)   AS valor_inventario
FROM categorias c
LEFT JOIN productos p ON p.categoria_id = c.id
GROUP BY c.id, c.nombre;

-- Uso
SELECT * FROM v_resumen_ventas WHERE total_productos > 10;
```

---

## 8. ⚙️ Procedimientos Almacenados y Funciones

| Importancia | Elemento | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|----------|----------|---------------|---------------|
| 🟠 | Crear procedimiento | `CREATE PROCEDURE nombre(params) BEGIN ... END` | `CREATE OR REPLACE FUNCTION ... LANGUAGE plpgsql` | `CREATE PROCEDURE nombre @params AS BEGIN ... END` |
| 🟠 | Ejecutar procedimiento | `CALL nombre(params)` | `SELECT nombre(params)` | `EXEC nombre @params` |
| 🟡 | Crear función | `CREATE FUNCTION nombre(...) RETURNS tipo BEGIN ... END` | `CREATE OR REPLACE FUNCTION ... RETURNS tipo` | `CREATE FUNCTION nombre(...) RETURNS tipo AS BEGIN ... END` |
| 🟡 | Eliminar procedimiento | `DROP PROCEDURE nombre` | `DROP FUNCTION nombre` | `DROP PROCEDURE nombre` |

```sql
-- Procedimiento: obtener productos por categoría
-- 🐬 MySQL
DELIMITER $$
CREATE PROCEDURE sp_productos_por_categoria(IN p_categoria_id INT)
BEGIN
    SELECT id, nombre, precio, stock
    FROM productos
    WHERE categoria_id = p_categoria_id AND activo = TRUE
    ORDER BY nombre;
END$$
DELIMITER ;
CALL sp_productos_por_categoria(3);

-- 🐘 PostgreSQL
CREATE OR REPLACE FUNCTION sp_productos_por_categoria(p_categoria_id INT)
RETURNS TABLE(id INT, nombre VARCHAR, precio DECIMAL, stock INT)
LANGUAGE plpgsql AS $$
BEGIN
    RETURN QUERY
        SELECT p.id, p.nombre, p.precio, p.stock
        FROM productos p
        WHERE p.categoria_id = p_categoria_id AND p.activo = TRUE
        ORDER BY p.nombre;
END;
$$;
SELECT * FROM sp_productos_por_categoria(3);

-- 🪟 SQL Server
CREATE PROCEDURE sp_productos_por_categoria @categoria_id INT
AS
BEGIN
    SELECT id, nombre, precio, stock
    FROM productos
    WHERE categoria_id = @categoria_id AND activo = 1
    ORDER BY nombre;
END;
EXEC sp_productos_por_categoria @categoria_id = 3;
```

---

## 9. 🔔 Triggers

| Importancia | Elemento | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|----------|----------|---------------|---------------|
| 🟡 | Trigger BEFORE/AFTER | `BEFORE INSERT`, `AFTER UPDATE`, etc. | ✅ Igual | `INSTEAD OF`, `AFTER` |
| 🟡 | Referenciar fila nueva | `NEW.columna` | `NEW.columna` | `INSERTED.columna` |
| 🟡 | Referenciar fila vieja | `OLD.columna` | `OLD.columna` | `DELETED.columna` |

```sql
-- Trigger: auditoría de cambios de precio
-- 🐬 MySQL
DELIMITER $$
CREATE TRIGGER trg_auditoria_precio
AFTER UPDATE ON productos FOR EACH ROW
BEGIN
    IF OLD.precio <> NEW.precio THEN
        INSERT INTO auditoria_precios (producto_id, precio_anterior, precio_nuevo, fecha)
        VALUES (OLD.id, OLD.precio, NEW.precio, NOW());
    END IF;
END$$
DELIMITER ;

-- 🐘 PostgreSQL
CREATE OR REPLACE FUNCTION fn_auditoria_precio() RETURNS TRIGGER AS $$
BEGIN
    IF OLD.precio <> NEW.precio THEN
        INSERT INTO auditoria_precios (producto_id, precio_anterior, precio_nuevo, fecha)
        VALUES (OLD.id, OLD.precio, NEW.precio, NOW());
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_auditoria_precio
AFTER UPDATE ON productos FOR EACH ROW EXECUTE FUNCTION fn_auditoria_precio();

-- 🪟 SQL Server
CREATE TRIGGER trg_auditoria_precio ON productos AFTER UPDATE
AS BEGIN
    INSERT INTO auditoria_precios (producto_id, precio_anterior, precio_nuevo, fecha)
    SELECT d.id, d.precio, i.precio, GETDATE()
    FROM DELETED d JOIN INSERTED i ON d.id = i.id
    WHERE d.precio <> i.precio;
END;
```

---

## 10. 📅 Funciones de Fecha y Hora

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🔴 | Fecha y hora actual | `NOW()` | `NOW()` | `GETDATE()` / `SYSDATETIME()` |
| 🔴 | Solo fecha actual | `CURDATE()` | `CURRENT_DATE` | `CAST(GETDATE() AS DATE)` |
| 🟠 | Extraer parte | `YEAR(fecha)`, `MONTH(fecha)` | `EXTRACT(YEAR FROM fecha)` | `YEAR(fecha)`, `MONTH(fecha)` |
| 🟠 | Diferencia entre fechas | `DATEDIFF(d1, d2)` | `AGE(d1, d2)` / `d1 - d2` | `DATEDIFF(day, d2, d1)` |
| 🟠 | Sumar intervalo | `DATE_ADD(fecha, INTERVAL n DAY)` | `fecha + INTERVAL '7 days'` | `DATEADD(day, 7, fecha)` |
| 🟡 | Formatear fecha | `DATE_FORMAT(fecha, '%d/%m/%Y')` | `TO_CHAR(fecha, 'DD/MM/YYYY')` | `FORMAT(fecha, 'dd/MM/yyyy')` |
| 🟡 | Truncar a mes | `DATE_FORMAT(fecha, '%Y-%m-01')` | `DATE_TRUNC('month', fecha)` | `DATEADD(month, DATEDIFF(month,0,fecha), 0)` |

```sql
-- Pedidos de los últimos 30 días
-- 🐬 MySQL
SELECT * FROM pedidos WHERE fecha >= DATE_SUB(NOW(), INTERVAL 30 DAY);

-- 🐘 PostgreSQL
SELECT * FROM pedidos WHERE fecha >= NOW() - INTERVAL '30 days';

-- 🪟 SQL Server
SELECT * FROM pedidos WHERE fecha >= DATEADD(day, -30, GETDATE());
```

---

## 11. 📝 Funciones de Cadena de Texto

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🔴 | Concatenar | `CONCAT(a, b)` | `CONCAT(a, b)` / `a \|\| b` | `CONCAT(a, b)` / `a + b` |
| 🔴 | Longitud | `LENGTH(str)` / `CHAR_LENGTH(str)` | `LENGTH(str)` | `LEN(str)` |
| 🟠 | Mayúsculas/minúsculas | `UPPER()` / `LOWER()` | ✅ Igual | ✅ Igual |
| 🟠 | Recortar espacios | `TRIM()` / `LTRIM()` / `RTRIM()` | ✅ Igual | ✅ Igual |
| 🟠 | Subcadena | `SUBSTRING(str, pos, len)` | `SUBSTRING(str, pos, len)` | `SUBSTRING(str, pos, len)` |
| 🟠 | Posición | `INSTR(str, sub)` | `POSITION(sub IN str)` | `CHARINDEX(sub, str)` |
| 🟡 | Reemplazar | `REPLACE(str, old, new)` | ✅ Igual | ✅ Igual |
| 🟡 | Rellenar | `LPAD(str, n, char)` | ✅ Igual | `RIGHT(REPLICATE('0',n)+str, n)` |
| 🟡 | Invertir | `REVERSE(str)` | `REVERSE(str)` | `REVERSE(str)` |

---

## 12. 🔢 Funciones Numéricas y Condicionales

| Importancia | Función | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|---------|----------|---------------|---------------|
| 🔴 | `ROUND(n, d)` | ✅ | ✅ | ✅ |
| 🔴 | `CEIL()` / `FLOOR()` | ✅ | ✅ | ✅ |
| 🟠 | `ABS(n)` | ✅ | ✅ | ✅ |
| 🟠 | `MOD(a, b)` / `a % b` | ✅ | ✅ | `a % b` |
| 🔴 | `CASE WHEN ... END` | ✅ | ✅ | ✅ |
| 🔴 | `COALESCE(a, b, ...)` | ✅ | ✅ | ✅ |
| 🟠 | `NULLIF(a, b)` | ✅ | ✅ | ✅ |
| 🟠 | `IFNULL(val, defecto)` | `IFNULL()` | `COALESCE()` | `ISNULL()` |
| 🟡 | `IIF(cond, v1, v2)` | ❌ (usar CASE) | ❌ (usar CASE) | `IIF()` |

```sql
-- CASE para clasificar precios
SELECT
    nombre,
    precio,
    CASE
        WHEN precio < 500    THEN 'Económico'
        WHEN precio < 2000   THEN 'Medio'
        WHEN precio < 10000  THEN 'Premium'
        ELSE 'Lujo'
    END AS rango_precio
FROM productos;

-- COALESCE: primer valor no nulo
SELECT nombre, COALESCE(descripcion, 'Sin descripción') AS descripcion FROM productos;
```

---

## 13. 📋 Comandos de Administración

| Importancia | Operación | 🐬 MySQL | 🐘 PostgreSQL | 🪟 SQL Server |
|-------------|-----------|----------|---------------|---------------|
| 🟠 | Listar tablas | `SHOW TABLES` | `\dt` / `SELECT tablename FROM pg_tables` | `SELECT * FROM INFORMATION_SCHEMA.TABLES` |
| 🟠 | Ver estructura | `DESCRIBE tabla` | `\d tabla` | `EXEC sp_columns 'tabla'` |
| 🟡 | Ver índices | `SHOW INDEX FROM tabla` | `\di` / `SELECT * FROM pg_indexes` | `sys.indexes` |
| 🟡 | Ver procesos | `SHOW PROCESSLIST` | `SELECT * FROM pg_stat_activity` | `EXEC sp_who2` |
| 🟡 | Ver tamaño de tablas | `information_schema.tables` | `pg_size_pretty(pg_total_relation_size(...))` | `sp_spaceused` |
| 🟡 | Backup | `mysqldump` | `pg_dump` | `BACKUP DATABASE` |
| 🟢 | Analizar tabla | `ANALYZE TABLE nombre` | `ANALYZE nombre` | `UPDATE STATISTICS nombre` |
| 🟢 | Optimizar tabla | `OPTIMIZE TABLE nombre` | `VACUUM ANALYZE nombre` | `ALTER INDEX ALL ON nombre REBUILD` |

---

## 14. 🗺️ Mapa Mental de SQL

```
SQL
├── DDL (Definición)
│   ├── CREATE  → DATABASE, TABLE, INDEX, VIEW, PROCEDURE
│   ├── ALTER   → TABLE (ADD, DROP, MODIFY columnas y constraints)
│   └── DROP    → Eliminar objetos
│
├── DML (Manipulación)
│   ├── SELECT  → FROM, JOIN, WHERE, GROUP BY, HAVING, ORDER BY, LIMIT
│   ├── INSERT  → VALUES, SELECT, ON CONFLICT / ON DUPLICATE KEY
│   ├── UPDATE  → SET, WHERE, JOIN
│   └── DELETE  → WHERE, TRUNCATE
│
├── DCL (Control)
│   ├── GRANT   → Otorgar permisos
│   └── REVOKE  → Revocar permisos
│
├── TCL (Transacciones)
│   ├── BEGIN / START TRANSACTION
│   ├── COMMIT
│   ├── ROLLBACK
│   └── SAVEPOINT
│
└── Objetos Avanzados
    ├── Índices       → Rendimiento de consultas
    ├── Vistas        → Consultas reutilizables
    ├── Procedimientos → Lógica en el servidor
    ├── Funciones     → Cálculos reutilizables
    └── Triggers      → Automatización de eventos
```

---

## 15. ✅ Buenas Prácticas

```sql
-- 1. Usar alias descriptivos en JOINs complejos
SELECT p.nombre, c.nombre AS categoria, pr.nombre AS proveedor
FROM productos p
JOIN categorias c  ON p.categoria_id = c.id
JOIN proveedores pr ON p.proveedor_id = pr.id;

-- 2. Siempre WHERE en UPDATE y DELETE
UPDATE productos SET activo = FALSE WHERE id = 42;  -- ✅
UPDATE productos SET activo = FALSE;                 -- ⚠️ PELIGROSO

-- 3. Usar transacciones para operaciones críticas
BEGIN;
    DELETE FROM pedido_detalle WHERE pedido_id = 100;
    DELETE FROM pedidos WHERE id = 100;
COMMIT;

-- 4. Indexar columnas usadas en WHERE, JOIN y ORDER BY
CREATE INDEX idx_pedidos_cliente_fecha ON pedidos(cliente_id, fecha DESC);

-- 5. Usar EXPLAIN para detectar consultas lentas
EXPLAIN ANALYZE SELECT * FROM pedidos WHERE cliente_id = 500;

-- 6. Evitar SELECT * en producción
SELECT id, nombre, precio FROM productos;  -- ✅
SELECT * FROM productos;                   -- ⚠️ En producción

-- 7. Usar parámetros en lugar de concatenar strings (previene SQL Injection)
-- En tu ORM o driver, SIEMPRE usar prepared statements:
-- SELECT * FROM usuarios WHERE email = ?   (no concatenar directamente)
```

---

*📌 Referencia generada para MySQL 8+, PostgreSQL 14+, SQL Server 2019+ · Actualización: 2026*
