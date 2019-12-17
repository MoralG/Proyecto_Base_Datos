# MySQL
# Fase 3: Explotación de la Base de Datos. Operaciones DML


### Tarea 1. 

* El grupo armado que fue el primero en participar en el conflicto que suma un mayor número de muertos y heridos hasta la fecha se ha incorporado hoy al conflicto de causa religiosa en el que más intervenciones mediadoras se han producido por parte de organizaciones mediadoras que no dependen de otras. Inserta el registro adecuado mediante una consulta de datos anexados.

``` sql
CREATE VIEW suma_heridos_muertos AS
SELECT codigo_conflicto, SUM(num_heridos) + SUM(num_muertos) AS sum_her_muer
FROM historial_de_conflictos
GROUP BY codigo_conflicto;

CREATE VIEW total_organizaciones AS
SELECT COUNT(codigo_org) AS contar_organizaciones
FROM historial_intervenciones_mediadoras
WHERE codigo_org in (SELECT codigo
                     FROM organizaciones
                     WHERE codigo_orgdepen IS NULL)
GROUP BY codigo_conflicto;

INSERT INTO historial_intervenciones_armadas(codigo_gruparmado,codigo_conflicto,fecha_incorporacion)
SELECT (SELECT codigo_gruparmado
       FROM historial_intervenciones_armadas
       WHERE codigo_conflicto = (SELECT codigo_conflicto
                                 FROM historial_de_conflictos
                                 GROUP BY codigo_conflicto
                                 HAVING SUM(num_heridos) + SUM(num_muertos) = (SELECT MAX(sum_her_muer)
                                                                               FROM suma_heridos_muertos))
       AND fecha_incorporacion = (SELECT MIN(fecha_incorporacion)
                                  FROM historial_intervenciones_armadas
                                  WHERE codigo_conflicto = (SELECT codigo_conflicto
                                                            FROM historial_de_conflictos
                                                            GROUP BY codigo_conflicto
                                                            HAVING SUM(num_heridos) + SUM(num_muertos) = (SELECT MAX(sum_her_muer)
                                                                                                          FROM suma_heridos_muertos))
                                  GROUP BY codigo_conflicto)),
(SELECT codigo_conflicto
FROM historial_intervenciones_mediadoras
WHERE codigo_conflicto in (SELECT codigo 
                           FROM conflictos
                           WHERE causa = 'Religioso')
GROUP BY codigo_conflicto, codigo_org
HAVING COUNT(codigo_org) = (SELECT MAX(contar_organizaciones)
                            FROM total_organizaciones)
AND codigo_org in (SELECT codigo
                     FROM organizaciones
                     WHERE codigo_orgdepen IS NULL)),
current_date;
```

### Tarea 2. 

* Se ha producido un acto de guerra en Kabul con el resultado de 12 muertos y 50 heridos. Dicho acto está relacionado con el conflicto en el que participa el grupo armado “Persa”. Actualiza la base de datos mediante una consulta de actualización.

``` sql
UPDATE historial_de_conflictos SET num_heridos = num_heridos + 50, num_muertos = num_muertos + 12
WHERE codigo_pais = (SELECT codigo
                     FROM paises
                     WHERE capital = 'Kabul')
AND codigo_conflicto = (SELECT codigo_conflicto
                        FROM historial_intervenciones_armadas
                        WHERE codigo_gruparmado = (SELECT codigo
                                                   FROM grupos_armados
                                                   WHERE nombre = 'Ponchos Rojos'));
```

### Tarea 3. 

* Muestra el número total de víctimas (muertos y heridos) que han causado los conflictos bélicos en cada país, incluyendo los países en los que no han habido víctimas.

``` sql
SELECT p.nombre AS "Paises", SUM(num_heridos) AS "Nº de heridos", SUM(num_muertos) AS "Nº de muertos"
FROM historial_de_conflictos hc
LEFT JOIN paises p
ON hc.codigo_pais = p.codigo
GROUP BY p.codigo;
```

### Tarea 4. 

* Muestra los campos de refugiados en los que hay más niños que adultos según el último censo efectuado a los que se hayan mandado menos de 10 litros de leche por niño en los últimos tres meses.

``` sql
SELECT nombre AS "Campos de refugiados"
FROM campos_refugiados
WHERE codigo in (SELECT e.codigo_refugio
                 FROM refugiados e
                 WHERE ((extract(year FROM current_date) - extract(year FROM e.fecha_llegada)) *12) + extract(MONTH FROM current_date) - extract(MONTH FROM e.fecha_llegada) <= 3
                 AND e.edad < '18'
                 GROUP BY e.codigo_refugio
                 HAVING COUNT(e.codigo_refugio) > (SELECT COUNT(codigo_refugio)
                                                   FROM refugiados
                                                   WHERE ((extract(year FROM current_date) - extract(year FROM fecha_llegada)) *12) + extract(MONTH FROM current_date) - extract(MONTH FROM fecha_llegada) <= 3
                                                   AND edad > '18'
                                                   AND codigo_refugio = e.codigo_refugio
                                                   GROUP BY codigo_refugio)
                 AND codigo_refugio in (SELECT codigo_refugio
                                        FROM paquetes
                                        WHERE nombre_producto = 'Leche'
                                        GROUP BY codigo_refugio
                                        HAVING sum(cantidad)/(SELECT COUNT(e.codigo)
                                                              FROM refugiados e
                                                              WHERE ((extract(year FROM current_date) - extract(year FROM e.fecha_llegada)) *12) + extract(MONTH FROM current_date) - extract(MONTH FROM e.fecha_llegada) <= 3
                                                              AND e.edad < '18'
                                                              GROUP BY e.codigo_refugio
                                                              HAVING COUNT(e.codigo_refugio) > (SELECT COUNT(codigo_refugio)
                                                                                                FROM refugiados
                                                                                                WHERE ((extract(year FROM current_date) - extract(year FROM fecha_llegada)) *12) + extract(MONTH FROM current_date) - extract(MONTH FROM fecha_llegada) <= 3
                                                                                                AND edad > '18'
                                                                                                AND codigo_refugio = e.codigo_refugio
                                                                                                GROUP BY codigo_refugio)) < 10));
```

### Tarea 5. 

* Muestra los nombres de los conflictos en los que se han realizado intervenciones mediadoras tanto en 2013 como en 2014 y en 2015.

``` sql
SELECT codigo AS "Codigo", nombre AS "Conflicto"
FROM conflictos
WHERE codigo in (SELECT codigo_conflicto
                 FROM historial_intervenciones_mediadoras
                 WHERE extract(YEAR FROM fecha_incorporacion) = '2013'
                 INTERSECT
                 SELECT codigo_conflicto
                 FROM historial_intervenciones_mediadoras
                 WHERE extract(YEAR FROM fecha_incorporacion) = '2014'
                 INTERSECT
                 SELECT codigo_conflicto
                 FROM historial_intervenciones_mediadoras
                 WHERE extract(YEAR FROM fecha_incorporacion) = '2015');

```

### Tarea 6. 

* Muestra para cada campo de refugiados el total de envíos que incluían leche en polvo que se han realizado en los últimos seis meses.

``` sql
SELECT cam.nombre AS "Nom. Campo de Refugiado", COUNT(codigo_envio) AS "Total de envios"
FROM campos_refugiados cam, paquetes paq
WHERE cam.codigo = paq.codigo_refugio
AND paq.codigo_envio in (SELECT env.codigo
                         FROM envios env, paquetes paq
                         WHERE env.codigo = paq.codigo_envio
                         AND ((extract(year FROM current_date) - extract(year FROM fecha_hora)) *12) + extract(MONTH FROM current_date) - extract(MONTH FROM fecha_hora) <= 6
                         AND UPPER(paq.nombre_producto) = 'LECHE EN POLVO')
GROUP BY cam.nombre;
```

### Tarea 7. 

* Muestra los nombres de los países que están involucrados en el conflicto religioso que lleva activo desde hace más tiempo.

``` sql
CREATE VIEW conflictos_activos AS 
SELECT MAX(DATEDIFF(current_date, fecha_incorporacion)) AS max_activo
FROM historial_intervenciones_armadas his, conflictos con
WHERE con.codigo = his.codigo_conflicto
AND his.fecha_retirada IS NULL
AND UPPER(con.causa) = 'RELIGIOSO'
GROUP BY codigo_conflicto;


SELECT nombre AS "Pais"
FROM paises
WHERE codigo in (SELECT codigo_pais
                FROM historial_de_conflictos
                WHERE codigo_conflicto in (SELECT codigo_conflicto
                                          FROM historial_intervenciones_armadas
                                          WHERE DATEDIFF(current_date, fecha_incorporacion) in (SELECT MAX(max_activo)
                                                                                                    FROM conflictos_activos)));
```

### Tarea 8. 

* Muestra el producto envíado en mayor cantidad en cada uno de los envíos realizados por organizaciones mediadores dependientes de la ONU.

``` sql
CREATE VIEW producto_mayor AS 
SELECT codigo_envio, sum(cantidad) AS cuenta_producto
FROM paquetes
GROUP BY codigo_envio, nombre_producto;

SELECT codigo_envio AS "Codigo de envio", nombre_producto AS "Producto", sum(cantidad) AS "Cantidad total"
FROM paquetes
GROUP BY codigo_envio, nombre_producto
HAVING (codigo_envio, sum(cantidad)) in (SELECT codigo_envio, MAX(cuenta_producto)
                                         FROM producto_mayor
                                         GROUP BY codigo_envio)
AND codigo_envio in (SELECT codigo
                     FROM envios
                     WHERE codigo_org in (SELECT codigo
                                          FROM organizaciones
                                          WHERE codigo_orgdepen = 'ONU'));
```

### Tarea 9. 

* Muestra el nombre de las organizaciones mediadoras que no han realizado ningún envío al campo de refugiados más poblado según el último censo realizado en el mismo.

``` sql
CREATE VIEW max_poblado AS
SELECT COUNT(codigo) AS total_refugiados
FROM refugiados
WHERE fecha_salida IS NULL
GROUP BY codigo_refugio;

SELECT codigo AS "Codigo", nombre AS "Oraganizaciones"
FROM organizaciones
WHERE codigo NOT IN (SELECT DISTINCT codigo_org
                     FROM envios env, paquetes paq
                     WHERE env.codigo = paq.codigo_envio
                     AND codigo_refugio = (SELECT codigo_refugio
                                               FROM refugiados
                                               WHERE fecha_salida IS NULL
                                               GROUP BY codigo_refugio
                                               HAVING COUNT(codigo) = (SELECT MAX(total_refugiados)
                                                                       FROM max_poblado)));
```

### Tarea 10. 

* Crea una vista con los nombres de los grupos armados que se han retirado de todos los conflictos en los que han participado junto con la fecha en que se retiraron del último de ellos.

``` sql
CREATE VIEW maximas_fechas AS
SELECT MAX(fecha_retirada) AS max_fecha 
FROM historial_intervenciones_armadas
GROUP BY codigo_conflicto;

CREATE VIEW gruparmados_retirados AS
SELECT gru.nombre AS "Grupo armado", his.fecha_retirada AS "Ultima fecha retirada"
FROM historial_intervenciones_armadas his, grupos_armados gru
WHERE his.codigo_gruparmado = gru.codigo
AND fecha_retirada IS NOT NULL
AND codigo_gruparmado NOT IN (SELECT codigo_gruparmado
                              FROM historial_intervenciones_armadas
                              WHERE fecha_retirada IS NULL)
AND fecha_retirada in (SELECT MAX(max_fecha)
                       FROM maximas_fechas)
GROUP BY gru.nombre, fecha_retirada;
```