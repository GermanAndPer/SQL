# 🪟 Referencia Completa de SQL Server (T-SQL)
> Transact-SQL · SQL Server 2016+ / Azure SQL Database
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
-- Seleccionar base de datos
USE nombre_db;

-- Versión del servidor
SELECT @@VERSION;
SELECT SERVERPROPERTY('ProductVersion') AS version,
       SERVERPROPERTY('Edition')        AS edicion;

-- Configuración de sesión
SET NOCOUNT ON;           -- Suprime mensajes "N rows affected"
SET ANSI_NULLS ON;        -- NULL se comporta según estándar ANSI
SET QUOTED_IDENTIFIER ON; -- Permite comillas dobles como delimitador

-- Variables de sistema útiles
SELECT @@SERVERNAME   AS servidor;
SELECT @@ROWCOUNT     AS filas_afectadas;
SELECT @@IDENTITY     AS ultimo_identity;
SELECT SCOPE_IDENTITY() AS identity_scope;  -- ✅ Más seguro que @@IDENTITY
SELECT @@ERROR        AS ultimo_error;
SELECT @@TRANCOUNT    AS transacciones_activas;
```

---

## 2. 🗂️ DDL — Definición de Datos

### 2.1 Bases de Datos

| Importancia | Comando | Descripción |
|-------------|---------|-------------|
| 🔴 | `CREATE DATABASE nombre` | Crea una base de datos |
| 🔴 | `DROP DATABASE nombre` | Elimina una base de datos |
| 🟠 | `ALTER DATABASE nombre ...` | Modifica propiedades de la BD |
| 🟡 | `BACKUP DATABASE nombre TO DISK = 'ruta'` | Genera un backup completo |
| 🟡 | `RESTORE DATABASE nombre FROM DISK = 'ruta'` | Restaura desde backup |

```sql
-- Crear BD con opciones
CREATE DATABASE Tienda
ON PRIMARY (
    NAME = 'Tienda_Data',
    FILENAME = 'C:\Data\Tienda.mdf',
    SIZE = 100MB,
    MAXSIZE = 1GB,
    FILEGROWTH = 10MB
)
LOG ON (
    NAME = 'Tienda_Log',
    FILENAME = 'C:\Data\Tienda.ldf',
    SIZE = 25MB
);

-- Cambiar modo de recuperación
ALTER DATABASE Tienda SET RECOVERY FULL;      -- Para producción
ALTER DATABASE Tienda SET RECOVERY SIMPLE;    -- Para desarrollo

-- Verificar bases de datos existentes
SELECT name, database_id, create_date, state_desc
FROM sys.databases
ORDER BY name;

-- Eliminar si existe (SQL Server 2016+)
DROP DATABASE IF EXISTS Tienda;
```

---

### 2.2 Esquemas (Schemas)

```sql
-- Crear esquema (agrupa objetos lógicamente)
CREATE SCHEMA ventas;
CREATE SCHEMA inventario;
CREATE SCHEMA rrhh;

-- Mover tabla a otro esquema
ALTER SCHEMA ventas TRANSFER dbo.pedidos;

-- Listar esquemas
SELECT name FROM sys.schemas ORDER BY name;
```

---

### 2.3 Tablas

| Importancia | Comando | Descripción |
|-------------|---------|-------------|
| 🔴 | `CREATE TABLE` | Crea una tabla |
| 🔴 | `ALTER TABLE` | Modifica la estructura |
| 🔴 | `DROP TABLE` | Elimina la tabla |
| 🟠 | `TRUNCATE TABLE` | Vacía la tabla rápidamente |
| 🟠 | `SELECT * INTO nueva FROM vieja` | Copia estructura y datos |
| 🟡 | `CREATE TABLE #tmp` | Tabla temporal local |
| 🟡 | `CREATE TABLE ##tmp` | Tabla temporal global |

```sql
-- Creación completa de tabla
CREATE TABLE ventas.Productos (
    ProductoID      INT             NOT NULL IDENTITY(1,1),
    Codigo          VARCHAR(20)     NOT NULL,
    Nombre          NVARCHAR(200)   NOT NULL,
    Descripcion     NVARCHAR(MAX)   NULL,
    Precio          DECIMAL(12,2)   NOT NULL DEFAULT 0.00,
    Stock           INT             NOT NULL DEFAULT 0,
    Activo          BIT             NOT NULL DEFAULT 1,
    CreadoEn        DATETIME2       NOT NULL DEFAULT SYSDATETIME(),
    ModificadoEn    DATETIME2       NULL,
    CategoriaID     INT             NULL,

    CONSTRAINT PK_Productos         PRIMARY KEY CLUSTERED (ProductoID),
    CONSTRAINT UQ_Productos_Codigo  UNIQUE (Codigo),
    CONSTRAINT FK_Productos_Cat     FOREIGN KEY (CategoriaID)
                                    REFERENCES inventario.Categorias(CategoriaID)
                                    ON DELETE SET NULL ON UPDATE CASCADE,
    CONSTRAINT CK_Productos_Precio  CHECK (Precio >= 0),
    CONSTRAINT CK_Productos_Stock   CHECK (Stock >= 0)
);

-- Eliminar si existe (SQL Server 2016+)
DROP TABLE IF EXISTS ventas.Productos;

-- Verificar si existe antes de crear (clásico)
IF NOT EXISTS (SELECT 1 FROM sys.tables WHERE name = 'Productos' AND schema_id = SCHEMA_ID('ventas'))
BEGIN
    CREATE TABLE ventas.Productos (...);
END;

-- Tabla temporal local (solo en la sesión actual)
CREATE TABLE #TempResultados (
    ID      INT,
    Valor   DECIMAL(10,2)
);

-- Tabla temporal global (compartida entre sesiones)
CREATE TABLE ##TempGlobal (
    ID      INT,
    Datos   NVARCHAR(200)
);
```

---

### 2.4 ALTER TABLE — Modificar Tablas

```sql
-- Agregar columna
ALTER TABLE ventas.Productos ADD PesoKg DECIMAL(8,3) NULL;

-- Agregar columna con valor por defecto
ALTER TABLE ventas.Productos
ADD FechaVencimiento DATE NULL
    CONSTRAINT DF_Productos_FechaVenc DEFAULT NULL;

-- Modificar tipo de dato
ALTER TABLE ventas.Productos ALTER COLUMN Nombre NVARCHAR(500) NOT NULL;

-- Renombrar columna
EXEC sp_rename 'ventas.Productos.Nombre', 'NombreProducto', 'COLUMN';

-- Renombrar tabla
EXEC sp_rename 'ventas.Productos', 'Articulos';

-- Eliminar columna
ALTER TABLE ventas.Productos DROP COLUMN PesoKg;

-- Eliminar constraint
ALTER TABLE ventas.Productos DROP CONSTRAINT CK_Productos_Precio;

-- Agregar constraint
ALTER TABLE ventas.Productos
ADD CONSTRAINT CK_Productos_Precio CHECK (Precio BETWEEN 0 AND 9999999);
```

---

### 2.5 Tipos de Datos

| Categoría | Tipo | Descripción | Rango / Notas |
|-----------|------|-------------|---------------|
| **Enteros** | `TINYINT` | Entero sin signo pequeño | 0 – 255 |
| | `SMALLINT` | Entero pequeño | -32,768 – 32,767 |
| | `INT` | Entero estándar | -2.1B – 2.1B |
| | `BIGINT` | Entero grande | ±9.2 × 10¹⁸ |
| **Decimales** | `DECIMAL(p,s)` / `NUMERIC(p,s)` | Exacto | Hasta 38 dígitos |
| | `MONEY` | Monetario | ±922 trillones, 4 decimales |
| | `SMALLMONEY` | Monetario pequeño | ±214,748, 4 decimales |
| | `FLOAT(n)` | Flotante | Aproximado |
| | `REAL` | Flotante simple | 7 dígitos precisión |
| **Texto** | `CHAR(n)` | Texto fijo (ASCII) | Hasta 8,000 chars |
| | `VARCHAR(n)` | Texto variable (ASCII) | Hasta 8,000 / `MAX` = 2GB |
| | `NCHAR(n)` | Texto fijo (Unicode) | Hasta 4,000 chars |
| | `NVARCHAR(n)` | Texto variable (Unicode) | Hasta 4,000 / `MAX` = 2GB |
| | `TEXT` | ⚠️ Obsoleto | Usar `VARCHAR(MAX)` |
| **Fecha/Hora** | `DATE` | Solo fecha | 0001-01-01 – 9999-12-31 |
| | `TIME(n)` | Solo hora | Precisión 0-7 |
| | `DATETIME` | Fecha y hora | Precisión ~3ms |
| | `DATETIME2(n)` | Fecha y hora extendida | Precisión hasta 100ns ✅ Preferido |
| | `DATETIMEOFFSET` | Con zona horaria | Para apps globales |
| | `SMALLDATETIME` | Fecha y hora compacta | Precisión 1 minuto |
| **Otros** | `BIT` | Booleano | 0 / 1 / NULL |
| | `UNIQUEIDENTIFIER` | GUID/UUID | `NEWID()` / `NEWSEQUENTIALID()` |
| | `VARBINARY(MAX)` | Datos binarios | Hasta 2GB |
| | `XML` | Documento XML | Nativo con XQuery |
| | `JSON` | ⚠️ No tipo nativo | Usar `NVARCHAR(MAX)` + `ISJSON()` |
| | `GEOGRAPHY` | Datos geoespaciales | Latitud, longitud |
| | `HIERARCHYID` | Jerarquías | Árboles y grafos |
| | `SQL_VARIANT` | Tipo genérico | Evitar en producción |

---

## 3. 🔍 DQL — Consulta de Datos

### 3.1 SELECT Fundamental

```sql
-- Estructura completa de un SELECT
SELECT [TOP n | TOP n PERCENT] [DISTINCT]
    columnas | expresiones | *
FROM tabla [AS alias]
    [JOIN ...]
[WHERE condicion]
[GROUP BY columnas]
[HAVING condicion_de_grupo]
[ORDER BY columnas [ASC | DESC]]
[OFFSET n ROWS FETCH NEXT m ROWS ONLY];  -- Paginación
```

```sql
-- Ejemplo completo
SELECT TOP 50
    p.ProductoID,
    p.Nombre,
    p.Precio,
    p.Precio * 1.16                             AS PrecioConIVA,
    c.Nombre                                    AS Categoria,
    ISNULL(p.Stock, 0)                          AS Stock,
    FORMAT(p.CreadoEn, 'dd/MM/yyyy')            AS FechaAlta,
    CASE
        WHEN p.Precio < 500   THEN 'Económico'
        WHEN p.Precio < 2000  THEN 'Medio'
        ELSE 'Premium'
    END                                         AS Rango
FROM ventas.Productos p
INNER JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID
WHERE p.Activo = 1
  AND p.Stock > 0
ORDER BY p.Precio DESC;
```

---

### 3.2 Paginación

```sql
-- Paginación estándar (SQL Server 2012+)
SELECT ProductoID, Nombre, Precio
FROM ventas.Productos
ORDER BY Nombre
OFFSET 20 ROWS              -- Saltar las primeras 20
FETCH NEXT 10 ROWS ONLY;    -- Tomar las siguientes 10

-- Con variables (página dinámica)
DECLARE @Pagina    INT = 3;
DECLARE @TamanoPag INT = 10;

SELECT ProductoID, Nombre, Precio
FROM ventas.Productos
ORDER BY Nombre
OFFSET (@Pagina - 1) * @TamanoPag ROWS
FETCH NEXT @TamanoPag ROWS ONLY;

-- TOP (solo primeras N filas, sin paginación)
SELECT TOP 10 * FROM ventas.Productos ORDER BY Precio DESC;

-- TOP con porcentaje
SELECT TOP 5 PERCENT * FROM ventas.Productos ORDER BY Precio DESC;

-- TOP con empates (WITH TIES)
SELECT TOP 3 WITH TIES Nombre, Precio
FROM ventas.Productos
ORDER BY Precio DESC;  -- Incluye todos los que empatan en el 3er precio
```

---

### 3.3 Filtros — WHERE

| Importancia | Operador | Ejemplo |
|-------------|----------|---------|
| 🔴 | `=`, `<>`, `<`, `>`, `<=`, `>=` | `WHERE Precio > 100` |
| 🔴 | `AND`, `OR`, `NOT` | `WHERE Activo = 1 AND Stock > 0` |
| 🔴 | `IS NULL` / `IS NOT NULL` | `WHERE Descripcion IS NULL` |
| 🔴 | `IN (...)` | `WHERE CategoriaID IN (1, 3, 5)` |
| 🔴 | `BETWEEN ... AND ...` | `WHERE Precio BETWEEN 100 AND 999` |
| 🔴 | `LIKE` | `WHERE Nombre LIKE 'Lap%'` |
| 🟠 | `NOT IN (...)` | `WHERE Estado NOT IN ('Cancelado')` |
| 🟠 | `EXISTS (subconsulta)` | `WHERE EXISTS (SELECT 1 FROM ...)` |
| 🟡 | `CONTAINS()` | Búsqueda full-text |
| 🟡 | `FREETEXT()` | Búsqueda semántica full-text |

```sql
-- LIKE: comodines
WHERE Nombre LIKE 'A%'      -- Empieza con A
WHERE Nombre LIKE '%pro%'   -- Contiene "pro"
WHERE Nombre LIKE '_ata%'   -- Segunda letra 'a', tercera 't'...
WHERE Codigo LIKE '[A-Z]%'  -- Empieza con letra mayúscula
WHERE Codigo LIKE '[^0-9]%' -- No empieza con número

-- BETWEEN (inclusivo)
WHERE Precio BETWEEN 500 AND 2000   -- Incluye 500 y 2000
WHERE Fecha BETWEEN '2026-01-01' AND '2026-12-31'

-- EXISTS vs IN (EXISTS es más eficiente con subqueries grandes)
SELECT * FROM ventas.Clientes c
WHERE EXISTS (
    SELECT 1 FROM ventas.Pedidos p WHERE p.ClienteID = c.ClienteID
);
```

---

### 3.4 JOINs

| Importancia | Tipo | Descripción |
|-------------|------|-------------|
| 🔴 | `INNER JOIN` | Solo coincidencias en ambas tablas |
| 🔴 | `LEFT JOIN` | Todas las filas izquierda + coincidencias |
| 🟠 | `RIGHT JOIN` | Todas las filas derecha + coincidencias |
| 🟠 | `FULL OUTER JOIN` | Todas las filas de ambas tablas |
| 🟡 | `CROSS JOIN` | Producto cartesiano |
| 🟡 | `SELF JOIN` | La tabla unida consigo misma |

```sql
-- INNER JOIN: solo coincidencias
SELECT p.Nombre, c.Nombre AS Categoria
FROM ventas.Productos p
INNER JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID;

-- LEFT JOIN: todos los productos, tengan categoría o no
SELECT p.Nombre, ISNULL(c.Nombre, 'Sin categoría') AS Categoria
FROM ventas.Productos p
LEFT JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID;

-- FULL OUTER JOIN: todos los productos Y todas las categorías
SELECT p.Nombre, c.Nombre AS Categoria
FROM ventas.Productos p
FULL OUTER JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID;

-- SELF JOIN: empleados con su jefe
SELECT e.Nombre AS Empleado, j.Nombre AS Jefe
FROM rrhh.Empleados e
LEFT JOIN rrhh.Empleados j ON e.JefeID = j.EmpleadoID;

-- JOIN múltiple
SELECT p.Nombre, c.Nombre AS Cat, pr.Nombre AS Proveedor
FROM ventas.Productos p
INNER JOIN inventario.Categorias  c  ON p.CategoriaID  = c.CategoriaID
INNER JOIN inventario.Proveedores pr ON p.ProveedorID  = pr.ProveedorID;
```

---

### 3.5 Agregación y GROUP BY

```sql
-- Resumen de ventas por categoría
SELECT
    c.Nombre                        AS Categoria,
    COUNT(p.ProductoID)             AS TotalProductos,
    SUM(p.Stock)                    AS StockTotal,
    AVG(p.Precio)                   AS PrecioPromedio,
    MIN(p.Precio)                   AS PrecioMinimo,
    MAX(p.Precio)                   AS PrecioMaximo,
    SUM(p.Precio * p.Stock)         AS ValorInventario
FROM inventario.Categorias c
LEFT JOIN ventas.Productos p ON p.CategoriaID = c.CategoriaID
WHERE p.Activo = 1
GROUP BY c.CategoriaID, c.Nombre
HAVING COUNT(p.ProductoID) >= 5
ORDER BY ValorInventario DESC;

-- ROLLUP: subtotales y total general
SELECT
    ISNULL(c.Nombre, 'TOTAL GENERAL') AS Categoria,
    SUM(p.Stock)                       AS StockTotal
FROM ventas.Productos p
JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID
GROUP BY ROLLUP(c.Nombre);

-- CUBE: totales por todas las combinaciones
SELECT Region, Categoria, SUM(Ventas) AS Total
FROM dbo.Datos
GROUP BY CUBE(Region, Categoria);

-- GROUPING SETS: control preciso de agrupaciones
SELECT Region, Categoria, SUM(Ventas)
FROM dbo.Datos
GROUP BY GROUPING SETS ((Region), (Categoria), ());
```

---

### 3.6 CTEs y Subconsultas

```sql
-- CTE simple
WITH VentasMensuales AS (
    SELECT
        YEAR(Fecha)  AS Anio,
        MONTH(Fecha) AS Mes,
        SUM(Total)   AS Ingresos
    FROM ventas.Pedidos
    GROUP BY YEAR(Fecha), MONTH(Fecha)
)
SELECT Anio, Mes, Ingresos,
       SUM(Ingresos) OVER (PARTITION BY Anio ORDER BY Mes) AS AcumuladoAnual
FROM VentasMensuales
ORDER BY Anio, Mes;

-- CTEs múltiples
WITH
Activos AS (
    SELECT * FROM ventas.Productos WHERE Activo = 1
),
Caros AS (
    SELECT * FROM Activos WHERE Precio > 5000
)
SELECT * FROM Caros ORDER BY Precio DESC;

-- CTE Recursiva: jerarquía organizacional
WITH JerarquiaEmpleados AS (
    -- Caso base: directores (sin jefe)
    SELECT EmpleadoID, Nombre, JefeID, 0 AS Nivel,
           CAST(Nombre AS NVARCHAR(500)) AS Ruta
    FROM rrhh.Empleados
    WHERE JefeID IS NULL

    UNION ALL

    -- Caso recursivo: empleados con jefe
    SELECT e.EmpleadoID, e.Nombre, e.JefeID, j.Nivel + 1,
           CAST(j.Ruta + ' > ' + e.Nombre AS NVARCHAR(500))
    FROM rrhh.Empleados e
    INNER JOIN JerarquiaEmpleados j ON e.JefeID = j.EmpleadoID
)
SELECT Nivel, Nombre, Ruta
FROM JerarquiaEmpleados
ORDER BY Ruta;
```

---

### 3.7 Window Functions (Funciones de Ventana)

| Importancia | Función | Descripción |
|-------------|---------|-------------|
| 🟠 | `ROW_NUMBER()` | Número de fila único por partición |
| 🟠 | `RANK()` | Posición con huecos en empates |
| 🟠 | `DENSE_RANK()` | Posición sin huecos en empates |
| 🟠 | `SUM() OVER()` | Suma acumulada o por partición |
| 🟠 | `AVG() OVER()` | Promedio por ventana |
| 🟡 | `LAG(col, n, default)` | Valor de N filas anteriores |
| 🟡 | `LEAD(col, n, default)` | Valor de N filas siguientes |
| 🟡 | `FIRST_VALUE()` | Primer valor de la ventana |
| 🟡 | `LAST_VALUE()` | Último valor de la ventana |
| 🟡 | `NTILE(n)` | Divide en N grupos iguales |
| 🟡 | `PERCENT_RANK()` | Rango como porcentaje (0.0 – 1.0) |
| 🟡 | `CUME_DIST()` | Distribución acumulada |

```sql
-- Análisis completo de ventas por vendedor
SELECT
    Vendedor,
    Mes,
    Ventas,
    ROW_NUMBER()   OVER (PARTITION BY Mes ORDER BY Ventas DESC)  AS Posicion,
    RANK()         OVER (PARTITION BY Mes ORDER BY Ventas DESC)  AS Ranking,
    DENSE_RANK()   OVER (PARTITION BY Mes ORDER BY Ventas DESC)  AS RankingDenso,
    NTILE(4)       OVER (PARTITION BY Mes ORDER BY Ventas DESC)  AS Cuartil,
    SUM(Ventas)    OVER (PARTITION BY Mes)                       AS TotalMes,
    SUM(Ventas)    OVER (PARTITION BY Mes ORDER BY Ventas DESC)  AS AcumMes,
    AVG(Ventas)    OVER (PARTITION BY Mes)                       AS PromedioMes,
    LAG(Ventas,1,0)  OVER (PARTITION BY Vendedor ORDER BY Mes)   AS VentasMesAnterior,
    LEAD(Ventas,1,0) OVER (PARTITION BY Vendedor ORDER BY Mes)   AS VentasMesSiguiente
FROM ventas.ResumenMensual;

-- Eliminar duplicados con ROW_NUMBER (muy común)
WITH Duplicados AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY Email ORDER BY CreadoEn DESC) AS RN
    FROM ventas.Clientes
)
DELETE FROM Duplicados WHERE RN > 1;
```

---

### 3.8 PIVOT y UNPIVOT

```sql
-- PIVOT: convertir filas en columnas
SELECT *
FROM (
    SELECT Categoria, MONTH(Fecha) AS Mes, Total
    FROM ventas.Pedidos p
    JOIN ventas.Productos pr ON p.ProductoID = pr.ProductoID
    JOIN inventario.Categorias c ON pr.CategoriaID = c.CategoriaID
) AS Origen
PIVOT (
    SUM(Total)
    FOR Mes IN ([1],[2],[3],[4],[5],[6],[7],[8],[9],[10],[11],[12])
) AS Resultado;

-- UNPIVOT: convertir columnas en filas
SELECT Mes, Ventas
FROM (SELECT Enero, Febrero, Marzo FROM dbo.ResumenAnual) AS Origen
UNPIVOT (
    Ventas FOR Mes IN (Enero, Febrero, Marzo)
) AS Resultado;
```

---

## 4. ✏️ DML — Manipulación de Datos

### 4.1 INSERT

```sql
-- Insert simple
INSERT INTO ventas.Productos (Nombre, Precio, Stock, CategoriaID)
VALUES ('Laptop Dell XPS', 25999.99, 15, 3);

-- Insert múltiple
INSERT INTO ventas.Productos (Nombre, Precio, Stock)
VALUES
    ('Mouse Logitech', 349.00, 100),
    ('Teclado Mecánico', 899.00,  75),
    ('Monitor 4K',      6499.00,  20);

-- Insert desde SELECT
INSERT INTO ventas.ProductosArchivo (Nombre, Precio, FechaArchivo)
SELECT Nombre, Precio, SYSDATETIME()
FROM ventas.Productos
WHERE Activo = 0;

-- Insert con OUTPUT: capturar los IDs generados
DECLARE @NuevosIDs TABLE (ProductoID INT, Nombre NVARCHAR(200));

INSERT INTO ventas.Productos (Nombre, Precio)
OUTPUT INSERTED.ProductoID, INSERTED.Nombre INTO @NuevosIDs
VALUES ('Webcam HD', 799.00);

SELECT * FROM @NuevosIDs;
```

---

### 4.2 UPDATE

```sql
-- Update simple
UPDATE ventas.Productos
SET Precio = Precio * 1.10,
    ModificadoEn = SYSDATETIME()
WHERE CategoriaID = 3;

-- Update con JOIN
UPDATE p
SET p.Precio        = p.Precio * 1.05,
    p.ModificadoEn  = SYSDATETIME()
FROM ventas.Productos p
INNER JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID
WHERE c.Nombre = 'Electrónica' AND p.Activo = 1;

-- Update con OUTPUT: ver valores antes y después
UPDATE ventas.Productos
SET Precio = Precio * 0.90
OUTPUT
    DELETED.ProductoID,
    DELETED.Nombre,
    DELETED.Precio    AS PrecioAnterior,
    INSERTED.Precio   AS PrecioNuevo
WHERE Stock > 100;

-- Update con subconsulta
UPDATE ventas.Productos
SET Precio = (
    SELECT AVG(Precio) * 0.95
    FROM ventas.Productos
    WHERE CategoriaID = p.CategoriaID
)
FROM ventas.Productos p
WHERE p.Activo = 0;
```

---

### 4.3 DELETE y TRUNCATE

```sql
-- Delete con condición (SIEMPRE usar WHERE)
DELETE FROM ventas.Productos WHERE Activo = 0 AND Stock = 0;

-- Delete con JOIN
DELETE p
FROM ventas.Productos p
INNER JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID
WHERE c.Activo = 0;

-- Delete con OUTPUT: recuperar filas eliminadas
DECLARE @Eliminados TABLE (ProductoID INT, Nombre NVARCHAR(200));

DELETE FROM ventas.Productos
OUTPUT DELETED.ProductoID, DELETED.Nombre INTO @Eliminados
WHERE FechaVencimiento < CAST(GETDATE() AS DATE);

SELECT * FROM @Eliminados;

-- TRUNCATE: vacía la tabla (más rápido, reinicia IDENTITY)
TRUNCATE TABLE ventas.LogsTemporales;
-- ⚠️ No se puede en tablas con FK activas, no activa triggers
```

---

### 4.4 MERGE (Upsert)

```sql
-- MERGE: INSERT, UPDATE y DELETE en una sola operación
MERGE INTO ventas.Productos AS Destino
USING (
    SELECT ProductoID, Nombre, Precio, Stock
    FROM staging.ProductosNuevos
) AS Origen ON Destino.ProductoID = Origen.ProductoID

WHEN MATCHED AND (Destino.Precio <> Origen.Precio OR Destino.Stock <> Origen.Stock)
    THEN UPDATE SET
        Destino.Precio       = Origen.Precio,
        Destino.Stock        = Origen.Stock,
        Destino.ModificadoEn = SYSDATETIME()

WHEN NOT MATCHED BY TARGET
    THEN INSERT (Nombre, Precio, Stock, CreadoEn)
         VALUES (Origen.Nombre, Origen.Precio, Origen.Stock, SYSDATETIME())

WHEN NOT MATCHED BY SOURCE
    THEN UPDATE SET Destino.Activo = 0

OUTPUT
    $action                 AS Accion,
    INSERTED.ProductoID,
    INSERTED.Nombre;
```

---

## 5. 🔄 Transacciones (TCL)

```sql
-- Transacción básica
BEGIN TRANSACTION;
    UPDATE cuentas SET Saldo = Saldo - 1000 WHERE CuentaID = 1;
    UPDATE cuentas SET Saldo = Saldo + 1000 WHERE CuentaID = 2;
COMMIT TRANSACTION;

-- Transacción con manejo de errores (TRY/CATCH)
BEGIN TRANSACTION;
BEGIN TRY
    INSERT INTO ventas.Pedidos (ClienteID, Total, Fecha)
    VALUES (42, 1500.00, SYSDATETIME());

    DECLARE @PedidoID INT = SCOPE_IDENTITY();

    INSERT INTO ventas.PedidoDetalle (PedidoID, ProductoID, Cantidad, Precio)
    VALUES (@PedidoID, 7, 2, 750.00);

    COMMIT TRANSACTION;
    SELECT 'Transacción exitosa' AS Resultado;
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    SELECT
        ERROR_NUMBER()    AS NumeroError,
        ERROR_SEVERITY()  AS Severidad,
        ERROR_STATE()     AS Estado,
        ERROR_PROCEDURE() AS Procedimiento,
        ERROR_LINE()      AS Linea,
        ERROR_MESSAGE()   AS Mensaje;
END CATCH;

-- SAVEPOINT
BEGIN TRANSACTION;
    INSERT INTO dbo.Tabla1 VALUES (1);
    SAVE TRANSACTION PuntoGuardado1;

    INSERT INTO dbo.Tabla2 VALUES (2);   -- Si esto falla...
    ROLLBACK TRANSACTION PuntoGuardado1; -- ...solo deshace hasta aquí

COMMIT TRANSACTION;

-- Nivel de aislamiento
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;   -- Default
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; -- Dirty reads permitidos
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;     -- Más restrictivo
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;         -- MVCC, sin bloqueos lectores
```

---

## 6. ⚡ Índices y Rendimiento

### 6.1 Tipos de Índices

| Importancia | Tipo | Descripción |
|-------------|------|-------------|
| 🔴 | `CLUSTERED` | Ordena físicamente la tabla (1 por tabla) |
| 🔴 | `NONCLUSTERED` | Estructura separada con puntero a la fila |
| 🟠 | `UNIQUE` | Garantiza unicidad de valores |
| 🟠 | `INCLUDE` (índice de cobertura) | Añade columnas extra al índice |
| 🟡 | `FILTERED` | Índice sobre un subconjunto de filas |
| 🟡 | `FULLTEXT` | Búsqueda de texto completo |
| 🟡 | `COLUMNSTORE` | Para analítica y DW (compresión columnar) |
| 🟢 | `SPATIAL` | Datos geoespaciales |

```sql
-- Índice clustered (la PK lo crea automáticamente)
CREATE CLUSTERED INDEX CIX_Pedidos_Fecha ON ventas.Pedidos(Fecha);

-- Índice nonclustered simple
CREATE NONCLUSTERED INDEX IX_Productos_Categoria
ON ventas.Productos(CategoriaID);

-- Índice compuesto
CREATE NONCLUSTERED INDEX IX_Pedidos_Cliente_Fecha
ON ventas.Pedidos(ClienteID, Fecha DESC);

-- Índice de cobertura (INCLUDE evita Key Lookup)
CREATE NONCLUSTERED INDEX IX_Productos_Categoria_Cobertura
ON ventas.Productos(CategoriaID)
INCLUDE (Nombre, Precio, Stock);

-- Índice filtrado (solo filas activas)
CREATE NONCLUSTERED INDEX IX_Productos_Activos
ON ventas.Productos(Precio)
WHERE Activo = 1;

-- Índice único
CREATE UNIQUE NONCLUSTERED INDEX UQ_Clientes_Email
ON ventas.Clientes(Email)
WHERE Email IS NOT NULL;

-- Eliminar índice
DROP INDEX IF EXISTS IX_Productos_Categoria ON ventas.Productos;

-- Reconstruir índice (desfragmentación)
ALTER INDEX IX_Productos_Categoria ON ventas.Productos REBUILD;
ALTER INDEX ALL ON ventas.Productos REBUILD;

-- Reorganizar (menos agresivo que REBUILD)
ALTER INDEX IX_Productos_Categoria ON ventas.Productos REORGANIZE;
```

---

### 6.2 Plan de Ejecución y Diagnóstico

```sql
-- Ver plan de ejecución estimado (sin ejecutar)
SET SHOWPLAN_ALL ON;
SELECT * FROM ventas.Productos WHERE CategoriaID = 3;
SET SHOWPLAN_ALL OFF;

-- Estadísticas de I/O y tiempo
SET STATISTICS IO ON;
SET STATISTICS TIME ON;
SELECT * FROM ventas.Productos WHERE CategoriaID = 3;
SET STATISTICS IO OFF;
SET STATISTICS TIME OFF;

-- Fragmentación de índices
SELECT
    i.name          AS Indice,
    ips.avg_fragmentation_in_percent AS Fragmentacion,
    ips.page_count  AS Paginas
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'SAMPLED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 5
ORDER BY ips.avg_fragmentation_in_percent DESC;

-- Índices no utilizados
SELECT
    OBJECT_NAME(i.object_id)   AS Tabla,
    i.name                     AS Indice,
    ius.user_seeks,
    ius.user_scans,
    ius.user_lookups
FROM sys.indexes i
LEFT JOIN sys.dm_db_index_usage_stats ius
    ON i.object_id = ius.object_id AND i.index_id = ius.index_id
WHERE OBJECTPROPERTY(i.object_id, 'IsUserTable') = 1
  AND ius.user_seeks + ius.user_scans + ius.user_lookups = 0
ORDER BY i.name;

-- Consultas más costosas (DMV)
SELECT TOP 10
    qs.total_elapsed_time / qs.execution_count / 1000 AS PromMS,
    qs.execution_count,
    SUBSTRING(st.text, (qs.statement_start_offset/2)+1, 200) AS Query
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY PromMS DESC;
```

---

## 7. 🧩 Vistas (Views)

```sql
-- Vista simple
CREATE VIEW ventas.v_ProductosActivos AS
SELECT p.ProductoID, p.Nombre, p.Precio, p.Stock, c.Nombre AS Categoria
FROM ventas.Productos p
JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID
WHERE p.Activo = 1;

-- Modificar vista
ALTER VIEW ventas.v_ProductosActivos AS
SELECT p.ProductoID, p.Nombre, p.Precio, p.Stock,
       c.Nombre AS Categoria, pr.Nombre AS Proveedor
FROM ventas.Productos p
JOIN inventario.Categorias  c  ON p.CategoriaID = c.CategoriaID
JOIN inventario.Proveedores pr ON p.ProveedorID = pr.ProveedorID
WHERE p.Activo = 1;

-- Eliminar vista
DROP VIEW IF EXISTS ventas.v_ProductosActivos;

-- Vista indexada (materializada en SQL Server)
-- Requiere WITH SCHEMABINDING
CREATE VIEW ventas.v_ResumenCategoria
WITH SCHEMABINDING AS
SELECT
    c.CategoriaID,
    c.Nombre                AS Categoria,
    COUNT_BIG(*)            AS TotalProductos,
    SUM(p.Precio)           AS SumaPrecio
FROM ventas.Productos p
JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID
GROUP BY c.CategoriaID, c.Nombre;

-- Crear índice clustered sobre la vista (la "materializa")
CREATE UNIQUE CLUSTERED INDEX CIX_v_ResumenCategoria
ON ventas.v_ResumenCategoria(CategoriaID);
```

---

## 8. ⚙️ Procedimientos Almacenados

```sql
-- Procedimiento básico
CREATE OR ALTER PROCEDURE ventas.sp_ObtenerProductos
    @CategoriaID INT     = NULL,
    @SoloActivos BIT     = 1,
    @Pagina      INT     = 1,
    @TamanoPag   INT     = 20
AS
BEGIN
    SET NOCOUNT ON;

    SELECT
        p.ProductoID, p.Nombre, p.Precio, p.Stock,
        c.Nombre AS Categoria
    FROM ventas.Productos p
    JOIN inventario.Categorias c ON p.CategoriaID = c.CategoriaID
    WHERE (@CategoriaID IS NULL OR p.CategoriaID = @CategoriaID)
      AND (@SoloActivos = 0 OR p.Activo = 1)
    ORDER BY p.Nombre
    OFFSET (@Pagina - 1) * @TamanoPag ROWS
    FETCH NEXT @TamanoPag ROWS ONLY;
END;

-- Ejecutar
EXEC ventas.sp_ObtenerProductos;
EXEC ventas.sp_ObtenerProductos @CategoriaID = 3, @Pagina = 2;

-- Procedimiento con parámetros de salida
CREATE OR ALTER PROCEDURE ventas.sp_InsertarProducto
    @Nombre      NVARCHAR(200),
    @Precio      DECIMAL(12,2),
    @CategoriaID INT,
    @ProductoID  INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    INSERT INTO ventas.Productos (Nombre, Precio, CategoriaID, CreadoEn)
    VALUES (@Nombre, @Precio, @CategoriaID, SYSDATETIME());

    SET @ProductoID = SCOPE_IDENTITY();
END;

-- Ejecutar con OUTPUT
DECLARE @NuevoID INT;
EXEC ventas.sp_InsertarProducto
    @Nombre      = 'Auriculares BT',
    @Precio      = 1299.00,
    @CategoriaID = 3,
    @ProductoID  = @NuevoID OUTPUT;

SELECT @NuevoID AS IDGenerado;

-- Eliminar procedimiento
DROP PROCEDURE IF EXISTS ventas.sp_ObtenerProductos;
```

---

## 9. 🔧 Funciones

```sql
-- Función escalar (devuelve un valor)
CREATE OR ALTER FUNCTION dbo.fn_PrecioConIVA (@Precio DECIMAL(12,2))
RETURNS DECIMAL(12,2)
AS
BEGIN
    RETURN @Precio * 1.16;
END;

-- Uso
SELECT Nombre, Precio, dbo.fn_PrecioConIVA(Precio) AS PrecioIVA
FROM ventas.Productos;

-- Función de tabla en línea (ITVF) - más eficiente
CREATE OR ALTER FUNCTION ventas.fn_ProductosPorCategoria (@CategoriaID INT)
RETURNS TABLE
AS
RETURN (
    SELECT p.ProductoID, p.Nombre, p.Precio, p.Stock
    FROM ventas.Productos p
    WHERE p.CategoriaID = @CategoriaID AND p.Activo = 1
);

-- Uso con CROSS APPLY
SELECT c.Nombre AS Categoria, p.*
FROM inventario.Categorias c
CROSS APPLY ventas.fn_ProductosPorCategoria(c.CategoriaID) p;

-- Función de tabla de múltiples instrucciones (MSTVF)
CREATE OR ALTER FUNCTION dbo.fn_JerarquiaEmpleado (@EmpleadoID INT)
RETURNS @Resultado TABLE (EmpleadoID INT, Nombre NVARCHAR(200), Nivel INT)
AS
BEGIN
    WITH Jerarquia AS (
        SELECT EmpleadoID, Nombre, 0 AS Nivel FROM rrhh.Empleados WHERE EmpleadoID = @EmpleadoID
        UNION ALL
        SELECT e.EmpleadoID, e.Nombre, j.Nivel + 1
        FROM rrhh.Empleados e JOIN Jerarquia j ON e.JefeID = j.EmpleadoID
    )
    INSERT INTO @Resultado SELECT * FROM Jerarquia;
    RETURN;
END;
```

---

## 10. 🔔 Triggers

```sql
-- Trigger AFTER INSERT/UPDATE/DELETE
CREATE OR ALTER TRIGGER ventas.trg_AuditoriaPrecio
ON ventas.Productos
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    -- Solo si cambió el precio
    IF UPDATE(Precio)
    BEGIN
        INSERT INTO ventas.AuditoriaPrecio (ProductoID, PrecioAnterior, PrecioNuevo, Fecha, Usuario)
        SELECT
            d.ProductoID,
            d.Precio        AS PrecioAnterior,
            i.Precio        AS PrecioNuevo,
            SYSDATETIME(),
            SYSTEM_USER
        FROM DELETED d
        JOIN INSERTED i ON d.ProductoID = i.ProductoID
        WHERE d.Precio <> i.Precio;
    END;
END;

-- Trigger INSTEAD OF (vistas actualizables)
CREATE OR ALTER TRIGGER ventas.trg_InsteadOf_v_Productos
ON ventas.v_ProductosActivos
INSTEAD OF INSERT
AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO ventas.Productos (Nombre, Precio, Stock, CreadoEn)
    SELECT Nombre, Precio, Stock, SYSDATETIME()
    FROM INSERTED;
END;

-- Deshabilitar / Habilitar trigger
DISABLE TRIGGER ventas.trg_AuditoriaPrecio ON ventas.Productos;
ENABLE  TRIGGER ventas.trg_AuditoriaPrecio ON ventas.Productos;

-- Eliminar trigger
DROP TRIGGER IF EXISTS ventas.trg_AuditoriaPrecio;
```

---

## 11. 📅 Funciones de Fecha y Hora

| Importancia | Función | Descripción | Ejemplo |
|-------------|---------|-------------|---------|
| 🔴 | `GETDATE()` | Fecha y hora actual (DATETIME) | `2026-05-03 14:30:00` |
| 🔴 | `SYSDATETIME()` | Fecha y hora actual (DATETIME2) | Mayor precisión ✅ |
| 🔴 | `CAST(... AS DATE)` | Extraer solo la fecha | |
| 🟠 | `DATEADD(parte, n, fecha)` | Sumar/restar a una fecha | `DATEADD(day, 30, GETDATE())` |
| 🟠 | `DATEDIFF(parte, f1, f2)` | Diferencia entre fechas | `DATEDIFF(day, Inicio, Fin)` |
| 🟠 | `YEAR()` / `MONTH()` / `DAY()` | Extraer componentes | |
| 🟠 | `DATEPART(parte, fecha)` | Extraer parte específica | `DATEPART(week, GETDATE())` |
| 🟠 | `FORMAT(fecha, 'formato')` | Formatear fecha | `FORMAT(GETDATE(), 'dd/MM/yyyy')` |
| 🟡 | `EOMONTH(fecha)` | Último día del mes | |
| 🟡 | `DATEFROMPARTS(y,m,d)` | Construir fecha | `DATEFROMPARTS(2026,12,31)` |
| 🟡 | `ISDATE(valor)` | Verificar si es fecha válida | Retorna 1 o 0 |

```sql
-- Partes de fecha válidas: year, quarter, month, dayofyear,
--                          day, week, weekday, hour, minute, second, millisecond

-- Ejemplos prácticos
SELECT
    GETDATE()                               AS Ahora,
    SYSDATETIME()                           AS AhoraPresicion,
    CAST(GETDATE() AS DATE)                 AS SoloFecha,
    CAST(GETDATE() AS TIME)                 AS SoloHora,
    DATEADD(month, -1, GETDATE())           AS HaceUnMes,
    DATEADD(year,   1, GETDATE())           AS ProximoAnio,
    DATEDIFF(day, '2026-01-01', GETDATE())  AS DiasDesdeInicio,
    EOMONTH(GETDATE())                      AS FinDeMes,
    EOMONTH(GETDATE(), 1)                   AS FinMesSiguiente,
    DATEPART(quarter, GETDATE())            AS Trimestre,
    FORMAT(GETDATE(), 'dd/MM/yyyy HH:mm')  AS Formateado,
    DATENAME(weekday, GETDATE())            AS DiaSemana;

-- Filtros de fecha comunes
WHERE Fecha >= DATEADD(day, -30, CAST(GETDATE() AS DATE))  -- Últimos 30 días
WHERE YEAR(Fecha) = 2026 AND MONTH(Fecha) = 5              -- Mayo 2026
WHERE Fecha >= DATEFROMPARTS(2026, 1, 1)
  AND Fecha <  DATEFROMPARTS(2027, 1, 1)                   -- Todo 2026

-- Truncar a inicio del mes
SELECT DATEFROMPARTS(YEAR(GETDATE()), MONTH(GETDATE()), 1) AS InicioMes;
```

---

## 12. 📝 Funciones de Texto

| Importancia | Función | Descripción |
|-------------|---------|-------------|
| 🔴 | `LEN(str)` | Longitud (sin espacios finales) |
| 🔴 | `UPPER()` / `LOWER()` | Mayúsculas / minúsculas |
| 🔴 | `TRIM()` / `LTRIM()` / `RTRIM()` | Eliminar espacios |
| 🔴 | `CONCAT(a, b, ...)` | Concatenar cadenas |
| 🔴 | `SUBSTRING(str, pos, len)` | Extraer subcadena |
| 🟠 | `REPLACE(str, old, new)` | Reemplazar texto |
| 🟠 | `CHARINDEX(sub, str)` | Posición de subcadena |
| 🟠 | `LEFT(str, n)` / `RIGHT(str, n)` | Primeros/últimos N caracteres |
| 🟠 | `STRING_AGG(col, sep)` | Concatenar filas en una cadena |
| 🟠 | `STRING_SPLIT(str, sep)` | Dividir cadena en filas |
| 🟡 | `PATINDEX('patrón', str)` | Posición de patrón (con wildcards) |
| 🟡 | `REPLICATE(str, n)` | Repetir cadena N veces |
| 🟡 | `REVERSE(str)` | Invertir cadena |
| 🟡 | `FORMAT(valor, 'formato')` | Formatear números y fechas |
| 🟡 | `STUFF(str, pos, len, new)` | Insertar texto reemplazando |
| 🟡 | `TRANSLATE(str, chars, repl)` | Reemplazar caracteres |

```sql
-- Ejemplos
SELECT
    LEN('  Hola  ')                                 AS Longitud,        -- 8
    TRIM('  Hola  ')                                AS Limpio,          -- 'Hola'
    UPPER('hola mundo')                             AS Mayus,
    CONCAT(Nombre, ' ', Apellido)                   AS NombreCompleto,
    LEFT(Codigo, 3)                                 AS Prefijo,
    SUBSTRING(Nombre, 1, 50)                        AS NombreCorto,
    REPLACE(Telefono, '-', '')                      AS TelSinGuiones,
    CHARINDEX('@', Email)                           AS PosArroba,
    RIGHT(REPLICATE('0', 6) + CAST(ID AS VARCHAR), 6) AS IDPadded,    -- '000042'
    FORMAT(Precio, 'N2', 'es-MX')                  AS PrecioFormato,  -- '1,299.00'
    STRING_AGG(Nombre, ', ') WITHIN GROUP (ORDER BY Nombre) AS Lista;

-- STRING_SPLIT: tabla de filas desde cadena separada por comas
SELECT value AS Tag
FROM STRING_SPLIT('sql,sqlserver,tsql,database', ',');

-- STRING_AGG: agrupar valores en una cadena
SELECT CategoriaID, STRING_AGG(Nombre, ' | ') AS Productos
FROM ventas.Productos
GROUP BY CategoriaID;
```

---

## 13. 🔢 Funciones Numéricas y Condicionales

```sql
-- Numéricas
SELECT
    ABS(-42)                AS Absoluto,       -- 42
    ROUND(3.14159, 2)       AS Redondeado,     -- 3.14
    CEILING(3.2)            AS TechoCeil,      -- 4
    FLOOR(3.9)              AS PisoFloor,      -- 3
    POWER(2, 10)            AS Potencia,       -- 1024
    SQRT(144)               AS RaizCuad,       -- 12
    10 % 3                  AS Modulo,         -- 1
    PI()                    AS Pi,
    RAND()                  AS Aleatorio,      -- entre 0 y 1
    NEWID()                 AS GUID;

-- Condicionales
SELECT
    -- CASE buscado (más flexible)
    CASE
        WHEN Precio < 500   THEN 'Económico'
        WHEN Precio < 2000  THEN 'Medio'
        ELSE 'Premium'
    END AS Rango,

    -- CASE simple
    CASE Estado
        WHEN 1 THEN 'Activo'
        WHEN 0 THEN 'Inactivo'
        ELSE 'Desconocido'
    END AS EstadoTexto,

    -- COALESCE: primer valor no NULL
    COALESCE(Descripcion, NombreCorto, Nombre, 'Sin nombre') AS Texto,

    -- ISNULL: reemplazar NULL con default
    ISNULL(Stock, 0)   AS StockSeguro,

    -- NULLIF: retorna NULL si ambos son iguales
    NULLIF(Descuento, 0) AS DescuentoReal,   -- NULL si descuento es 0

    -- IIF: if ternario
    IIF(Stock > 0, 'Disponible', 'Agotado') AS Disponibilidad,

    -- CHOOSE: seleccionar por índice
    CHOOSE(MONTH(GETDATE()), 'Ene','Feb','Mar','Abr','May','Jun',
                              'Jul','Ago','Sep','Oct','Nov','Dic') AS MesAbrev;
```

---

## 14. 🔒 DCL — Control de Acceso

```sql
-- Crear login (acceso al servidor)
CREATE LOGIN app_usuario WITH PASSWORD = 'C0ntraseña$egura!';

-- Crear usuario en la base de datos
USE Tienda;
CREATE USER app_usuario FOR LOGIN app_usuario;

-- Asignar roles predefinidos
ALTER ROLE db_datareader ADD MEMBER app_usuario;   -- Solo lectura
ALTER ROLE db_datawriter ADD MEMBER app_usuario;   -- Lectura y escritura
ALTER ROLE db_owner      ADD MEMBER app_usuario;   -- Propietario (⚠️ precaución)

-- Permisos granulares
GRANT SELECT ON ventas.Productos    TO app_usuario;
GRANT INSERT ON ventas.Pedidos      TO app_usuario;
GRANT EXECUTE ON ventas.sp_ObtenerProductos TO app_usuario;
GRANT SELECT ON SCHEMA::ventas      TO app_usuario;  -- Todo el esquema

-- Denegar permisos específicos
DENY DELETE ON ventas.Productos TO app_usuario;

-- Revocar permisos
REVOKE INSERT ON ventas.Pedidos FROM app_usuario;

-- Eliminar usuario y login
DROP USER  IF EXISTS app_usuario;
DROP LOGIN IF EXISTS app_usuario;

-- Ver permisos de un usuario
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');
SELECT dp.name, dp.type_desc, p.permission_name, p.state_desc
FROM sys.database_permissions p
JOIN sys.database_principals dp ON p.grantee_principal_id = dp.principal_id
WHERE dp.name = 'app_usuario';
```

---

## 15. 📋 Administración y Monitoreo

```sql
-- ═══════════════════════════════════════
-- INFORMACIÓN DE OBJETOS
-- ═══════════════════════════════════════

-- Listar tablas del esquema
SELECT TABLE_SCHEMA, TABLE_NAME, TABLE_TYPE
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE'
ORDER BY TABLE_SCHEMA, TABLE_NAME;

-- Estructura de una tabla
EXEC sp_help 'ventas.Productos';
EXEC sp_columns 'Productos', 'ventas';

-- Listar columnas y tipos
SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH,
       IS_NULLABLE, COLUMN_DEFAULT
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Productos' AND TABLE_SCHEMA = 'ventas'
ORDER BY ORDINAL_POSITION;

-- Listar procedimientos
SELECT ROUTINE_SCHEMA, ROUTINE_NAME, ROUTINE_TYPE
FROM INFORMATION_SCHEMA.ROUTINES
ORDER BY ROUTINE_TYPE, ROUTINE_NAME;

-- ═══════════════════════════════════════
-- TAMAÑO Y ESPACIO
-- ═══════════════════════════════════════
EXEC sp_spaceused 'ventas.Productos';  -- Espacio de una tabla

-- Tamaño de todas las tablas
SELECT
    s.name + '.' + t.name  AS Tabla,
    p.rows                  AS Filas,
    SUM(a.total_pages) * 8  AS TotalKB,
    SUM(a.used_pages)  * 8  AS UsadoKB
FROM sys.tables t
JOIN sys.schemas       s ON t.schema_id   = s.schema_id
JOIN sys.indexes       i ON t.object_id   = i.object_id
JOIN sys.partitions    p ON i.object_id   = p.object_id AND i.index_id = p.index_id
JOIN sys.allocation_units a ON p.partition_id = a.container_id
GROUP BY s.name, t.name, p.rows
ORDER BY TotalKB DESC;

-- ═══════════════════════════════════════
-- PROCESOS Y BLOQUEOS
-- ═══════════════════════════════════════
EXEC sp_who2;  -- Procesos activos

-- Procesos con query activa
SELECT
    r.session_id,
    r.status,
    r.blocking_session_id,
    r.wait_type,
    r.wait_time / 1000 AS EsperaSegundos,
    SUBSTRING(st.text, (r.statement_start_offset/2)+1, 200) AS Query
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) st
WHERE r.session_id > 50
ORDER BY r.wait_time DESC;

-- Terminar proceso (usar con cuidado)
KILL 72;  -- session_id del proceso a terminar

-- ═══════════════════════════════════════
-- BACKUP Y RESTORE
-- ═══════════════════════════════════════
-- Backup completo
BACKUP DATABASE Tienda
TO DISK = 'C:\Backups\Tienda_Full.bak'
WITH COMPRESSION, STATS = 10;

-- Backup diferencial
BACKUP DATABASE Tienda
TO DISK = 'C:\Backups\Tienda_Diff.bak'
WITH DIFFERENTIAL, COMPRESSION;

-- Backup de log de transacciones
BACKUP LOG Tienda TO DISK = 'C:\Backups\Tienda_Log.trn';

-- Restore
RESTORE DATABASE Tienda
FROM DISK = 'C:\Backups\Tienda_Full.bak'
WITH RECOVERY;

-- ═══════════════════════════════════════
-- ESTADÍSTICAS Y MANTENIMIENTO
-- ═══════════════════════════════════════
UPDATE STATISTICS ventas.Productos;
UPDATE STATISTICS ventas.Productos WITH FULLSCAN;

-- Ver historial de backups
SELECT database_name, backup_start_date, type,
       backup_size / 1024 / 1024 AS SizeMB
FROM msdb.dbo.backupset
WHERE database_name = 'Tienda'
ORDER BY backup_start_date DESC;
```

---

## 16. 🔀 T-SQL Programático

### 16.1 Variables y Control de Flujo

```sql
-- Declarar variables
DECLARE @Nombre     NVARCHAR(100) = 'SQL Server';
DECLARE @Precio     DECIMAL(12,2);
DECLARE @Fecha      DATE          = CAST(GETDATE() AS DATE);
DECLARE @Contador   INT           = 0;

-- Asignar valores
SET @Precio = 1299.99;
SELECT @Precio = MAX(Precio) FROM ventas.Productos;  -- Desde query

-- IF / ELSE
IF @Precio > 1000
BEGIN
    PRINT 'Precio alto: ' + CAST(@Precio AS VARCHAR);
END
ELSE IF @Precio > 500
BEGIN
    PRINT 'Precio medio';
END
ELSE
BEGIN
    PRINT 'Precio bajo';
END;

-- WHILE
WHILE @Contador < 10
BEGIN
    SET @Contador += 1;
    IF @Contador = 5 CONTINUE;  -- Saltar esta iteración
    IF @Contador = 8 BREAK;     -- Salir del bucle
    PRINT 'Contador: ' + CAST(@Contador AS VARCHAR);
END;

-- CASE en asignación
SET @Nombre =
    CASE
        WHEN @Precio > 5000 THEN 'Lujo'
        WHEN @Precio > 1000 THEN 'Premium'
        ELSE 'Estándar'
    END;
```

---

### 16.2 Cursores

```sql
-- ⚠️ Usar solo cuando no haya alternativa basada en conjuntos
DECLARE @ProductoID INT;
DECLARE @Nombre     NVARCHAR(200);

DECLARE cur_Productos CURSOR FAST_FORWARD FOR
    SELECT ProductoID, Nombre FROM ventas.Productos WHERE Activo = 1;

OPEN cur_Productos;
FETCH NEXT FROM cur_Productos INTO @ProductoID, @Nombre;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT CAST(@ProductoID AS VARCHAR) + ' - ' + @Nombre;
    FETCH NEXT FROM cur_Productos INTO @ProductoID, @Nombre;
END;

CLOSE cur_Productos;
DEALLOCATE cur_Productos;
```

---

### 16.3 SQL Dinámico

```sql
-- EXEC con cadena dinámica
DECLARE @Tabla  NVARCHAR(200) = 'ventas.Productos';
DECLARE @SQL    NVARCHAR(MAX);

SET @SQL = N'SELECT TOP 10 * FROM ' + QUOTENAME(@Tabla);
EXEC (@SQL);

-- sp_executesql (más seguro, permite parámetros)
DECLARE @CategoriaID INT = 3;

SET @SQL = N'SELECT Nombre, Precio FROM ventas.Productos
             WHERE CategoriaID = @Cat AND Activo = 1';

EXEC sp_executesql
    @SQL,
    N'@Cat INT',
    @Cat = @CategoriaID;
```

---

## 17. 🔍 Vistas del Sistema (DMVs y Catálogos)

| Importancia | Vista / Función | Descripción |
|-------------|-----------------|-------------|
| 🟠 | `sys.tables` | Tablas del usuario |
| 🟠 | `sys.columns` | Columnas de todas las tablas |
| 🟠 | `sys.indexes` | Índices definidos |
| 🟠 | `sys.procedures` | Procedimientos almacenados |
| 🟠 | `sys.databases` | Bases de datos del servidor |
| 🟡 | `sys.dm_exec_requests` | Consultas en ejecución |
| 🟡 | `sys.dm_exec_query_stats` | Estadísticas de consultas |
| 🟡 | `sys.dm_db_index_physical_stats` | Fragmentación de índices |
| 🟡 | `sys.dm_db_index_usage_stats` | Uso de índices |
| 🟡 | `sys.dm_os_wait_stats` | Estadísticas de espera |
| 🟡 | `INFORMATION_SCHEMA.TABLES` | Tablas estándar ANSI |
| 🟡 | `INFORMATION_SCHEMA.COLUMNS` | Columnas estándar ANSI |
| 🟢 | `sys.dm_os_performance_counters` | Contadores de rendimiento |
| 🟢 | `sys.dm_exec_sessions` | Sesiones activas |

---

## 18. ✅ Buenas Prácticas T-SQL

```sql
-- 1. Usar esquemas para organizar objetos
CREATE TABLE ventas.Pedidos (...);    -- ✅
CREATE TABLE dbo.Pedidos (...);       -- ⚠️ Mezcla todo en dbo

-- 2. Siempre SET NOCOUNT ON en SP y Triggers
CREATE OR ALTER PROCEDURE sp_MiProcedimiento AS
BEGIN
    SET NOCOUNT ON;  -- Evita mensajes "X rows affected"
    ...
END;

-- 3. Usar SCOPE_IDENTITY() en lugar de @@IDENTITY
INSERT INTO tabla (col) VALUES (val);
SELECT SCOPE_IDENTITY();  -- ✅ Seguro en triggers y SP
SELECT @@IDENTITY;        -- ⚠️ Puede capturar ID de otro trigger

-- 4. Usar tipos de datos correctos para fechas
CreadoEn DATETIME2(7) DEFAULT SYSDATETIME()  -- ✅
CreadoEn DATETIME     DEFAULT GETDATE()       -- ⚠️ Menor precisión

-- 5. Evitar SELECT * en producción
SELECT ProductoID, Nombre, Precio FROM ventas.Productos;  -- ✅
SELECT * FROM ventas.Productos;                            -- ⚠️

-- 6. Parámetros en lugar de SQL dinámico concatenado
EXEC sp_executesql N'SELECT * FROM t WHERE id = @id', N'@id INT', @id = 5;  -- ✅
EXEC('SELECT * FROM t WHERE id = ' + @id);                                   -- ❌ SQL Injection

-- 7. Usar transacciones con TRY/CATCH
BEGIN TRANSACTION;
BEGIN TRY
    -- operaciones
    COMMIT TRANSACTION;
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;
    THROW;
END CATCH;

-- 8. Índices: crear INCLUDE para evitar Key Lookups
CREATE INDEX IX_Productos_Cat
ON ventas.Productos(CategoriaID)
INCLUDE (Nombre, Precio, Stock);  -- ✅ Índice de cobertura

-- 9. Usar OR ALTER en lugar de DROP + CREATE
CREATE OR ALTER PROCEDURE sp_Mi AS BEGIN ... END;  -- ✅ SQL Server 2016+

-- 10. Filtros SARGable (que usan índices)
WHERE Fecha >= '2026-01-01'                     -- ✅ SARGable
WHERE YEAR(Fecha) = 2026                        -- ❌ No usa índice
WHERE Precio * 1.16 > 1000                      -- ❌ No usa índice
WHERE Precio > 1000 / 1.16                      -- ✅ SARGable
```

---

## 19. 🗺️ Mapa Mental T-SQL

```
SQL SERVER (T-SQL)
├── DDL (Definición)
│   ├── DATABASE     → CREATE, ALTER, DROP, BACKUP, RESTORE
│   ├── SCHEMA       → CREATE, ALTER, DROP
│   ├── TABLE        → CREATE, ALTER (ADD/MODIFY/DROP), DROP, TRUNCATE
│   ├── INDEX        → CREATE (CLUSTERED/NONCLUSTERED/INCLUDE), DROP, REBUILD
│   └── CONSTRAINTS  → PK, FK, UNIQUE, CHECK, DEFAULT
│
├── DML (Manipulación)
│   ├── SELECT   → TOP, DISTINCT, JOIN, WHERE, GROUP BY, HAVING, ORDER BY
│   │             → PIVOT, UNPIVOT, CTE, Window Functions, Paginación
│   ├── INSERT   → VALUES, SELECT, OUTPUT
│   ├── UPDATE   → SET, FROM+JOIN, OUTPUT
│   ├── DELETE   → WHERE, FROM+JOIN, OUTPUT
│   └── MERGE    → MATCHED / NOT MATCHED / NOT MATCHED BY SOURCE
│
├── DCL (Control)
│   ├── GRANT    → SELECT, INSERT, UPDATE, DELETE, EXECUTE
│   ├── DENY     → Bloqueo explícito
│   └── REVOKE   → Quitar permiso
│
├── TCL (Transacciones)
│   ├── BEGIN TRANSACTION / COMMIT / ROLLBACK
│   ├── SAVE TRANSACTION (Savepoints)
│   └── SET TRANSACTION ISOLATION LEVEL
│
├── Programación T-SQL
│   ├── Variables     → DECLARE, SET, SELECT
│   ├── Flujo         → IF/ELSE, WHILE, BREAK, CONTINUE, RETURN
│   ├── Manejo errores → TRY/CATCH, THROW, ERROR_MESSAGE()
│   ├── Cursores      → DECLARE, OPEN, FETCH, CLOSE, DEALLOCATE
│   └── SQL Dinámico  → EXEC, sp_executesql
│
└── Objetos
    ├── Vistas         → CREATE/ALTER/DROP VIEW, Vistas Indexadas
    ├── Procedimientos → CREATE OR ALTER PROCEDURE, EXEC
    ├── Funciones      → Escalares, ITVF, MSTVF
    └── Triggers       → AFTER, INSTEAD OF, INSERTED/DELETED
```

---

*📌 Referencia para SQL Server 2016+ / SQL Server 2019+ / Azure SQL Database · T-SQL · Actualización: 2026*
