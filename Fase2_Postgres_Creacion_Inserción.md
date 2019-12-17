# Postgres
# Fase 2: Creación de la Base de Datos. Carga de datos

* Script de creación de tablas y restrincciones [AQUÍ]()
* Script de inserción de datos [AQUÍ]()

## Creación de Tablas y Restrincciones:

#### Los tipos de datos y el tamaño de las columnas deben ser asignados correctamente por el alumno.

![Fase2](image/Fase2.png)

-------------------------------

#### Restrincciones Generales:

* Todas las claves primarias, ajenas y candidatas de todas las tablas.
* Las columnas que no puedan dejarse en blanco tendrán la correspondiente restricción
de obligatoriedad.

-------------------------------
### CONFLICTOS

##### Restrincciones:

* La causa de un conflicto será una de las siguientes: Racial, Económico, Religioso o
Desconocida.

``` sql

CREATE TABLE conflictos (
    codigo                  VARCHAR(3),
    nombre                  VARCHAR(50),
    causa                   VARCHAR(15),
    CONSTRAINT pk_codigo_conflictos PRIMARY KEY(codigo),
    CONSTRAINT causa_seleccion CHECK(upper(causa) in ('RACIAL','ECONOMICO','RELIGIOSO','DESCONOCIDA')),
    CONSTRAINT notnull_nombreconfl CHECK(nombre IS NOT NULL),
    CONSTRAINT unique_nombreconfl UNIQUE(nombre)
);
```
-------------------------------
### PAISES

##### Restrincciones:
* El código de país incluye una letra que indica el continente y dos dígitos para indicar
el número de orden alfabético del país dentro de dicho continente.

``` sql
CREATE TABLE paises (
    codigo                  VARCHAR(5),
    nombre                  VARCHAR(30),
    capital                 VARCHAR(30),
    CONSTRAINT pk_codigo_paises PRIMARY KEY(codigo),
    CONSTRAINT cod_letrapais_dosdig CHECK(codigo ~ '^[A-Z]{2}[0-9]{2}$' AND SUBSTR(codigo,1,2) in ('EU','AS','AF','NA','SA','OC','AN')),
    CONSTRAINT notnull_nombrepais CHECK(nombre IS NOT NULL),
    CONSTRAINT unique_nombrepais UNIQUE(nombre)
);
```
-------------------------------
### Historial de Conflictos

##### Restrincciones:
* Cuando se introducen los datos de un nuevo conflicto el número de heridos y muertos
será 0 a no ser que se especifique lo contrario.

``` sql
CREATE TABLE historial_de_conflictos (
    codigo_conflicto        VARCHAR(3),
    codigo_pais             VARCHAR(5),
    num_heridos             NUMERIC(6) DEFAULT 0,
    num_muertos             NUMERIC(6) DEFAULT 0,
    CONSTRAINT pk_codconflicto_codpais_historial PRIMARY KEY(codigo_conflicto,codigo_pais),
    CONSTRAINT fk_cod_conflicto FOREIGN KEY(codigo_conflicto) REFERENCES conflictos(codigo),
    CONSTRAINT fk_cod_pais FOREIGN KEY(codigo_pais) REFERENCES paises(codigo)
);
```
-------------------------------
### Grupos Armados

##### Restrincciones:
* El código de un grupo armado está compuesto de una letra que será A, B o C, un
guión y entre uno y tres dígitos.

``` sql
CREATE TABLE grupos_armados (
    codigo                  VARCHAR(5),
    nombre                  VARCHAR(50),
    CONSTRAINT pk_codigo_gruparmados PRIMARY KEY(codigo),
    CONSTRAINT codigo_ABC CHECK(codigo ~ '^[ABC]-[0-9]{1,3}$'),
    CONSTRAINT notnull_nombregrup CHECK(nombre IS NOT NULL),
    CONSTRAINT unique_nombregrup UNIQUE(nombre)
);
```
-------------------------------
### Historial de Intervenciones Armadas

##### Restrincciones:
* Las fechas de incorporación de un grupo armado a un conflicto son siempre
posteriores a Abril de 2001.

``` sql
CREATE TABLE historial_intervenciones_armadas (
    codigo_gruparmado       VARCHAR(5),
    codigo_conflicto        VARCHAR(3),
    fecha_incorporacion     DATE,
    fecha_retirada          DATE,
    CONSTRAINT pk_codgruparmado_codconflicto_fecha PRIMARY KEY(codigo_gruparmado,codigo_conflicto,fecha_incorporacion),
    CONSTRAINT fk_cod_gruparmado_armadas FOREIGN KEY(codigo_gruparmado) REFERENCES grupos_armados(codigo),
    CONSTRAINT fk_cod_conflicto_armadas FOREIGN KEY(codigo_conflicto) REFERENCES conflictos(codigo),
    CONSTRAINT mas_abril2001 CHECK(to_char(fecha_incorporacion, 'YYYY/MM') > '2001/04')
);
```
-------------------------------
### Organizaciones

##### Restrincciones:
* Los nombres de las organizaciones se almacenarán con la inicial de cada palabra en
mayúsculas.

``` sql
CREATE TABLE organizaciones (
    codigo                  VARCHAR(3),
    codigo_orgdepen         VARCHAR(3),
    nombre                  VARCHAR(30),
    numpersonas_conflicto   NUMERIC(3),
    tipo                    VARCHAR(30),
    CONSTRAINT pk_codigo_organizaciones PRIMARY KEY(codigo),
    CONSTRAINT fk_codigo_orgdepend FOREIGN KEY(codigo_orgdepen) REFERENCES organizaciones(codigo),
    CONSTRAINT iniciales_mayus CHECK(nombre = INITCAP(nombre)),
    CONSTRAINT notnull_nombreorg CHECK(nombre IS NOT NULL),
    CONSTRAINT notnull_tipoorg CHECK(tipo IS NOT NULL),
    CONSTRAINT unique_nombreorg UNIQUE(nombre)
);
```
-------------------------------
### Historial de Intervenciones Mediadoras

``` sql
CREATE TABLE historial_intervenciones_mediadoras (
    codigo_org              VARCHAR(3),
    codigo_conflicto        VARCHAR(3),
    fecha_incorporacion     DATE,
    fecha_retirada          DATE,
    tipo_ayuda              VARCHAR(30),
    CONSTRAINT pk_orga_conflitos_fecha_historial PRIMARY KEY(codigo_org,codigo_conflicto,fecha_incorporacion),
    CONSTRAINT fk_cod_orga FOREIGN KEY(codigo_org) REFERENCES organizaciones(codigo),
    CONSTRAINT fk_cod_conflictos FOREIGN KEY(codigo_conflicto) REFERENCES conflictos(codigo)
);
```
-------------------------------
### Envios
##### Restrincciones:
* Los envíos se realizan siempre entre las ocho de la mañana y las cuatro de la tarde.
* Los envíos se realizan siempre el día 1 ó 15 de cada mes.

``` sql
CREATE TABLE envios (
    codigo                  VARCHAR(3),
    codigo_org              VARCHAR(3),
    fecha_hora              TIMESTAMP,
    CONSTRAINT pk_codigo_envios PRIMARY KEY(codigo),
    CONSTRAINT fk_cod_org FOREIGN KEY(codigo_org) REFERENCES organizaciones(codigo),
    CONSTRAINT entre8_16 CHECK(to_char(fecha_hora, 'HH24:MI') between '08:00' and '16:00'),
    CONSTRAINT dia1_15 CHECK(to_char(fecha_hora, 'dd') = '01' OR to_char(fecha_hora, 'dd') = '15')
);
```
-------------------------------
### Productos

``` sql
CREATE TABLE productos (
    nombre                  VARCHAR(20),
    descripcion             VARCHAR(100),
    CONSTRAINT pk_nombre_productos PRIMARY KEY(nombre)
);
```
-------------------------------
### Campos de Refugiados

``` sql
CREATE TABLE campos_refugiados (
    codigo                  VARCHAR(3),
    nombre                  VARCHAR(30),
    zona_asentamiento       VARCHAR(50),
    CONSTRAINT  pk_codigo_camprefugiados PRIMARY KEY(codigo),
    CONSTRAINT notnull_nombrecamp CHECK(nombre IS NOT NULL),
    CONSTRAINT notnull_zonacamp CHECK(zona_asentamiento IS NOT NULL),
    CONSTRAINT unique_nombrcamp UNIQUE(nombre)
);
```
-------------------------------
### Paquetes


``` sql
CREATE TABLE paquetes  (
    codigo                  VARCHAR(3),
    codigo_refugiados       VARCHAR(3),
    codigo_envio            VARCHAR(3),
    nombre_producto         VARCHAR(30),
    cantidad                NUMERIC(4),
    CONSTRAINT pk_codigo_paquetes PRIMARY KEY(codigo),
    CONSTRAINT fk_cod_refugiados FOREIGN KEY(codigo_refugiados) REFERENCES campos_refugiados(codigo),
    CONSTRAINT fk_cod_envio FOREIGN KEY(codigo_envio) REFERENCES envios(codigo),
    CONSTRAINT fk_nombre_productos FOREIGN KEY(nombre_producto) REFERENCES productos(nombre),
    CONSTRAINT notnull_cantidadpaquete CHECK(cantidad IS NOT NULL)
);
```
-------------------------------
### Refugiados

``` sql
CREATE TABLE refugiados (
    codigo                  VARCHAR(4),
    codigo_refugio          VARCHAR(3),
    nombre                  VARCHAR(40),
    edad                    NUMERIC(3),
    fecha_llegada           DATE,
    fecha_salida            DATE,
    CONSTRAINT pk_refugiado_refugio PRIMARY KEY(codigo,codigo_refugio),
    CONSTRAINT notnull_nombrerefug CHECK(nombre IS NOT NULL),
    CONSTRAINT notnull_edadrefug CHECK(edad IS NOT NULL)
);
```
-------------------------------

## Inserción de Datos

* La carga de datos debe realizarse con datos consistentes y cumpliendo todas las
restricciones. La cantidad mínima de datos en cada una de las tablas viene detallada a
continuación:

|       Nombre Tabla                       |  Nº Mínimo Registros  |
|:----------------------------------------:|:---------------------:|
|Países                                    |     8                 |
|Conflictos                                |     9                 | 
|Grupos Armados                            |     5                 |
|Historial de Conflictos                   |     10                |
|Historial de intervenciones armadas       |     10                |
|Organizaciones mediadoras                 |     6                 |
|Historial de intervenciones mediadoras    |     10                |
|Envíos                                    |     12                |
|Paquetes                                  |     20                |
|Productos                                 |     10                |
|Campos de refugiados                      |     6                 |


### CONFLICTOS

##### Inserción de datos:

``` sql
INSERT INTO conflictos VALUES ('1','La guerra de Afganistan','Racial');
INSERT INTO conflictos VALUES ('2','Campaña Nacional','Economico');
INSERT INTO conflictos VALUES ('3','La guerra civil de Sudan del Sur','Religioso');
INSERT INTO conflictos VALUES ('4','La guerra civil de Myanmar','Religioso');
INSERT INTO conflictos VALUES ('5','La guerra de Biafra','Economico');
INSERT INTO conflictos VALUES ('6','Segunda guerra del Golfo','Religioso');
INSERT INTO conflictos VALUES ('7','Guerra civil de Siria','Racial');
INSERT INTO conflictos VALUES ('8','La guerra civil de Yemen','Desconocida');
INSERT INTO conflictos VALUES ('9','La guerra civil somali','Economico');
INSERT INTO conflictos VALUES ('10','Conflicto del Delta del Niger','Economico');
```

##### Comprobación de las restrincciones:

``` sql
INSERT INTO conflictos VALUES ('1','La guerra de Afganistan','Racial');
--INSERT 0 1
INSERT INTO conflictos VALUES ('1','La guerra de Afganistan','Racia');
--ERROR:  el nuevo registro para la relación «conflictos» viola la restricción «check» «causa_seleccion»
--DETALLE:  La fila que falla contiene (1, La guerra de Afganistan, Racia).
INSERT INTO conflictos VALUES ('1','La guerra de Afganistan','Racial');
--ERROR:  llave duplicada viola restricción de unicidad «pk_codigo_conflictos»
--DETALLE:  Ya existe la llave (codigo)=(1).
```
-------------------------------
### PAISES

##### Inserción de datos:

``` sql
INSERT INTO paises VALUES ('AS01','Afganistan','Kabul');
INSERT INTO paises VALUES ('AS43','Turquia','Ankara');
INSERT INTO paises VALUES ('AF49','Sudan del Sur','Yuba');
INSERT INTO paises VALUES ('AS32','Myanmar','Naipyidó');
INSERT INTO paises VALUES ('AF38','Nigeria','Abuya');
INSERT INTO paises VALUES ('AS19','Irak','Bagdad');
INSERT INTO paises VALUES ('AS38','Siria','Damasco');
INSERT INTO paises VALUES ('AS46','Yemen','Sana');
INSERT INTO paises VALUES ('AF46','Somalia','Mogadiscio');
```

##### Comprobación de las restrincciones:

``` sql
INSERT INTO paises VALUES ('AS93','Afganistan','Kabul');
--INSERT 0 1
INSERT INTO paises VALUES ('A93','Afganistan','Kabul');
--ERROR:  el nuevo registro para la relación «paises» viola la restricción «check» «cod_letrapais_dosdig»
--DETALLE:  La fila que falla contiene (A093, Afganistan, Kabul).
INSERT INTO paises VALUES ('93','Afganistan','Kabul');
--ERROR:  el nuevo registro para la relación «paises» viola la restricción «check» «cod_letrapais_dosdig»
--DETALLE:  La fila que falla contiene (093, Afganistan, Kabul).
INSERT INTO paises VALUES ('AT93','Afganistan','Kabul');
--ERROR:  el nuevo registro para la relación «paises» viola la restricción «check» «cod_letrapais_dosdig»
--DETALLE:  La fila que falla contiene (AS93, Afganistan, Kabul).
INSERT INTO paises VALUES ('as93','Afganistan','Kabul');
--ERROR:  el nuevo registro para la relación «paises» viola la restricción «check» «cod_letrapais_dosdig»
--DETALLE:  La fila que falla contiene (as093, Afganistan, Kabul).
INSERT INTO paises VALUES ('AS934','Afganistan','Kabul');
--ERROR:  el nuevo registro para la relación «paises» viola la restricción «check» «cod_letrapais_dosdig»
--DETALLE:  La fila que falla contiene (AS0934, Afganistan, Kabul).
INSERT INTO paises VALUES ('ASD93','Afganistan','Kabul');
--ERROR:  el nuevo registro para la relación «paises» viola la restricción «check» «cod_letrapais_dosdig»
--DETALLE:  La fila que falla contiene (ASD093, Afganistan, Kabul).
INSERT INTO paises VALUES ('AS93','Afganistan','Kabul');
--ERROR:  llave duplicada viola restricción de unicidad «pk_codigo_paises»
--DETALLE:  Ya existe la llave (codigo)=(AS093).
```
-------------------------------
### Historial de Conflictos

##### Inserción de datos:

``` sql
INSERT INTO historial_de_conflictos(codigo_conflicto, codigo_pais, num_muertos) VALUES ('1','AS01','35000');
INSERT INTO historial_de_conflictos(codigo_conflicto, codigo_pais, num_heridos) VALUES ('2','AS43','20000');
INSERT INTO historial_de_conflictos VALUES ('3','AF49','120000','70000');
INSERT INTO historial_de_conflictos VALUES ('4','AS32','30000','20000');
INSERT INTO historial_de_conflictos(codigo_conflicto, codigo_pais, num_muertos) VALUES ('5','AF46','25000');
INSERT INTO historial_de_conflictos VALUES ('6','AS38','67500','50000');
INSERT INTO historial_de_conflictos VALUES ('7','AS46','200000','100000');
INSERT INTO historial_de_conflictos VALUES ('8','AS19','12000','6000');
INSERT INTO historial_de_conflictos(codigo_conflicto, codigo_pais, num_muertos) VALUES ('9','AF38','15000');
INSERT INTO historial_de_conflictos VALUES ('10','AF46','400','80');;
```

##### Comprobación de las restrincciones:

``` sql
INSERT INTO historial_de_conflictos VALUES ('1','AS93','70000','35000');
--INSERT 0 1
INSERT INTO historial_de_conflictos VALUES ('1','AS93','70000','35000');
--ERROR:  llave duplicada viola restricción de unicidad «pk_codconflicto_codpais_historial»
--DETALLE:  Ya existe la llave (codigo_conflicto, codigo_pais)=(1, AS093).
INSERT INTO historial_de_conflictos(codigo_conflicto,codigo_pais,num_heridos) VALUES ('2','AS93','70000');
--INSERT 0 1
--select * from historial_de_conflictos;
-- codigo_conflicto | codigo_pais | num_heridos | num_muertos 
--------------------+-------------+-------------+-------------
-- 1                | AS093       |       70000 |       35000
-- 2                | AS093       |       70000 |           0
```
-------------------------------
### Grupos Armados

##### Inserción de datos:

``` sql
INSERT INTO grupos_armados VALUES ('A-111','Ponchos Rojos');
INSERT INTO grupos_armados VALUES ('B-111','Legion Blanca');
INSERT INTO grupos_armados VALUES ('C-111','Monadire');
INSERT INTO grupos_armados VALUES ('A-222','Milicia Urbana');
INSERT INTO grupos_armados VALUES ('B-222','NSKK');
INSERT INTO grupos_armados VALUES ('C-222','Movimiento Liberacion de Sudan');
INSERT INTO grupos_armados VALUES ('A-333','Los Rastrojos');
INSERT INTO grupos_armados VALUES ('B-333','Legion Nauvoo');
INSERT INTO grupos_armados VALUES ('C-333','Legion Al-Rahman');
INSERT INTO grupos_armados VALUES ('A-444','Kamajoh');
INSERT INTO grupos_armados VALUES ('B-444','Werwolf');
INSERT INTO grupos_armados VALUES ('C-444','Resistencia Nacional');
```

##### Comprobación de las restrincciones:

``` sql
INSERT INTO grupos_armados VALUES ('A-111','Ponchos Rojos');
--INSERT 0 1
INSERT INTO grupos_armados VALUES ('AA-111','Ponchos Rojos');
--ERROR:  el nuevo registro para la relación «grupos_armados» viola la restricción «check» «codigo_abc»
--DETALLE:  La fila que falla contiene (AA-111, Ponchos Rojos).
INSERT INTO grupos_armados VALUES ('D-111','Ponchos Rojos');
--ERROR:  el nuevo registro para la relación «grupos_armados» viola la restricción «check» «codigo_abc»
--DETALLE:  La fila que falla contiene (D-111, Ponchos Rojos).
INSERT INTO grupos_armados VALUES ('A111','Ponchos Rojos');
--ERROR:  el nuevo registro para la relación «grupos_armados» viola la restricción «check» «codigo_abc»
--DETALLE:  La fila que falla contiene (A111, Ponchos Rojos).
INSERT INTO grupos_armados VALUES ('A-1111','Ponchos Rojos');
--ERROR:  el nuevo registro para la relación «grupos_armados» viola la restricción «check» «codigo_abc»
--DETALLE:  La fila que falla contiene (A-1111, Ponchos Rojos).
INSERT INTO grupos_armados VALUES ('1A-111','Ponchos Rojos');
--ERROR:  el nuevo registro para la relación «grupos_armados» viola la restricción «check» «codigo_abc»
--DETALLE:  La fila que falla contiene (1A-111, Ponchos Rojos).
INSERT INTO grupos_armados VALUES ('A-111','Ponchos Rojos');
--ERROR:  llave duplicada viola restricción de unicidad «pk_codigo_gruparmados»
--DETALLE:  Ya existe la llave (codigo)=(A-111).
```
-------------------------------
### Historial de Intervenciones Armadas

##### Inserción de datos:

``` sql
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('B-333','5','12/05/2002');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('B-444','10','30/01/2002');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('A-111','1','23/09/2004');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('B-111','2','09/12/2004');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('C-111','3','01/10/2001');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('A-222','4','27/08/2001');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('B-222','5','30/05/2003');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('C-222','6','23/11/2004');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('A-333','7','02/07/2003');
INSERT INTO historial_intervenciones_armadas VALUES ('C-333','8','30/05/2001','14/02/2005');
INSERT INTO historial_intervenciones_armadas VALUES ('C-333','9','29/05/2003','27/08/2007');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('A-444','10','03/06/2001');
INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion) VALUES ('C-444','10','17/08/2006');
```

##### Comprobación de las restrincciones:

``` sql
INSERT INTO historial_intervenciones_armadas VALUES ('B-333','5','12/05/2002');
--INSERT 0 1
INSERT INTO historial_intervenciones_armadas VALUES ('B-333','5','12/04/2001');
--ERROR:  el nuevo registro para la relación «historial_intervenciones_armadas» viola la restricción «check» «mas_abril2001»
--DETALLE:  La fila que falla contiene (B-333, 5, 2001-04-12, null).
INSERT INTO historial_intervenciones_armadas VALUES ('B-333','5','30/04/2001');
--ERROR:  el nuevo registro para la relación «historial_intervenciones_armadas» viola la restricción «check» «mas_abril2001»
--DETALLE:  La fila que falla contiene (B-333, 5, 2001-04-30, null).
INSERT INTO historial_intervenciones_armadas VALUES ('B-333','5','31/04/2001');
--ERROR:  el valor de hora/fecha está fuera de rango: «31/04/2001»
--LÍNEA 1: ...torial_intervenciones_armadas VALUES ('B-333','5','31/04/200...'
--                                                                ^
INSERT INTO historial_intervenciones_armadas VALUES ('B-333','5','01/05/2001');
--INSERT 0 1
INSERT INTO historial_intervenciones_armadas VALUES ('B-333','5','01/05/2001');
--ERROR:  llave duplicada viola restricción de unicidad «pk_codgruparmado_codconflicto_fecha»
--DETALLE:  Ya existe la llave (codigo_gruparmado, codigo_conflicto, fecha_incorporacion)=(B-333, 5, 2001-05-01).
```
-------------------------------
### Organizaciones

##### Inserción de datos:

``` sql
INSERT INTO organizaciones(codigo,nombre,numpersonas_conflicto,tipo) VALUES ('2','Soleil De Afrique','40','Accion Humanitaria');
INSERT INTO organizaciones(codigo,nombre,numpersonas_conflicto,tipo) VALUES ('5','Accion Humana','70','Accion Humanitaria');
INSERT INTO organizaciones(codigo,nombre,numpersonas_conflicto,tipo) VALUES ('6','Accion Verapaz','40','Accion Humanitaria');
INSERT INTO organizaciones VALUES ('4','6','Accion Solidaria','200','Accion Humanitaria');
INSERT INTO organizaciones VALUES ('3','4','Accem','60','Incidencia Politica');
INSERT INTO organizaciones VALUES ('1','2','Luchemos Por La Vida','130','Proyectos de Desarrollo');
```

##### Comprobación de las restrincciones:

``` sql
INSERT INTO organizaciones(codigo,nombre,numpersonas_conflicto,tipo) VALUES ('2','Soleil De Afrique','40','Accion Humanitaria');
--INSERT 0 1
INSERT INTO organizaciones(codigo,nombre,numpersonas_conflicto,tipo) VALUES ('2','Soleil De afrique','40','Accion Humanitaria');
--ERROR:  el nuevo registro para la relación «organizaciones» viola la restricción «check» «iniciales_mayus»
--DETALLE:  La fila que falla contiene (2, null, Soleil De afrique, 40, Accion Humanitaria).
INSERT INTO organizaciones(codigo,nombre,numpersonas_conflicto,tipo) VALUES ('2','Soleil De AFrique','40','Accion Humanitaria');
--ERROR:  el nuevo registro para la relación «organizaciones» viola la restricción «check» «iniciales_mayus»
--DETALLE:  La fila que falla contiene (2, null, Soleil De AFrique, 40, Accion Humanitaria).
INSERT INTO organizaciones(codigo,nombre,numpersonas_conflicto,tipo) VALUES ('2','SOLEIL De Afrique','40','Accion Humanitaria');
--ERROR:  el nuevo registro para la relación «organizaciones» viola la restricción «check» «iniciales_mayus»
--DETALLE:  La fila que falla contiene (2, null, SOLEIL De Afrique, 40, Accion Humanitaria).
INSERT INTO organizaciones(codigo,nombre,numpersonas_conflicto,tipo) VALUES ('2','Soleil De Afrique','40','Accion Humanitaria');
--ERROR:  llave duplicada viola restricción de unicidad «pk_codigo_organizaciones»
--DETALLE:  Ya existe la llave (codigo)=(2).
```
-------------------------------
### Historial de Intervenciones Mediadoras

##### Inserción de datos:

``` sql
INSERT INTO historial_intervenciones_mediadoras(codigo_org,codigo_conflicto,fecha_incorporacion,tipo_ayuda) VALUES ('1','4','12/05/2001','Trabajo social');
INSERT INTO historial_intervenciones_mediadoras(codigo_org,codigo_conflicto,fecha_incorporacion,tipo_ayuda) VALUES ('2','5','02/07/2003','Trabajo socila');
INSERT INTO historial_intervenciones_mediadoras VALUES ('3','8','30/03/1995','14/02/2005','Trabajo de transporte');
INSERT INTO historial_intervenciones_mediadoras(codigo_org,codigo_conflicto,fecha_incorporacion,tipo_ayuda) VALUES ('4','7','03/02/2001','Trabajo de transporte');
INSERT INTO historial_intervenciones_mediadoras(codigo_org,codigo_conflicto,fecha_incorporacion,tipo_ayuda) VALUES ('5','2','17/08/2000','Representante intermediario');
INSERT INTO historial_intervenciones_mediadoras VALUES ('6','9','29/05/1997','27/08/2007','Trabajo social');
INSERT INTO historial_intervenciones_mediadoras(codigo_org,codigo_conflicto,fecha_incorporacion,tipo_ayuda) VALUES ('1','10','23/09/2004','Trabajo de social');
INSERT INTO historial_intervenciones_mediadoras(codigo_org,codigo_conflicto,fecha_incorporacion,tipo_ayuda) VALUES ('4','2','1/10/1997','Representante intermediario');
INSERT INTO historial_intervenciones_mediadoras VALUES ('4','9','29/12/1997','27/08/2007','Trabajo de transporte');
INSERT INTO historial_intervenciones_mediadoras(codigo_org,codigo_conflicto,fecha_incorporacion,tipo_ayuda) VALUES ('1','7','05/04/1998','Trabajo social');
```

-------------------------------
### Envios

##### Inserción de datos:

``` sql
INSERT INTO envios VALUES ('1','1','15/08/2003 8:30');
INSERT INTO envios VALUES ('2','2','01/05/2000 12:45');
INSERT INTO envios VALUES ('3','3','01/05/1999 13:23');
INSERT INTO envios VALUES ('4','4','15/09/2008 15:00');
INSERT INTO envios VALUES ('5','5','15/10/1997 09:25');
INSERT INTO envios VALUES ('6','6','15/12/1999 10:15');
INSERT INTO envios VALUES ('7','4','01/04/2005 09:45');
INSERT INTO envios VALUES ('8','6','01/05/2003 11:30');
INSERT INTO envios VALUES ('9','4','01/07/2009 10:25');
INSERT INTO envios VALUES ('10','3','15/02/2005 10:35');
INSERT INTO envios VALUES ('11','3','01/08/2007 12:00');
INSERT INTO envios VALUES ('12','5','15/05/2008 12:45');
```

##### Comprobación de las restrincciones:

``` sql
INSERT INTO envios VALUES ('1','1','15/08/2003 8:30');
--INSERT 0 1
INSERT INTO envios VALUES ('1','1','15/08/2003 7:59');
--ERROR:  el nuevo registro para la relación «envios» viola la restricción «check» «entre8_16»
--DETALLE:  La fila que falla contiene (1, 1, 2003-08-15 07:59:00).
INSERT INTO envios VALUES ('1','1','15/08/2003 7:00');
--ERROR:  el nuevo registro para la relación «envios» viola la restricción «check» «entre8_16»
--DETALLE:  La fila que falla contiene (1, 1, 2003-08-15 07:00:00).
INSERT INTO envios VALUES ('1','1','15/08/2003 16:01');
--ERROR:  el nuevo registro para la relación «envios» viola la restricción «check» «entre8_16»
--DETALLE:  La fila que falla contiene (1, 1, 2003-08-15 16:01:00).
INSERT INTO envios VALUES ('1','1','15/08/2003 16:10');
--ERROR:  el nuevo registro para la relación «envios» viola la restricción «check» «entre8_16»
--DETALLE:  La fila que falla contiene (1, 1, 2003-08-15 16:10:00).
INSERT INTO envios VALUES ('1','1','15/08/2003 13:00');
--ERROR:  llave duplicada viola restricción de unicidad «pk_codigo_envios»
--DETALLE:  Ya existe la llave (codigo)=(1).
```

##### Comprobación de las restrincciones:

``` sql
INSERT INTO envios VALUES ('1','1','15/08/2003 8:30');
--INSERT 0 1
    INSERT INTO envios VALUES ('1','1','14/08/2003 8:30');
--ERROR:  el nuevo registro para la relación «envios» viola la restricción «check» «dia1_15»
--DETALLE:  La fila que falla contiene (1, 1, 2003-08-14 08:30:00).
INSERT INTO envios VALUES ('1','1','16/08/2003 8:30');
--ERROR:  el nuevo registro para la relación «envios» viola la restricción «check» «dia1_15»
--DETALLE:  La fila que falla contiene (1, 1, 2003-08-16 08:30:00).
INSERT INTO envios VALUES ('1','1','02/08/2003 8:30');
--ERROR:  el nuevo registro para la relación «envios» viola la restricción «check» «dia1_15»
--DETALLE:  La fila que falla contiene (1, 1, 2003-08-02 08:30:00).
INSERT INTO envios VALUES ('1','1','31/08/2003 8:30');
--ERROR:  el nuevo registro para la relación «envios» viola la restricción «check» «dia1_15»
--DETALLE:  La fila que falla contiene (1, 1, 2003-08-31 08:30:00).
```
-------------------------------
### Productos

##### Inserción de datos:

``` sql
INSERT INTO productos VALUES ('Legumbres','Alimento, alto contenido en proteinas (sacos de 10 Kg)');
INSERT INTO productos VALUES ('Cereales','Alimento, alto contenido de carbohidratos (cajas de 500g)');
INSERT INTO productos VALUES ('Sal','Ayuda alimentaria (Paquetes de 10 Kg)');
INSERT INTO productos VALUES ('Azucar','Ayuda alimentaria, alto contenido calorico (paquetes de 10 Kg)');
INSERT INTO productos VALUES ('Pescado','Alimento, alto contenido en proteinas (por piezas)');
INSERT INTO productos VALUES ('Carne Enlatada','Alimento, alto contenido en proteninas (Por latas)');
INSERT INTO productos VALUES ('Agua','Esencial para vivir (garrafas de 50 L)');
INSERT INTO productos VALUES ('Aceite','Ayuda alimentaria, alto contenido calorico (garrafas de 20L)');
INSERT INTO productos VALUES ('Calzado','Proteccion para el cuerpo (por pares)');
INSERT INTO productos VALUES ('Juguetes','Ayuda para el anime de los niños (por unidades)');
```

-------------------------------
### Campos de Refugiados

##### Inserción de datos:

``` sql
INSERT INTO campos_refugiados VALUES ('1','Dadaab','Hagadera');
INSERT INTO campos_refugiados VALUES ('2','Dollo Ado','Eritrea');
INSERT INTO campos_refugiados VALUES ('3','Kakuma','Sur de Kenia');
INSERT INTO campos_refugiados VALUES ('4','Jabalia','Gaza');
INSERT INTO campos_refugiados VALUES ('5','Al Zaatari','Jordania');
INSERT INTO campos_refugiados VALUES ('6','Katumba','Tanzania');
INSERT INTO campos_refugiados VALUES ('7','Panian','Norte de Pakistan');
INSERT INTO campos_refugiados VALUES ('8','Yida','Sudan del Sur');
```

-------------------------------
### Paquetes


##### Inserción de datos:

``` sql
INSERT INTO paquetes VALUES ('1','1','1','Legumbres',10);
INSERT INTO paquetes VALUES ('2','1','1','Cereales',50);
INSERT INTO paquetes VALUES ('3','2','2','Sal',20);
INSERT INTO paquetes VALUES ('4','3','3','Sal',20);
INSERT INTO paquetes VALUES ('5','4','4','Azucar',20);
INSERT INTO paquetes VALUES ('6','5','5','Pescado',30);
INSERT INTO paquetes VALUES ('7','6','6','Carne Enlatada',100);
INSERT INTO paquetes VALUES ('8','6','6','Agua',20);
INSERT INTO paquetes VALUES ('9','7','7','Agua',20);
INSERT INTO paquetes VALUES ('10','8','8','Agua',30);
INSERT INTO paquetes VALUES ('11','8','8','Calzado',100);
INSERT INTO paquetes VALUES ('12','4','4','Juguetes',40);
INSERT INTO paquetes VALUES ('13','5','9','Legumbres',20);
INSERT INTO paquetes VALUES ('14','3','10','Legumbres',20);
INSERT INTO paquetes VALUES ('15','2','11','Cereales',50);
INSERT INTO paquetes VALUES ('16','7','12','Sal',30);
INSERT INTO paquetes VALUES ('17','3','10','Legumbres',20);
```

-------------------------------

### Refugiados

##### Inserción de datos:

``` sql
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('1','1','Muminah Hawazin Aswad','18',           '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('2','1','Ghayda Jawna Mansour','34',            '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('3','1','Najla Nafisah Sarkis','65',            '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('4','1','Wasfiyah Nadiyah Amari','23',          '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('5','1','Niyaf Layan Bishara','45',             '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('6','1','Fatimah Tibah Tahan','56',             '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('7','1','Rubaa Khawlah Bata','23',              '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('8','1','Baligh Abdul-Razzaq Fakhoury','16',    '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('9','1','Aliyyah Ranim Sarkis','23',            '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('10','1','Muhtadi Shareef Srour','64',          '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('11','1','Bahira Makaarim Abadi','82',          '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('12','1','Musnah Kawkab Botros','23',           '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('13','1','Zaki Jihad Isa','12',                 '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('14','1','Busr Badra Asfour','2',               '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('15','1','Radiyah Saidah Basara','5',           '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('16','1','Mansur Sariyah Shammas','53',         '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('17','1','Salihah Fadilah Toma','3',            '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('18','1','Kazim Naji Haik','9',                 '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('19','1','Basilah Sayyidah Arian','13',         '17/08/2000');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('20','1','Habbab Marwan Morcos','15',           '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('21','1','Wasil Imad al Din Handal','23',       '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('22','1','Bilqees Subhah Srour','56',           '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('23','1','Ubaydah Safi Maloof','96',            '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('24','1','Imran Abdul-Muid Halabi','14',        '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('25','1','Sahl Jad Allah Ganem','23',           '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('26','1','Abdul-Adl Haroun Boutros','15',       '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('27','1','Ruwaydah Istilah Kanaan','11',        '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('28','1','Wadi Bakri Morcos','33',              '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('29','1','Muna Subhiyah Issa','22',             '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('30','1','Nahla Wafiqah Shammas','67',          '12/05/2001');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('31','1','Abdul-Nasir Shadi Wasem','16',        '12/05/2001');
INSERT INTO Refugiados VALUES ('32','2','Kateb Khalil Zogby','18',                  '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('33','2','Kareef Ghassan Bishara','20',              '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('34','2','Hameeda Kareema Antoun','30',              '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('35','2','Rafif Khalisah Kattan','21',               '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('36','2','Karim Abdul-Khaliq Hanania','45',          '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('37','2','Juwayriyah Huriyah Sabbagh','43',          '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('38','2','Amir Safwan Hadad','24',                   '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('39','2','Sumaira Khulood Bitar','42',               '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('40','2','Rasul Fareeq Mansour','1',                 '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('41','2','Sura Suha Bahar','2',                      '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('42','2','Muizz Nusrah Toma','14',                   '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('43','2','Arfan Sameh Gaber','34',                   '30/03/1995','23/12/2003');
INSERT INTO Refugiados VALUES ('44','2','Haniya Shiyam Sarraf','11',                '30/05/1996','23/12/2003');
INSERT INTO Refugiados VALUES ('45','2','Sumayra Wordah Hanania','18',              '30/05/1996','23/12/2003');
INSERT INTO Refugiados VALUES ('46','2','Amid Nail Seif','78',                      '30/05/1996','23/12/2003');
INSERT INTO Refugiados VALUES ('47','2','Mufeed Nabil Sabbagh','45',                '30/05/1996','23/12/2003');
INSERT INTO Refugiados VALUES ('48','2','Amala Raaida Guirguis','31',               '30/05/1996','23/12/2003');
INSERT INTO Refugiados VALUES ('49','2','Wafeeq Shahin Maloof','22',                '30/05/1996','23/12/2003');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('50','3','Fatin Munahid Daher','4',             '29/05/1997');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('51','3','Ala al din Nuhayd Sayegh','67',       '29/05/1997');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('52','3','Malakah Adeela Abadi','21',           '29/05/1997');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('53','3','Abdul-Sami Abdul-Sami Shamon','5',    '29/05/1997');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('54','3','Sajidah Khawlah Haddad','18',         '29/05/1997');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('55','3','Adel Nasir Masih','18',               '29/05/1997');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('56','3','Abdel Mohammed Mustafa','56',         '29/05/1997');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('57','3','Budur Hudun Assaf','21',              '19/04/1998');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('58','3','Sukainah Nadira Toma','18',           '19/04/1998');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('59','3','Abal Baraah Cham','2',                '19/04/1998');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('60','3','Jumaana Mukhlisah Totah','5',         '19/04/1998');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('61','3','Karimah Rihana Bitar','79',           '19/04/1998');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('62','4','Adan Duqaq Naifeh','57',              '02/07/2003');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('63','4','Kareema Suhaymah Tuma','43',          '02/07/2003');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('64','4','Qais Qudamah Aswad','12',             '02/07/2003');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('65','4','Rabi Ahmed Mansour','56',             '02/07/2003');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('66','4','Ilhaam Aliah Antoun','32',            '02/09/2003');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('67','4','Abdul-Salam Shakir Seif','1',         '02/09/2003');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('68','4','Samah Samiyah Attia','5',             '02/09/2003');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('69','5','Butrus Sadun Sayegh','23',            '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('70','5','Mumayyaz Afra Shadid','31',           '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('71','5','Mubin Boulos Abadi','21',             '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('72','5','Nimah Intisaar Isa','1',              '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('73','5','Naif Mishaal Atiyeh','90',            '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('74','5','Subhi Abdul-Karim Botros','102',      '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('75','5','Abdul-Aziz Ubadah Aswad','25',        '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('76','5','Janan Aida Naser','4',                '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('77','5','Wafiqah Asalah Maalouf','18',         '23/09/2008');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('78','6','Irfan Abdul-Samad Najjar','26',       '23/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('79','6','Rumaythah Walaa Shammas','4',         '23/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('80','6','Mulhim Abdul-Muizz Daher','7',        '23/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('81','6','Dunyana Ghusun Wasem','1',            '23/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('82','6','Isa Taim Allah Bishara','5',          '23/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('83','6','Bilqis Mahabbah Sabbag','19',         '23/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('84','6','Abdul-Mubdi Umar Sayegh','20',        '23/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('85','6','Nahla Ramlah Ghannam','44',           '23/11/2006');
INSERT INTO Refugiados VALUES ('86','7','Ratib Wadi Naser','55',                    '20/01/2001','06/11/2006');
INSERT INTO Refugiados VALUES ('87','7','Samar Wardah Asker','33',                  '20/01/2001','06/11/2006');
INSERT INTO Refugiados VALUES ('88','7','Taqwa Luban Wasem','1',                    '20/01/2001','06/11/2006');
INSERT INTO Refugiados VALUES ('89','7','Ishfaq Nabihah Samaha','34',               '20/01/2001','06/11/2006');
INSERT INTO Refugiados VALUES ('90','7','Nuaym Amid Essa','67',                     '20/01/2001','06/11/2006');
INSERT INTO Refugiados VALUES ('91','7','Rushd Sayyar Ghanem','2',                  '20/01/2001','06/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('92','7','Shukrah Jun Wasem','66',              '20/01/2001');
INSERT INTO Refugiados VALUES ('93','7','Asriyah Sayyidah Gerges','12',             '20/01/2001','06/11/2006');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('94','8','Kadir Tahsin Khoury','4',             '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('95','8','Badi al Zaman Kahil Asker','15',      '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('96','8','Shaymaa Ijlal Bishara','2',           '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('97','8','Humayrah Asma Bitar','18',            '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('98','8','Munisah Nusaibah Bitar','18',         '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('99','8','Husayn Murad Boulos','64',            '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('100','8','Rafid Rabi Shamon','43',             '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('101','8','Hadiyah Balqis Mansour','25',        '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('102','8','Niyaf Nida Shalhoub','6',            '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('103','8','Nihad Khuzama Ganem','36',           '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('104','8','Raja Istilah Bitar','21',            '26/04/1999');
INSERT INTO Refugiados(codigo,codigo_refugio,nombre,edad,fecha_llegada) VALUES ('105','8','Maysoon Dhakirah Zogby','86',        '26/04/1999');
```
-------------------------------