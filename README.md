# Proyecto-con-Amazon-Athena
# Manual de Uso: Amazon Athena con Datos de FP de Canarias

[cite_start]Este manual documenta el proceso seguido para analizar el alumnado egresado de Formación Profesional en Canarias utilizando el ecosistema de AWS[cite: 8].

## 1. Subida de datos a Amazon S3
[cite_start]Primero, descargamos el dataset del portal del Gobierno de Canarias en formato CSV[cite: 15, 17].
[cite_start]Creamos un bucket en S3 llamado `datos-fp-canarias-grupoX` y subimos nuestro archivo `egresados_fp.csv`[cite: 19]. [cite_start]Este bucket actuará como repositorio de almacenamiento[cite: 20].

<img width="1919" height="678" alt="image" src="https://github.com/user-attachments/assets/f19beebe-bc89-409a-a00e-f9dc08052281" />

## 2. Configuración de la base de datos en Athena
Antes de nada le debemos indicar donde queremos que se guarden los resultados.
<img width="1549" height="699" alt="image" src="https://github.com/user-attachments/assets/3bd947cf-48bd-4585-8bff-3c34d1471185" />

Nos dirigimos a Amazon Athena. [cite_start]Para que Athena pueda leer el archivo CSV que está en S3, debemos crear una tabla externa[cite: 35].
**Código utilizado:**
```sql
-- Creamos la base de datos si no existe
CREATE DATABASE IF NOT EXISTS canarias_estadisticas;

-- Usamos la base de datos
USE canarias_estadisticas;

-- Creamos la tabla definiendo las columnas que tiene nuestro CSV
-- NOTA: Adapta estos nombres a los que tenga tu CSV real
CREATE EXTERNAL TABLE IF NOT EXISTS egresados_fp (
  curso STRING,
  familia_profesional STRING,
  sexo STRING,
  total_egresados INT
)
-- Le decimos que es un CSV y cómo están separados los datos
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
-- Aquí debes poner la ruta exacta de tu bucket de S3
LOCATION 's3://datos-egresados/'
TBLPROPERTIES ('skip.header.line.count'='1'); -- Ignoramos la primera fila 
```
*Creamos la BBDD*
<img width="1915" height="752" alt="image" src="https://github.com/user-attachments/assets/480e6bfb-faf3-487f-9ecc-129f2e9ddbc4" />

*Creamos la tabla*
<img width="1918" height="821" alt="image" src="https://github.com/user-attachments/assets/fe88aa0b-66e3-423d-99c0-c558da3ee9ec" />

*Mostramos algunos datos par verificar que los lee bien*
<img width="1919" height="856" alt="image" src="https://github.com/user-attachments/assets/39d8ed9b-a2ee-4e3e-b81e-a74add0999d7" />


