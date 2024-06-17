CREATE DATABASE clean;
USE clean;


-- metodo procedure limp, para seleccionar todo de la tabla limpieza --- 

CREATE PROCEDURE limp
AS
BEGIN
    SELECT * FROM limpieza;
END;
GO
-- ejecutamos metodo limp

EXEC limp;

--- cambiar nombre de la columna 

EXEC sp_rename 
'limpieza.Id?empleado',
'Id_empl',
'Column'

-- seleccion de valores duplicados, traidos en un nueva columna -- 

SELECT Id_empl, COUNT(*) AS TotalDuplicados
FROM limpieza
GROUP BY Id_empl
HAVING COUNT(*) > 1;

-- subconsulta para sabar la cantidad de duplicados existentes --

SELECT COUNT(*) AS cantidad_duplicados
FROM (
    SELECT Id_empl
    FROM limpieza
    GROUP BY Id_empl
    HAVING COUNT(*) > 1
) AS subquery;

-- crear tabla temporal --

EXEC sp_rename 'limpieza', 'conduplicados';

-- Crear una tabla temporal de usuario
-- Crear una tabla temporal de usuario y copiar datos sin valores repetidos

SELECT DISTINCT *
INTO temporal
FROM conduplicados;

SELECT * FROM temporal;

SELECT Id_empl, COUNT(*) AS TotalDuplicados
FROM #temporal
GROUP BY Id_empl
HAVING COUNT(*) > 1;

-- create tabla y pasamos de temporal a estatica

SELECT *
INTO limpieza
FROM #temporal;

EXEC limp;
--- revisamos si hay duplicados en limpieza -- 

SELECT Id_empl, COUNT(*) AS TotalDuplicados
FROM limpieza
GROUP BY Id_empl
HAVING COUNT(*) > 1;

---- borrar tabla y dejamos tabla depurada ---

DROP TABLE conduplicados;

EXEC limp;

--- renombrar columnas -- 
EXEC sp_rename 'limpieza.limpieza.last_name', 'apellido', 'COLUMN';

EXEC sp_rename 'limpieza.last_name', 'apellido', 'COLUMN';


EXEC sp_rename 'limpieza.género', 'gender', 'COLUMN';

-- revisar tipos de datos por columna --
SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'limpieza' AND COLUMN_NAME = 'Id_empl';


-- revisar tipos de datos por tabla --

SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'limpieza';

-- identificar espacios en blanco por columnan, en este caso columna name --

SELECT name
FROM limpieza 
WHERE DATALENGTH(name) - DATALENGTH(RTRIM(LTRIM(name))) > 0;

--- quitamos espacios en blanco

UPDATE limpieza
SET name = LTRIM(RTRIM(name));

-- Adicionar espacios extra a propósito
UPDATE limpieza SET area = REPLACE(area, ' ', '       '); 

EXEC limp;

-- # Explorar si hay dos o más espacios entre dos palabras  
SELECT area
FROM limpieza 
WHERE PATINDEX('%  %', area) > 0;

--Consultar los espacios extra 
SELECT area, LTRIM(RTRIM(REPLACE(REPLACE(REPLACE(area, CHAR(9), ''), CHAR(13), ''), CHAR(10), ''))) AS ensayo
FROM limpieza;

UPDATE limpieza
SET area = LTRIM(RTRIM(REPLACE(REPLACE(REPLACE(area, CHAR(9), ''), CHAR(13), ''), CHAR(10), ' ')));

--Elimina los espacios en blanco entre palabras---
UPDATE limpieza
SET area = REPLACE(area, ' ', '');


EXEC limp;

 -- ===========  Buscar y reemplazar (textos) ================== --
-- ------ Ajustar gender -----

SELECT gender,
    CASE
        WHEN gender = 'hombre' THEN 'Male'
        WHEN gender = 'mujer' THEN 'Female'
        ELSE 'Other'
    END AS gender1
FROM limpieza;

--- actualizar tabla
UPDATE Limpieza
SET Gender = 
    CASE
        WHEN gender = 'hombre' THEN 'Male'
        WHEN gender = 'mujer' THEN 'Female'
        ELSE 'Other'
    END;

	EXEC limp;


--  para obtener información sobre las columnas de una tabla
EXEC sp_columns 'limpieza';

--Para modificar el tipo de datos de una columna en SQL Server
ALTER TABLE limpieza
ALTER COLUMN Type nvarchar(50);

   EXEC sp_columns 'limpieza';

   -- quitar la coma precednte a los valores ---
UPDATE limpieza
SET type = REPLACE(type, ',', '');

--- seleccion con cambio pasar de 0 a 1 a remote e hibrid ----

SELECT type,
CASE 
	WHEN type = 1 THEN 'Remote'
    WHEN type = 0 THEN 'Hybrid'
    ELSE 'Other'
END as ejemplo
FROM limpieza;

exec limp;

--- modificar columna type y  pasar de 0 a 1 a remote e hibrid -----
--- utilizando comillas simples alrededor de los valores numéricos (1 y 0) para asegurarme de que se traten como cadenas de caracteres. --
--- Esto debería resolver el problema de conversión y permitir que la actualización se ejecute correctamente.---

UPDATE limpieza
SET type = 
    CASE 
		WHEN type = '1' THEN 'Remote'
		WHEN type = '0' THEN 'Hybrid'
		ELSE NULL -- o algún otro valor numérico si es necesario
    END;


-- ----- consultar :reemplazar $ por un vacío y cambiar el separador de mil por vacío---

/* Explicación 
-- REPLACE(salary, '$', ''):Se utiliza para eliminar el símbolo de dólar ('$') de la columna 'salary',
		y reemplaza cada aparición del símbolo $ con una cadena vacía ('').
-- REPLACE(..., ',', ''): Se utiliza nuevamente para eliminar las comas (',') de la columna 'salary'. 
		y reemplaza cada aparición de la coma con una cadena vacía ('').

-- DECIMAL(15, 2)):  se utiliza para convertir el valor resultante en un número decimal con una precisión de 15 dígitos en total y 2 decimales. 
---- La función TRY_CONVERT() intentará convertir el valor de salary a un tipo de datos decimal. Si la conversión es exitosa, devolverá el valor convertido;---
---de lo contrario, devolverá NULL. Esto te ayudará a identificar los valores que están causando problemas de conversión-----*/



SELECT salary, 
    TRY_CONVERT(DECIMAL(15, 2), REPLACE(REPLACE(salary, '$', ''), '"', '')) AS salario_decimal
FROM limpieza;

---actulizamos la columna salary ----

UPDATE limpieza
SET salary = TRY_CONVERT(DECIMAL(15, 2), REPLACE(REPLACE(salary, '$', ''), '"', ''))


-- dar formato a la fecha ----
*/---En SQL Server, hay varios estilos de formato de fecha que puedes utilizar en la función CONVERT para convertir una fecha a una cadena de caracteres con un formato específico.
--Aquí tienes algunos 
--ejemplos de formatos comunes:
--101: Formato MM/DD/AAAA.
---102: Formato AAAA.MM.DD.
--103: Formato DD/MM/AAAA.
--104: Formato DD.MM.AAAA.
--105: Formato DD-MM-AAAA.
--106: Formato DD MMM AAAA.
--107: Formato MMM DD, AAAA.
--108: Formato HH:MM:SS.
--109: Formato MMM DD AAAA HH:MM:SS:MMMAM.
--110: Formato AAAA-MM-DD.
--111: Formato AAAA/MM/DD.
--112: Formato AAAAMMDD.-----*/


SELECT birth_date,
       CASE 
           WHEN CHARINDEX('/', birth_date) > 0 THEN CONVERT(VARCHAR, TRY_CONVERT(DATE, birth_date, 101), 23)
           WHEN CHARINDEX('-', birth_date) > 0 THEN CONVERT(VARCHAR, TRY_CONVERT(DATE, birth_date, 110), 23)
           ELSE NULL
       END AS new_birth_date
FROM limpieza;

---actualizamos columna birth_date---
UPDATE limpieza
SET birth_date = 
    CASE
		  WHEN CHARINDEX('/', birth_date) > 0 THEN CONVERT(VARCHAR, TRY_CONVERT(DATE, birth_date, 101), 23)
           WHEN CHARINDEX('-', birth_date) > 0 THEN CONVERT(VARCHAR, TRY_CONVERT(DATE, birth_date, 110), 23)
           ELSE NULL
       END;


-- Cambiar el tipo de dato de la columna 
ALTER TABLE limpieza
ALTER COLUMN birth_date date;

----consultamos para verificar el cambio de tipo---
exec sp_columns limpieza;

exec limp;

exec sp_rename 'limpieza.finish_date','start_date','COLUMN';

exec sp_rename 'limpieza.promotion_date','finish_date','COLUMN';

--eliminamos column
ALTER TABLE limpieza
DROP COLUMN star_date;



---cambios en tabla finish date
SELECT finish_date, CONVERT(DATETIME, REPLACE(finish_date, ' UTC', ''), 120) AS fecha
FROM limpieza;

-- seleccion y formateo columna finish_date----
SELECT finish_date,
       CASE 
           WHEN CHARINDEX('/', finish_date) > 0 THEN CONVERT(DATETIME, REPLACE(finish_date, ' UTC', ''), 120)
           WHEN CHARINDEX('-', finish_date) > 0 THEN CONVERT(DATETIME, REPLACE(finish_date, ' UTC', ''), 120)
		   ELSE NULL
       END AS new_finish_date
FROM limpieza;

---actulizamos cambios finish_date---

UPDATE limpieza
SET finish_date =
       CASE 
           WHEN CHARINDEX('/', finish_date) > 0 THEN CONVERT(DATETIME, REPLACE(finish_date, ' UTC', ''), 120)
           WHEN CHARINDEX('-', finish_date) > 0 THEN CONVERT(DATETIME, REPLACE(finish_date, ' UTC', ''), 120)
		   ELSE NULL
       END;

----formater finish_date a tipo date---
ALTER TABLE limpieza
ALTER COLUMN finish_date date; 

exec limp;

---------------------------en caso que se quiera particionar por hora, minuto y segundo ---------
SELECT 
    finish_date,
    DATEPART(HOUR, finish_date) AS hora,
    DATEPART(MINUTE, finish_date) AS minutos,
    DATEPART(SECOND, finish_date) AS segundos,
    CONVERT(VARCHAR(8), finish_date, 108) AS hour_stamp
FROM limpieza;

-----------------Agregar una nueva columna para la copia de seguridad:
ALTER TABLE limpieza
ADD copia_seguridad_finish_date [date];

---copiar los datos 
UPDATE limpieza
SET copia_seguridad_finish_date = finish_date;


---------------------------------
-- ========= Cálculos con fechas ====== -- 

-- # Agregar columna para albergar la edad
ALTER TABLE limpieza ADD age [int];

exec limp;

---- calculamos la edad en base birth_date con star_date para la edad al momento de ingreso---{
SELECT name,
       birth_date,
       start_date,
       DATEDIFF(YEAR, birth_date, start_date) AS edad_de_ingreso
FROM limpieza;




---actualizamos columna birth_date---
UPDATE limpieza
SET birth_date = 
    CASE
		  WHEN CHARINDEX('/', birth_date) > 0 THEN CONVERT(VARCHAR, TRY_CONVERT(DATE, birth_date, 101), 23)
           WHEN CHARINDEX('-', birth_date) > 0 THEN CONVERT(VARCHAR, TRY_CONVERT(DATE, birth_date, 110), 23)
           ELSE NULL
       END;

select * from conduplicados;

UPDATE limpieza
SET birth_date = (
    SELECT birth_date
    FROM conduplicados
);

UPDATE l
SET l.birth_date = cd.birth_date
FROM limpieza l
INNER JOIN conduplicados cd ON l.birth_date = cd.birth_date;
