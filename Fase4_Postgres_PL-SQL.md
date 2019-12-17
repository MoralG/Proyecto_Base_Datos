# Oracle

# Fase 4: Explotación de la Base de Datos mediante PL/SQL

### Tarea 1.

* Escribe una función que reciba un nombre de producto, un código de campo de refugiados y dos fechas y devuelva el número de unidades de ese producto que se ha recibido en dicho campo entre las dos fechas.

``` sql
CREATE OR REPLACE FUNCTION SaberUnidadesTotales(p_NombreProducto productos.nombre%TYPE,
                                                p_CodCampoRefugiado campos_refugiados.codigo%TYPE,
                                                p_FechaInicio envios.fecha_hora%TYPE,
                                                p_FechaFin envios.fecha_hora%TYPE)
returns NUMERIC AS $$
DECLARE
    v_CantidadTotal NUMERIC := 0;
BEGIN
    PERFORM ComprobarExcepciones(p_NombreProducto, p_CodCampoRefugiado, p_FechaInicio, p_FechaFin);
    v_CantidadTotal:=SaberCodigoEnviosValidos(p_NombreProducto, p_CodCampoRefugiado, p_FechaInicio, p_FechaFin, v_CantidadTotal);
    if v_CantidadTotal > 0 then
        return v_CantidadTotal;
    else
        RAISE EXCEPTION 'No se ha enviado ninguna cantidad de %',p_NombreProducto;
    end if;
END;
$$ LANGUAGE plpgsql;

----------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION SaberCodigoEnviosValidos(p_NombreProducto productos.nombre%TYPE,
                                                     p_CodCampoRefugiado campos_refugiados.codigo%TYPE,
                                                     p_FechaInicio envios.fecha_hora%TYPE,
                                                     p_FechaFin envios.fecha_hora%TYPE)
returns NUMERIC AS $$
DECLARE
    reg RECORD;
    c_CodigoEnvios CURSOR FOR
    SELECT codigo
    FROM envios
    WHERE to_char(fecha_hora,'YYYY/MM/DD') >= to_char(p_FechaInicio,'YYYY/MM/DD') 
    AND to_char(fecha_hora,'YYYY/MM/DD') <= to_char(p_FechaFin,'YYYY/MM/DD'); 
    v_CantidadTotal paquetes.cantidad%TYPE := 0;
BEGIN
    for reg in c_CodigoEnvios loop
        v_CantidadTotal:=SaberCantidadTotal(reg.codigo, p_CodCampoRefugiado, p_NombreProducto, v_CantidadTotal);
    end loop;
    return v_CantidadTotal;
END;
$$ LANGUAGE plpgsql;

----------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION SaberCantidadTotal(p_CodigoEnvio VARCHAR,
                                               p_CodigoCampo VARCHAR,
                                               p_NombreProducto VARCHAR,
                                               p_CantidadTotal NUMERIC)
returns NUMERIC AS $$
DECLARE
    reg RECORD;
    c_CantidadProducto CURSOR FOR
    SELECT cantidad
    FROM paquetes
    WHERE codigo_envio = p_CodigoEnvio
    AND codigo_refugiados = p_CodigoCampo
    AND nombre_producto = p_NombreProducto;
BEGIN
    for reg in c_CantidadProducto loop
        p_CantidadTotal := reg.cantidad + p_CantidadTotal;
    end loop;
    return p_CantidadTotal;
END;
$$ LANGUAGE plpgsql;

----------------------------------------------------------------------------------------
SELECT ComprobarExcepciones('Legumbres','140','1990/04/01','2009/04/01');
CREATE OR REPLACE FUNCTION ComprobarExcepciones(p_NombreProducto productos.nombre%TYPE,
                                                 p_CodCampoRefugiado campos_refugiados.codigo%TYPE,
                                                 p_FechaInicio envios.fecha_hora%TYPE,
                                                 p_FechaFin envios.fecha_hora%TYPE)
returns VOID AS $$
DECLARE
BEGIN
    PERFORM ExceptionProductoInexistente(p_NombreProducto);
    PERFORM ExceptionCodCampoInexistente(p_CodCampoRefugiado);
    PERFORM ExceptionEnvioEntreFechas(p_FechaInicio, p_FechaFin);
END;
$$ LANGUAGE plpgsql;

----------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION ExceptionProductoInexistente (p_NombreProducto productos.nombre%TYPE)
returns VOID AS $$
DECLARE
    v_ContadorDeProducto NUMERIC;
BEGIN
    SELECT count(nombre) INTO v_ContadorDeProducto
    FROM productos
    WHERE nombre = p_NombreProducto;
    if v_ContadorDeProducto = 0 then
        RAISE EXCEPTION 'ERROR, Nombre de producto no encontrado' ;
    end if;
END;
$$ LANGUAGE plpgsql;

----------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION ExceptionCodCampoInexistente(p_CodigoCampo campos_refugiados.codigo%TYPE)
returns VOID AS $$
DECLARE
    v_ContadorCodCampo NUMERIC;
BEGIN
    SELECT count(codigo) INTO v_ContadorCodCampo
    FROM campos_refugiados
    WHERE codigo = p_CodigoCampo;
    if v_ContadorCodCampo = 0 then
        RAISE EXCEPTION 'ERROR, Codigo de campo de refugido no encontrado';
    end if;
END;
$$ LANGUAGE plpgsql;

---------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION ExceptionEnvioEntreFechas(p_FechaInicio envios.fecha_hora%TYPE,
                                                      p_FechaFin envios.fecha_hora%TYPE)
returns VOID AS $$
DECLARE
    v_ContadorEnvios NUMERIC;
BEGIN
    SELECT count(codigo) INTO v_ContadorEnvios
    FROM envios
    WHERE to_char(fecha_hora,'YYYY/MM/DD') >= to_char(p_FechaInicio,'YYYY/MM/DD')
    AND to_char(fecha_hora,'YYYY/MM/DD') <= to_char(p_FechaFin,'YYYY/MM/DD');
    if v_ContadorEnvios = 0 then
        RAISE EXCEPTION 'No hay envios entre las dos fechas dadas';
    end if;
END;
$$ LANGUAGE plpgsql;
```

### Tarea 3.

* Realizar un trigger que garantice que una organización mediadora con menos de diez personas desplegadas en un conflicto no pueda ofrecer ayuda de tipo Ayuda Humanitaria.

``` sql
CREATE OR REPLACE FUNCTION ORGNoMenosDiezUpdate() 
RETURNS TRIGGER AS $ORGNoMenosDiez$ 
BEGIN
    PERFORM ComprobarDatos(old.numpersonas_conflicto, new.numpersonas_conflicto, old.tipo, new.tipo);
    RETURN NEW;
END;
$ORGNoMenosDiez$ LANGUAGE plpgsql;


CREATE OR REPLACE FUNCTION ComprobarDatosUpdate(p_NumPersonasAntes NUMERIC,
                                                p_NumPersonasDespues NUMERIC,
                                                p_TipoAyudaAntes VARCHAR,
                                                p_TipoAyudaDespues VARCHAR)
returns VOID AS $$
DECLARE
BEGIN
    if p_NumPersonasAntes < 10 AND p_TipoAyudaDespues = 'Accion Humanitaria' then
        RAISE EXCEPTION 'ERROR, Tipo de ayuda no valido';
    elseif p_NumPersonasDespues < 10 AND p_TipoAyudaAntes = 'Accion Humanitaria' then
        RAISE EXCEPTION 'ERROR, Numero de personas no valido';
    end if;
END;
$$ LANGUAGE plpgsql;


CREATE TRIGGER ORGNoMenosDiezUpdate BEFORE UPDATE
    ON organizaciones FOR EACH row
    EXECUTE PROCEDURE ORGNoMenosDiezUpdate();

------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION ORGNoMenosDiezInsert() 
RETURNS TRIGGER AS $ORGNoMenosDiez$ 
BEGIN
    if new.numpersonas_conflicto < 10 AND new.tipo = 'Accion Humanitaria' then
    RAISE EXCEPTION 'ERROR, Numero de personas y tipo no valido';
    end if;
    RETURN NEW;
END;
$ORGNoMenosDiez$ LANGUAGE plpgsql;


CREATE TRIGGER ORGNoMenosDiezInsert BEFORE INSERT
ON organizaciones FOR EACH row
    EXECUTE PROCEDURE ORGNoMenosDiezInsert();
```