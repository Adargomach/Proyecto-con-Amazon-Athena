# Proyecto-con-Amazon-Athena
# Manual de Uso: Amazon Athena con Datos de egresados de Canarias

Este manual documenta el proceso seguido para investigar la herramienta Athena del ecosistema de análisis de datos de AWS, explicar su funcionamiento y utilizarla para trabajar con un conjunto de datos real.

## 1. Subida de datos a Amazon S3
Primero, descargamos el dataset del portal del Gobierno de Canarias en formato CSV.
Creamos un bucket en S3 llamado `datos-egresados` y subimos nuestro archivo. Este bucket actuará como repositorio de almacenamiento.
Es recomendable crear una carpeta extra donde guardaremos los datos virgenes, ya que esta herramienta guarda los datos en la carpeta origen, por lo que con la carpeta permitiremos que los datos se mantengan intactos y de esta manera no ocasionen problemas en las consultas.

<img width="1913" height="475" alt="image" src="https://github.com/user-attachments/assets/1b93c14d-7a3f-4649-a27f-6d6d230d4122" />

## 2. Configuración de la base de datos en Athena
Antes de nada le debemos indicar donde queremos que se guarden los resultados, por lo que entramos en Amazon Athena.
Abrimos esta pestaña y le damos a `administrar`:
<img width="1913" height="607" alt="image" src="https://github.com/user-attachments/assets/f413a460-86a7-4db7-9528-5ea55908c5a0" />
Y le indicamos la ruta exacta de nuestro S3
<img width="1549" height="699" alt="image" src="https://github.com/user-attachments/assets/3bd947cf-48bd-4585-8bff-3c34d1471185" />

Nos dirigimos a Amazon Athena. Para que Athena pueda leer el archivo CSV que está en S3, debemos crear una tabla externa.
**Código utilizado:**
```sql
-- Creamos la base de datos
CREATE DATABASE IF NOT EXISTS canarias_estadisticas;
-- Creamos la tabla 
CREATE EXTERNAL TABLE egresados_fp (
  curso STRING,
  sexo STRING,
  titulacion STRING,
  modalidad_formacion STRING,
  lugar_residencia STRING,
  tiempo_egreso STRING,
  relacion_actividad STRING,
  medidas STRING,
  total STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t' 
STORED AS TEXTFILE
LOCATION 's3://datos-egresados/datos-origen/dataset-ISTAC-C00051A_000047-1_1/';
TBLPROPERTIES ('skip.header.line.count'='1'); 
```
### Creamos la BBDD
<img width="1915" height="752" alt="image" src="https://github.com/user-attachments/assets/480e6bfb-faf3-487f-9ecc-129f2e9ddbc4" />

### Creamos la tabla
<img width="1909" height="769" alt="image" src="https://github.com/user-attachments/assets/cd1bce42-61dd-4b4d-b22b-fa0591d6671b" />


### Mostramos algunos datos par verificar que los lee bien
<img width="1919" height="868" alt="image" src="https://github.com/user-attachments/assets/58b2bd69-0690-4dc3-aea8-27b554214fbe" />

## 3. Análisis de datos con la herramienta Athena
### Familias Profesionales con más graduados

Esta consulta nos muestra qué titulaciones tienen un mayor volumen de egresados.
<img width="1919" height="850" alt="image" src="https://github.com/user-attachments/assets/f4820951-1722-45a8-b22c-acd02f5fd6cc" />

### Evolución Temporal por Curso

Analizamos si el número de graduados en FP en Canarias ha crecido o disminuido a lo largo de los años.
<img width="1919" height="837" alt="image" src="https://github.com/user-attachments/assets/834632b7-3f58-4277-9e24-153f91d64d8b" />

### Comparativa por Sexo

En esta consulta separamos y analizamos el número de graduados por género para cada familia profesional, eliminando los datos duplicados y los porcentajes para obtener una comparativa real de hombres y mujeres.
<img width="1901" height="844" alt="image" src="https://github.com/user-attachments/assets/1d69b100-37a5-418d-8818-bdb4bc9fb5b6" />




