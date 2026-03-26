# Proyecto-con-Amazon-Athena
# Manual de Uso: Amazon Athena con Datos de egresados de Canarias

Este manual documenta el proceso seguido para investigar la herramienta Athena del ecosistema de análisis de datos de AWS, explicar su funcionamiento y utilizarla para trabajar con un conjunto de datos real.

## 1. Subida de datos a Amazon S3
Primero, descargamos el dataset del portal del Gobierno de Canarias en formato CSV.
Creamos un bucket en S3 llamado `datos-egresados` y subimos nuestro archivo. Este bucket actuará como repositorio de almacenamiento.
Es recomendable crear una carpeta extra donde guardaremos los datos virgenes, ya que esta herramienta guarda los datos en la carpeta origen, por lo que con la carpeta permitiremos que los datos se mantengan intactos y de esta manera no ocasionen problemas en las consultas.

<img width="1913" height="475" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/1b93c14d-7a3f-4649-a27f-6d6d230d4122" />

## 2. Configuración de la base de datos en Athena
Antes de nada le debemos indicar donde queremos que se guarden los resultados, por lo que entramos en Amazon Athena.
Abrimos esta pestaña y le damos a `administrar`:
<img width="1913" height="607" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/f413a460-86a7-4db7-9528-5ea55908c5a0" />
### Y le indicamos la ruta exacta de nuestro S3
<img width="1549" height="699" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/3bd947cf-48bd-4585-8bff-3c34d1471185" />

### Creamos la BBDD de la manera más sencilla 
<img width="1915" height="752" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/480e6bfb-faf3-487f-9ecc-129f2e9ddbc4" />

### Le damos a crear tabla a partir de los datos del S3.
<img width="1919" height="462" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/649b77d1-bac7-4552-9ff4-1ca63a4c45ba" />

### Le indicamos el nombre de la tabla que queremos crear y le decimos que la cree en la BBDD que ya habiamos creado antes.
<img width="1918" height="560" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/ec737e39-8a3e-4180-8b3a-fec92f3fdfcd" />

### Ponemos donde se encuentran los datos en nuestro S3
<img width="1472" height="196" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/601e0edf-2bf8-4970-a8c4-3d58ccedff34" />

### Cambiamos el formato del archivo para indicarle que es un .tsv
<img width="1637" height="408" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/1203e79d-1286-457e-9b70-5de43003db76" />

### Agregamos las columnas y el tipo
<img width="846" height="480" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/3d353b3c-25db-471c-a578-dd6936357ce5" />

### Revisamos por si fallamos en algo, si esta todo bien continuamos
<img width="1919" height="856" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/54d7e5a1-98c5-4caa-9ca2-6b0f59a9e547" />

### Si lo hemos hecho bien, nos aparecerá el código de nuestra tabla y ya le damos a crear la tabla.
<img width="1919" height="834" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/ac22527b-6d8d-4fc1-807b-4bb20fa32b9e" />

```
CREATE EXTERNAL TABLE IF NOT EXISTS `canarias_estadisticas`.`ISTAC_egresados` (
  `time_period` string,
  `sexo` string,
  `titulacion` string,
  `modalidad_formacion` string,
  `lugar_residencia` string,
  `tiempo_egreso` string,
  `relacion_actividad` string,
  `medidas` string,
  `attribute` string,
  `attribute_value` string,
  `attribute_value_code` string,
  `territorio` string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES ('field.delim' = '\t')
STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat' OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION 's3://datos-egresados/datos-origen/dataset-ISTAC-C00051A_000047-1_1/'
TBLPROPERTIES ('skip.header.line.count'='1'); #esta linea la añadimos para que no se repitan los titulos de las columns
```

### Ejecutamos el script
<img width="1917" height="801" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/f7e69505-da57-47c5-bbd7-f7265eb505f7" />

### Hacemos un `select * ` para comprobar que efetivamente los datos están
<img width="1914" height="843" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/96fdddaf-cf4a-47b3-b5a9-efd6101e80da" />

## 3. Análisis de datos con la herramienta Athena
### ¿A qué se dedican los graduados?

Esta consulta te permite saber cuántos de esos egresados han seguido estudiando, cuántos están trabajando o en otras situaciones
<img width="1913" height="847" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/ec9bab46-be8a-49c7-bf8f-d8f500afd91d" />

```
SELECT 
  relacion_actividad, 
  ROUND(SUM(TRY_CAST(attribute AS DOUBLE))) AS total_personas
FROM istac_egresados
WHERE medidas LIKE '%Egresados%' 
  AND relacion_actividad NOT LIKE '%Total%' 
  AND sexo NOT LIKE '%Total%'
  AND titulacion NOT LIKE '%Total%'
GROUP BY relacion_actividad
ORDER BY total_personas DESC;
```

### ¿En qué momento se recogen los datos?

Esta consulta nos permite ver cuántos datos hay de gente recién graduada vs gente que se graduó hace 5 años
<img width="1919" height="875" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/0f5255c6-1998-4c8b-8eb5-897a9cb3082c" />
```
SELECT 
  tiempo_egreso, 
  ROUND(SUM(TRY_CAST(attribute AS DOUBLE))) AS volumen_datos
FROM istac_egresados
WHERE medidas LIKE '%Egresados%' 
  AND tiempo_egreso NOT LIKE '%Total%'
  AND sexo NOT LIKE '%Total%'
GROUP BY tiempo_egreso
ORDER BY volumen_datos DESC;
```
### Comparativa por Sexo

En esta consulta separamos y analizamos el número de graduados por género para cada familia profesional, eliminando los datos duplicados y los porcentajes para obtener una comparativa real de hombres y mujeres.
<img width="1913" height="847" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/1e51f742-1a6e-4c5f-9c37-5663fc0b69ec" />
```
SELECT 
  titulacion AS familia_profesional,
  ROUND(SUM(CASE WHEN sexo LIKE '%Hombres%' THEN TRY_CAST(attribute AS DOUBLE) ELSE 0 END)) AS total_hombres,
  ROUND(SUM(CASE WHEN sexo LIKE '%Mujeres%' THEN TRY_CAST(attribute AS DOUBLE) ELSE 0 END)) AS total_mujeres
FROM istac_egresados
WHERE medidas LIKE '%Egresados%' 
  AND titulacion NOT LIKE '%Total%' 
  AND sexo NOT LIKE '%Total%'
  AND sexo NOT LIKE '%Ambos%'
GROUP BY titulacion
ORDER BY (total_hombres + total_mujeres) DESC;
```
### Revisión de los resultados
<img width="1918" height="843" alt="image" src="https://raw.githubusercontent.com/user-attachments/assets/c277af15-d3c8-4edd-afd7-46f090be82fc" />

### Comprobamos que las modalidades son Presencial y No presencial, aunque aparezca total
"El archivo original contiene filas agregadas (Totales) para facilitar la lectura manual, pero para el análisis en Athena las he filtrado para evitar duplicidades y asegurar la integridad de los cálculos".