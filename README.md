# Proyecto-con-Amazon-Athena
# Manual de Uso: Amazon Athena con Datos de FP de Canarias

[cite_start]Este manual documenta el proceso seguido para analizar el alumnado egresado de Formación Profesional en Canarias utilizando el ecosistema de AWS[cite: 8].

## 1. Subida de datos a Amazon S3
[cite_start]Primero, descargamos el dataset del portal del Gobierno de Canarias en formato CSV[cite: 15, 17].
[cite_start]Creamos un bucket en S3 llamado `datos-fp-canarias-grupoX` y subimos nuestro archivo `egresados_fp.csv`[cite: 19]. [cite_start]Este bucket actuará como repositorio de almacenamiento[cite: 20].

<img width="1919" height="678" alt="image" src="https://github.com/user-attachments/assets/f19beebe-bc89-409a-a00e-f9dc08052281" />

## 2. Configuración de la base de datos en Athena
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
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
-- Aquí debes poner la ruta exacta de tu bucket de S3
LOCATION 's3://datos-fp-canarias-grupoX/'
TBLPROPERTIES ('skip.header.line.count'='1'); -- Ignoramos la primera fila si son los títulos
