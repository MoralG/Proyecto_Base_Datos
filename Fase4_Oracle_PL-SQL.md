# Oracle

# Fase 4: Explotación de la Base de Datos mediante PL/SQL

### Tarea 1. 

* Escribe una función que reciba un nombre de producto, un código de campo de refugiados y dos fechas y devuelva el número de unidades de ese producto que se ha recibido en dicho campo entre las dos fechas.

``` sql
SET SERVEROUTPUT ON
SELECT SaberUnidadesTotales('Legumbres','1',to_date('2001/04/01', 'YYYY/MM/DD'), to_date('2004/04/01', 'YYYY/MM/DD')) 
AS UnidadesTotales 
FROM dual;

----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION SaberUnidadesTotales (p_NombreProducto productos.nombre%TYPE,
                                                 p_CodCampoRefugiado campos_refugiados.codigo%TYPE,
                                                 p_FechaInicio envios.fecha_hora%TYPE,
                                                 p_FechaFin envios.fecha_hora%TYPE)
return NUMBER
IS
    v_CantidadTotal NUMBER := 0;
BEGIN
    ComprobarExcepciones(p_NombreProducto, p_CodCampoRefugiado, p_FechaInicio, p_FechaFin);
    SaberCodigoEnviosValidos(p_NombreProducto, p_CodCampoRefugiado, p_FechaInicio, p_FechaFin, v_CantidadTotal);
    if v_CantidadTotal > 0 then
        return(v_CantidadTotal);
    else
        RAISE_APPLICATION_ERROR(-20008, 'No se ha enviado ninguna cantidad de '||p_NombreProducto);
    end if;
END SaberUnidadesTotales;
/

---------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE SaberCodigoEnviosValidos (p_NombreProducto productos.nombre%TYPE,
                                                      p_CodCampoRefugiado campos_refugiados.codigo%TYPE,
                                                      p_FechaInicio envios.fecha_hora%TYPE,
                                                      p_FechaFin envios.fecha_hora%TYPE,
                                                      p_CantidadTotal IN OUT NUMBER)
IS
    CURSOR c_CodigoEnvios IS
    SELECT codigo
    FROM envios
    WHERE to_char(fecha_hora,'YYYY/MM/DD') >= to_char(p_FechaInicio,'YYYY/MM/DD') 
    AND to_char(fecha_hora,'YYYY/MM/DD') <= to_char(p_FechaFin,'YYYY/MM/DD'); 
    v_CantidadTotal paquetes.cantidad%TYPE := 0;
BEGIN
    for registro in c_CodigoEnvios loop
        SaberCantidadTotal(registro.codigo, p_CodCampoRefugiado, p_NombreProducto, v_CantidadTotal);
    end loop;
    p_CantidadTotal := v_CantidadTotal;
END SaberCodigoEnviosValidos;
/

------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE SaberCantidadTotal (p_CodigoEnvio VARCHAR2,
                                                p_CodigoCampo VARCHAR2,
                                                p_NombreProducto VARCHAR2,
                                                p_CantidadTotal IN OUT NUMBER)
IS
    CURSOR c_CantidadProducto IS
    SELECT cantidad
    FROM paquetes
    WHERE codigo_envio = p_CodigoEnvio
    AND codigo_refugio = p_CodigoCampo
    AND nombre_producto = p_NombreProducto;
BEGIN
    for registro in c_CantidadProducto loop
        p_CantidadTotal := registro.cantidad + p_CantidadTotal;
    end loop;
END SaberCantidadTotal;
/


---------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE ComprobarExcepciones (p_NombreProducto productos.nombre%TYPE,
                                                  p_CodCampoRefugiado campos_refugiados.codigo%TYPE,
                                                  p_FechaInicio envios.fecha_hora%TYPE,
                                                  p_FechaFin envios.fecha_hora%TYPE)
IS
BEGIN
    ExceptionProductoInexistente(p_NombreProducto);
    ExceptionCodCampoInexistente(p_CodCampoRefugiado);
    ExceptionEnvioEntreFechas(p_FechaInicio, p_FechaFin);
END ComprobarExcepciones;
/

---------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE ExceptionProductoInexistente (p_NombreProducto productos.nombre%TYPE)
IS
    v_ContadorDeProducto NUMBER;
BEGIN
    SELECT count(nombre) INTO v_ContadorDeProducto
    FROM productos
    WHERE nombre = p_NombreProducto;
    if v_ContadorDeProducto = 0 then
        RAISE_APPLICATION_ERROR(-20001, 'ERROR, Nombre de producto no encontrado');
    end if;
END ExceptionProductoInexistente;
/

---------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE ExceptionCodCampoInexistente (p_CodigoCampo campos_refugiados.codigo%TYPE)
IS
    v_ContadorCodCampo NUMBER;
BEGIN
    SELECT count(codigo) INTO v_ContadorCodCampo
    FROM campos_refugiados
    WHERE codigo = p_CodigoCampo;
    if v_ContadorCodCampo = 0 then
        RAISE_APPLICATION_ERROR(-20002, 'ERROR, Codigo de campo de refugido no encontrado');
    end if;
END ExceptionCodCampoInexistente;
/

---------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE ExceptionEnvioEntreFechas (p_FechaInicio envios.fecha_hora%TYPE,
                                                       p_FechaFin envios.fecha_hora%TYPE)
IS
    v_ContadorEnvios NUMBER;
BEGIN
    SELECT count(codigo) INTO v_ContadorEnvios
    FROM envios
    WHERE to_char(fecha_hora,'YYYY/MM/DD') >= to_char(p_FechaInicio,'YYYY/MM/DD')
    AND to_char(fecha_hora,'YYYY/MM/DD') <= to_char(p_FechaFin,'YYYY/MM/DD');
    if v_ContadorEnvios = 0 then
        RAISE_APPLICATION_ERROR(-20003, 'No hay envios entre las dos fechas dadas');
    end if;
END ExceptionEnvioEntreFechas;
/
```

### Tarea 2. 

* Realiza un procedimiento que genere informes sobre los conflictos gestionando las excepciones que consideres oportunas.

``` sql

-- Informe Tipo 1: El segundo parámetro será una causa de conflicto. Se mostrarán 
-- todos los conflictos que se deben a dicha causa, con la siguiente información:

Causa: xxxxxxxxxxxxxx
    Conflicto: xxxxxxxxx1       Zona: xxxxxxx1
        País1                   NumHeridos1         NumMuertos1
        …
        PaísN                   NumHeridosN         NumMuertosN
    
    Total Heridos Conflicto: nnnnnnnnn
    
    Total Muertos Conflicto: nnnnnnnnn
    
    Conflicto: xxxxxxxxxx2      Zona: xxxxxxx2
    ….
Total Muertos Causa xxxxxxx: n,nnn,nnn

SET SERVEROUTPUT ON
EXEC InformesDeConflictos(1,'Racial')

------------------------------------------------------------------------------------

-- Informe Tipo 2: El segundo parámetro será un grupo armado. Se mostrarán todas las 
-- intervenciones del citado grupo armado, con la siguiente información ordenada por fecha 
-- de incorporación al conflicto:

Grupo Armado: xxxxxxxxxxxxxx
    Conflicto1      FechaIncorporación      FechaRetirada
        País1   NumHeridos1     NumMuertos1
        …
        PaísN   NumHeridosN     NumMuertosN

    Conflicto2 FechaIncorporación FechaRetirada
    ...

SET SERVEROUTPUT ON
EXEC InformesDeConflictos(2,'Legion Al-Rahman')

------------------------------------------------------------------------------------

-- Informe Tipo 3: El segundo parámetro será un nombre de país. Se mostrarán todos los 
-- conflictos que han afectado a dicho país, incluyendo la siguiente información:

País: xxxxxxxxxxx
    Conflicto1             Causa       NumHeridos      NumMuertos
    …
    ConflictoN            Causa       NumHeridos      NumMuertos
    ..
Total Heridos País xxxxxxxx: n,nnn,nnn

Total Muertos País xxxxxxxx: n,nnn,nnn

SET SERVEROUTPUT ON
EXEC InformesDeConflictos(3,'Somalia')

------------------------------------------------------------------------------------
------------------------------------------------------------------------------------

-- CODIGO COMÚN DE LOS TRES INFORMES

CREATE OR REPLACE PROCEDURE InformesDeConflictos (p_NumInforme NUMBER,
                                                  p_ParametroInforme VARCHAR2)
IS
BEGIN 
    case 
        when p_NumInforme = 1 then
            ExcepcionesInforme1(p_ParametroInforme);
            InformeNum1(p_ParametroInforme);
        when p_NumInforme = 2 then
            ExcepcionesInforme2(p_ParametroInforme);
            InformeNum2(p_ParametroInforme);
        when p_NumInforme = 3 then
            ExcepcionesInforme3(p_ParametroInforme);
            InformeNum3(p_ParametroInforme);
        else
            RAISE_APPLICATION_ERROR(-20001, 'ERROR, Numero de informe no valido');
    end case;
END InformesDeConflictos;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE ExcepcionesInforme1 (p_ParametroInforme VARCHAR2)
IS
    v_ContadorCausaConflicto NUMBER;
BEGIN
    SELECT count(causa) INTO v_ContadorCausaConflicto
    FROM conflictos
    WHERE causa = p_ParametroInforme;
    if v_ContadorCausaConflicto = 0 then
        RAISE_APPLICATION_ERROR(-20002, 'ERROR, Parametro del informe 1 no valido o no existente');
    else
        MostrarCabeceras(p_ParametroInforme,1);
    end if;
END ExcepcionesInforme1;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE ExcepcionesInforme2 (p_ParametroInforme VARCHAR2)
IS
    v_ContadorNombreArmado NUMBER;
BEGIN
    SELECT count(nombre) INTO v_ContadorNombreArmado
    FROM grupos_armados
    WHERE nombre = p_ParametroInforme;
    if v_ContadorNombreArmado = 0 then
        RAISE_APPLICATION_ERROR(-20003, 'ERROR, Parametro del informe 2 no valido o no existente');
    else
        MostrarCabeceras(p_ParametroInforme,2);
    end if;
END ExcepcionesInforme2;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE ExcepcionesInforme3 (p_ParametroInforme VARCHAR2)
IS
    v_ContadorNombrePais NUMBER;
BEGIN
    SELECT count(nombre) INTO v_ContadorNombrePais
    FROM paises
    WHERE nombre = p_ParametroInforme;
    if v_ContadorNombrePais = 0 then
        RAISE_APPLICATION_ERROR(-20004, 'ERROR, Parametro del informe 3 no valido o no existente');
    else
        MostrarCabeceras(p_ParametroInforme,3);
    end if;
END ExcepcionesInforme3;
/


CREATE OR REPLACE PROCEDURE MostrarCabeceras (p_ParametroInforme VARCHAR2,
                                              p_NumInforme NUMBER)
IS
BEGIN
    dbms_output.put_line(LPAD(' ',50,'<>'));
    dbms_output.put_line(CHR(13));
    case 
        WHEN p_NumInforme = 1 THEN
            dbms_output.put_line('Causa: '||p_ParametroInforme);
        WHEN p_NumInforme = 2 THEN
            dbms_output.put_line('Grupo Armado: '||p_ParametroInforme);
        WHEN p_NumInforme = 3 THEN
            dbms_output.put_line('Pais: '||p_ParametroInforme);
    end case;
    dbms_output.put_line(chr(13));
    dbms_output.put_line(LPAD(' ',50,'<>'));
    dbms_output.put_line(CHR(13));
END;
/

------------------------------------------------------------------------------------
------------------------------------------------------------------------------------
------------------------------------------------------------------------------------

-- CÓDIGO DEL INFORME 1

CREATE OR REPLACE PROCEDURE InformeNum1 (p_CausaConflicto conflictos.causa%TYPE)
IS
    v_CantidadTotalCausa NUMBER := 0;
BEGIN
    SaberDatosConflicto(p_CausaConflicto, v_CantidadTotalCausa);
    MostrarMuertosCausa(v_CantidadTotalCausa, p_CausaConflicto);
END InformeNum1;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE SaberDatosConflicto (p_CausaConflicto conflictos.causa%TYPE,
                                                 p_CantidadTotalMuertos IN OUT NUMBER)
IS
    CURSOR c_NomCodConflicto IS
    SELECT con.nombre, con.codigo, his.codigo_pais
    FROM conflictos con, historial_de_conflictos his
    WHERE con.codigo = his.codigo_conflicto
    AND causa = p_CausaConflicto;
    v_ContHeridosTotales NUMBER := 0;
    v_ContMuertesTotales NUMBER := 0;
    v_CodigoConflictoIndicador VARCHAR2(3) := '0';
BEGIN
    for registro in c_NomCodConflicto loop
        if v_CodigoConflictoIndicador = registro.codigo then
            SaberNumHeridosMuertos(registro.codigo_pais, registro.codigo, v_ContHeridosTotales, v_ContMuertesTotales);
        else
            MostrarHeridosMuertesTotales(v_ContHeridosTotales, v_ContMuertesTotales, p_CantidadTotalMuertos);
            MostrarConflictos(registro.nombre);
            SaberNumHeridosMuertos(registro.codigo_pais, registro.codigo, v_ContHeridosTotales, v_ContMuertesTotales);
            v_CodigoConflictoIndicador := registro.codigo;
        end if;
    end loop;
    MostrarHeridosMuertesTotales(v_ContHeridosTotales, v_ContMuertesTotales, p_CantidadTotalMuertos);
END SaberDatosConflicto;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION FuncionSaberPais (p_CodigoPais paises.codigo%TYPE)
return VARCHAR2
IS
    v_NombrePais paises.nombre%TYPE;
BEGIN
    SELECT nombre INTO v_NombrePais
    FROM paises
    WHERE codigo = p_CodigoPais;
    return v_NombrePais;
END FuncionSaberPais;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE SaberNumHeridosMuertos (p_CodigoPais IN paises.codigo%TYPE,
                                                    p_CodigoConflicto IN conflictos.codigo%TYPE,
                                                    p_ContHeridosTotales IN OUT NUMBER,
                                                    p_ContMuertesTotales IN OUT NUMBER)
IS
    v_PaisN paises.nombre%TYPE;
    v_HeridosPorPais historial_de_conflictos.num_heridos%TYPE;
    v_MuertosPorPais historial_de_conflictos.num_muertos%TYPE;
BEGIN
    v_PaisN := FuncionSaberPais(p_CodigoPais);
    SELECT num_heridos, num_muertos INTO v_HeridosPorPais, v_MuertosPorPais
    FROM historial_de_conflictos
    WHERE codigo_pais = p_CodigoPais
    AND codigo_conflicto = p_CodigoConflicto;
    p_ContHeridosTotales := v_HeridosPorPais + p_ContHeridosTotales;
    p_ContMuertesTotales := v_MuertosPorPais + p_ContMuertesTotales;
    dbms_output.put_line(CHR(9)||'* '||RPAD(v_PaisN,15)||' '||RPAD(v_HeridosPorPais,10)||' '||RPAD(v_MuertosPorPais,10));
END SaberNumHeridosMuertos;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE MostrarHeridosMuertesTotales (p_ContHeridosTotales IN OUT NUMBER,
                                                          p_ContMuertesTotales  IN OUT NUMBER,
                                                          p_CantidadTotalMuertos IN OUT NUMBER)
IS
    v_Heridos NUMBER := p_ContHeridosTotales;
    v_Muertos NUMBER := p_ContMuertesTotales;
BEGIN
    if v_Heridos != 0 OR v_Muertos != 0 then
        dbms_output.put_line(CHR(13));
        dbms_output.put_line('Total Heridos Conflicto: '||RPAD(v_Heridos,10));
        dbms_output.put_line('Total Muertos Conflicto: '||RPAD(v_Muertos,10));
        dbms_output.put_line(CHR(13));
        dbms_output.put_line(CHR(13));
        p_ContHeridosTotales := 0;
        p_ContMuertesTotales := 0;
        p_CantidadTotalMuertos := v_Muertos + p_CantidadTotalMuertos;
    end if;
END MostrarHeridosMuertesTotales;
/    

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE MostrarConflictos(p_NombreConflicto conflictos.nombre%TYPE)
IS
BEGIN
    dbms_output.put_line(chr(13));
    dbms_output.put_line('# Conflicto: '||p_NombreConflicto);
END MostrarConflictos;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE MostrarMuertosCausa (p_CantidadTotalMuertos NUMBER,
                                                 p_CausaConflicto conflictos.causa%TYPE)
IS
BEGIN
    dbms_output.put_line(LPAD(' ',50,'<>'));
    dbms_output.put_line(CHR(13));
    dbms_output.put_line('Total Muertos Causa '||p_CausaConflicto||': '||p_CantidadTotalMuertos);
    dbms_output.put_line(CHR(13));
    dbms_output.put_line(CHR(13));
END MostrarMuertosCausa;
/

------------------------------------------------------------------------------------
------------------------------------------------------------------------------------
------------------------------------------------------------------------------------

-- CÓDIGO DEL INFORME 2

CREATE OR REPLACE PROCEDURE InformeNum2 (p_NombreGrupoArmado grupos_armados.nombre%TYPE)
IS
    v_CodigoGrupoArmado grupos_armados.codigo%TYPE;
BEGIN
    v_CodigoGrupoArmado := SaberCodigoGrupoArmado(p_NombreGrupoArmado);
    MostrarFechas(v_CodigoGrupoArmado);
END InformeNum2;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION SaberCodigoGrupoArmado (p_NombreGrupoArmado grupos_armados.nombre%TYPE)
return VARCHAR2
IS
    v_CodigoGrupoArmado grupos_armados.codigo%TYPE;
BEGIN
    SELECT codigo INTO v_CodigoGrupoArmado
    FROM grupos_armados
    WHERE nombre = p_NombreGrupoArmado;
    return v_CodigoGrupoArmado;
END SaberCodigoGrupoArmado;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE MostrarFechas (p_CodigoGrupoArmado grupos_armados.codigo%TYPE)
IS
    CURSOR c_Fechas IS
    SELECT fecha_incorporacion, fecha_retirada, codigo_conflicto
    FROM his_intervenciones_armadas
    WHERE codigo_gruparmado = p_CodigoGrupoArmado;
    v_NombreConflicto conflictos.nombre%TYPE;
BEGIN
    for registro in c_Fechas loop
        SELECT nombre INTO v_NombreConflicto
        FROM conflictos
        WHERE codigo = registro.codigo_conflicto;
        MostrarConflictoYFechas(v_NombreConflicto, registro.codigo_conflicto, registro.fecha_incorporacion, registro.fecha_retirada);
    end loop;
END MostrarFechas;
/

------------------------------------------------------------------------------------
---Procedimiento, porque puede haber varios grupos armados que hayan entrado en la misma fecha a distintos conflictos
CREATE OR REPLACE PROCEDURE MostrarConflictoYFechas (p_NombreConflicto grupos_armados.nombre%TYPE,
                                                     p_CodigoConflicto grupos_armados.codigo%TYPE,
                                                     p_FechaIncorporacion DATE,
                                                     p_FechaRetirada DATE)
IS
BEGIN
        dbms_output.put_line(CHR(13));
        if p_FechaRetirada IS NOT NULL then
            dbms_output.put_line(RPAD('# '||p_NombreConflicto,35)||'     '||RPAD(p_FechaIncorporacion,8)||' hasta '||RPAD(p_FechaRetirada,8));
            dbms_output.put_line(CHR(9));
        else
            dbms_output.put_line(RPAD('# '||p_NombreConflicto,35)||'     '||RPAD(p_FechaIncorporacion,8)||' y aun en guerra');
            dbms_output.put_line(CHR(9));
        end if;
        SaberNumHeridosMuertosII(p_CodigoConflicto);
END MostrarConflictoYFechas;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE SaberNumHeridosMuertosII (p_CodigoConflicto IN conflictos.codigo%TYPE)
IS
    CURSOR c_PaisesPorConflictos IS
    SELECT codigo_pais
    FROM historial_de_conflictos
    WHERE codigo_conflicto = p_CodigoConflicto;
    v_PaisN paises.nombre%TYPE;
    v_HeridosPorPais historial_de_conflictos.num_heridos%TYPE;
    v_MuertosPorPais historial_de_conflictos.num_muertos%TYPE;
BEGIN
    for registro in c_PaisesPorConflictos loop
        v_PaisN := FuncionSaberPais(registro.codigo_pais);
        SELECT num_heridos, num_muertos INTO v_HeridosPorPais, v_MuertosPorPais
        FROM historial_de_conflictos
        WHERE codigo_pais = registro.codigo_pais
        AND codigo_conflicto = p_CodigoConflicto;
        dbms_output.put_line(CHR(9)||'* '||RPAD(v_PaisN,15)||' '||RPAD(v_HeridosPorPais,10)||' '||RPAD(v_MuertosPorPais,10));
    end loop;
    dbms_output.put_line(CHR(13));
    dbms_output.put_line(CHR(13));
END SaberNumHeridosMuertosII;
/

------------------------------------------------------------------------------------
------------------------------------------------------------------------------------
------------------------------------------------------------------------------------

-- CÓDIGO DEL INFORME 3

CREATE OR REPLACE PROCEDURE InformeNum3 (p_NombrePais paises.nombre%TYPE)
IS
    v_CodigoPais paises.codigo%TYPE;
BEGIN
    v_CodigoPais := FuncionSaberCodigoPais(p_NombrePais);
    MostrarDatosConflicto(v_CodigoPais);
END InformeNum3;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION FuncionSaberCodigoPais (p_NombrePais paises.nombre%TYPE)
return VARCHAR2
IS
    v_CodigoPais paises.codigo%TYPE;
BEGIN
    SELECT codigo INTO v_CodigoPais
    FROM paises
    WHERE nombre = p_NombrePais;
    return v_CodigoPais;
END FuncionSaberCodigoPais;
/
------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE MostrarDatosConflicto (p_CodigoPais paises.codigo%TYPE)
IS
    CURSOR c_DatosConflictos IS
    SELECT codigo_conflicto, num_heridos, num_muertos
    FROM historial_de_conflictos
    WHERE codigo_pais = p_CodigoPais;
    v_CausaConflicto conflictos.causa%TYPE;
    v_NombreConflicto conflictos.nombre%TYPE;
    v_TotalHeridos NUMBER := 0;
    v_TotalMuertos NUMBER := 0;
BEGIN
    dbms_output.put_line(CHR(13));
    for registro in c_DatosConflictos loop
        v_CausaConflicto := FuncionSaberCausaConflicto(registro.codigo_conflicto);
        v_NombreConflicto := FuncionSaberNombreConflicto(registro.codigo_conflicto);
        dbms_output.put_line(CHR(9)||'* '||RPAD(v_NombreConflicto,35)||' '||RPAD(v_CausaConflicto,15)||' '||RPAD(registro.num_heridos,10)||' '||RPAD(registro.num_muertos,5));
        SaberTotalHeridosMuertos(registro.num_heridos, registro.num_muertos, v_TotalHeridos, v_TotalMuertos);
    end loop;
    dbms_output.put_line(CHR(13));
    dbms_output.put_line('Heridos Totales Pais: '||v_TotalHeridos);
    dbms_output.put_line('Muertos Totales Pais: '||v_TotalMuertos);
END MostrarDatosConflicto;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION FuncionSaberCausaConflicto (p_CodigoConflicto conflictos.codigo%TYPE)
return VARCHAR2
IS
    v_CausaConflicto conflictos.causa%TYPE;
BEGIN
    SELECT causa INTO v_CausaConflicto
    FROM conflictos
    WHERE codigo = p_CodigoConflicto;
    return v_CausaConflicto;
END FuncionSaberCausaConflicto;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION FuncionSaberNombreConflicto (p_CodigoConflicto conflictos.codigo%TYPE)
return VARCHAR2
IS
    v_NombreConflicto conflictos.nombre%TYPE;
BEGIN
    SELECT nombre INTO v_NombreConflicto
    FROM conflictos
    WHERE codigo = p_CodigoConflicto;
    return v_NombreConflicto;
END FuncionSaberNombreConflicto;
/

------------------------------------------------------------------------------------

CREATE OR REPLACE PROCEDURE SaberTotalHeridosMuertos (p_HeridosConflicto IN NUMBER,
                                                      p_MuertesConflicto   NUMBER,
                                                      p_TotalHeridos IN O  NUMBER,
                                                      p_TotalMuertos IN O  NUMBER)
IS
BEGIN
    p_TotalHeridos := p_HeridosConflicto + p_TotalHeridos;
    p_TotalMuertos := p_MuertesConflicto + p_TotalMuertos;
END SaberTotalHeridosMuertos;
/
```

### Tarea 3. 

* Realizar un trigger que garantice que una organización mediadora con menos de diez personas desplegadas en un conflicto no pueda ofrecer ayuda de tipo Ayuda Humanitaria.

``` sql
CREATE OR REPLACE TRIGGER ORGNoMenosDiez
BEFORE
INSERT OR UPDATE OF tipo, numpersonas_conflicto
ON organizaciones
for each row
DECLARE
BEGIN
    ComprobarDatos(:old.numpersonas_conflicto, :new.numpersonas_conflicto, :old.tipo, :new.tipo);
END ORGNoMenosDiez;
/

CREATE OR REPLACE PROCEDURE ComprobarDatos(p_NumPersonasAntes NUMBER,
                                           p_NumPersonasDespues NUMBER,
                                           p_TipoAyudaAntes VARCHAR2,
                                           p_TipoAyudaDespues VARCHAR2)
IS
BEGIN
    if p_NumPersonasAntes < 10 AND p_TipoAyudaDespues = 'Accion Humanitaria' then
        RAISE_APPLICATION_ERROR(-20001, 'ERROR, Tipo de ayuda no valido');
    elsif p_NumPersonasDespues < 10 AND p_TipoAyudaAntes = 'Accion Humanitaria' then
        RAISE_APPLICATION_ERROR(-20002, 'ERROR, Numero de personas no valido');
    elsif p_NumPersonasDespues < 10 AND p_TipoAyudaDespues = 'Accion Humanitaria' then
        RAISE_APPLICATION_ERROR(-20003, 'ERROR, Numero de personas y tipo no valido');
    end if;
END ComprobarDatos;  
/
```

### Tarea 4. 

* Realizar los módulos de programación necesarios para garantizar que los diferentes periodos de intervención de un grupo armado en un conflicto no se solapan entre ellos.

``` sql
create OR replace package conflictosarmados
IS
	TYPE tRegistro is RECORD
		(
		codigogrupo grupos_armados.codigo%TYPE,
		codigoconflicto conflictos.codigo%TYPE,
		fechaincorporacion his_intervenciones_armadas.fecha_incorporacion%TYPE,
		fecharetirada his_intervenciones_armadas.fecha_retirada%TYPE
		);
	TYPE tTabla IS TABLE OF tRegistro INDEX BY BINARY_INTEGER;
    v_tabla tTabla;
END;
/

-----------------------------------------------------------------------------

CREATE OR REPLACE TRIGGER rellenarpkg
before INSERT OR UPDATE OR DELETE ON his_intervenciones_armadas
declare
	CURSOR c_conflictos
	is
	SELECT codigo_gruparmado, codigo_conflicto, fecha_incorporacion, fecha_retirada
	FROM his_intervenciones_armadas;
	cont NUMBER:= 0;
BEGIN
	for i IN c_conflictos loop
		conflictosarmados.v_tabla(cont).codigogrupo := i.codigo_gruparmado;
		conflictosarmados.v_tabla(cont).codigoconflicto := i.codigo_conflicto;
		conflictosarmados.v_tabla(cont).fechaincorporacion := i.fecha_incorporacion;
		conflictosarmados.v_tabla(cont).fecharetirada := i.fecha_retirada;
		cont:=cont+1;
	END loop;
END;
/

-----------------------------------------------------------------------------

CREATE OR REPLACE TRIGGER comprobarconflicto
before INSERT OR UPDATE ON his_intervenciones_armadas
for each row
declare
	indicador NUMBER:=0;
BEGIN
	for i IN conflictosarmados.v_tabla.FIRST .. conflictosarmados.v_tabla.LAST loop
		if (conflictosarmados.v_tabla(i).codigogrupo=:new.codigo_gruparmado OR conflictosarmados.v_tabla(i).codigogrupo!=:new.codigo_gruparmado)
		AND conflictosarmados.v_tabla(i).codigoconflicto=:new.codigo_conflicto
		AND (:new.fecha_incorporacion between conflictosarmados.v_tabla(i).fechaincorporacion AND conflictosarmados.v_tabla(i).fecharetirada) THEN
			raise_application_error(-20001,'Dos grupos armados no pueden entrar en un mismo conflicto mientras ya esta uno');
		ELSE
			indicador := conflictosarmados.v_tabla.LAST;
			conflictosarmados.v_tabla(indicador+1).codigogrupo:= :new.codigo_gruparmado;
			conflictosarmados.v_tabla(indicador+1).codigoconflicto := :new.codigo_conflicto;
			conflictosarmados.v_tabla(indicador+1).fechaincorporacion := :new.fecha_incorporacion;
			conflictosarmados.v_tabla(indicador+1).fecharetirada := :new.fecha_retirada;
		END if;
	END loop;
END;
/
```

### Tarea 5. 

* Realizar los módulos de programación necesarios para garantizar que un grupo armado no ha estado en mas de 5 conflictos en el mismo año.

``` sql
create OR replace package conflictosgrupo
IS
	TYPE tRegistro is RECORD
		(
		codigogrupo grupos_armados.codigo%TYPE,
		codigoconflicto conflictos.codigo%TYPE,
		fechaincorporacion his_intervenciones_armadas.fecha_incorporacion%TYPE
		);
	TYPE tTabla IS TABLE OF tRegistro INDEX BY BINARY_INTEGER;
    v_tabla tTabla;
END;
/

-----------------------------------------------------------------------------

CREATE OR REPLACE TRIGGER rellenarconflictosgrupo
before INSERT OR UPDATE ON his_intervenciones_armadas
declare
	CURSOR c_grupos
	is
	SELECT codigo_gruparmado, codigo_conflicto, fecha_incorporacion
	FROM his_intervenciones_armadas;
	cont NUMBER:= 0;
BEGIN
	for i IN c_grupos loop
		conflictosgrupo.v_tabla(cont).codigogrupo:=i.codigo_gruparmado;
		conflictosgrupo.v_tabla(cont).codigoconflicto:=i.codigo_conflicto;
		conflictosgrupo.v_tabla(cont).fechaincorporacion:=i.fecha_incorporacion;
		cont:=cont+1;
	END loop;
END;
/

-----------------------------------------------------------------------------

CREATE OR REPLACE FUNCTION comprobarnumeroconflictos(p_codigogrupo grupos_armados.codigo%TYPE,p_codigoconflicto conflictos.codigo%TYPE, p_fecha his_intervenciones_armadas.fecha_incorporacion%TYPE)
RETURN NUMBER
AS
	v_Cont NUMBER:=0;
BEGIN
	for i IN conflictosgrupo.v_tabla.FIRST .. conflictosgrupo.v_tabla.LAST loop
		if conflictosgrupo.v_tabla(i).codigogrupo=p_codigogrupo
		AND (conflictosgrupo.v_tabla(i).codigoconflicto=p_codigoconflicto OR conflictosgrupo.v_tabla(i).codigoconflicto!=p_codigoconflicto)
		AND to_char(conflictosgrupo.v_tabla(i).fechaincorporacion,'YYYY') = to_char(p_fecha,'YYYY') THEN
			v_Cont:=v_Cont + 1;
		END if;
	END loop;
	return v_Cont;
END;
/

-----------------------------------------------------------------------------

create or replace procedure insertarfila(p_codigogrupo grupos_armados.codigo%TYPE,
                                         p_codigoconflicto conflictos.codigo%TYPE,
										 p_fechaincorporacion his_intervenciones_armadas.fecha_incorporacion%TYPE)
is
begin
	conflictosgrupo.v_tabla(conflictosgrupo.v_tabla.LAST+1).codigogrupo:=p_codigogrupo;
	conflictosgrupo.v_tabla(conflictosgrupo.v_tabla.LAST+1).codigoconflicto:=p_codigoconflicto;
    conflictosgrupo.v_tabla(conflictosgrupo.v_tabla.LAST+1).fechaincorporacion:=p_fechaincorporacion;
end;
/

-----------------------------------------------------------------------------

create or replace procedure borrarfila(p_codigogrupo grupos_armados.codigo%TYPE,
                                       p_codigoconflicto conflictos.codigo%TYPE,
									   p_fechaincorporacion his_intervenciones_armadas.fecha_incorporacion%TYPE)
is
begin
	
	for i in conflictosgrupo.v_tabla.FIRST .. conflictosgrupo.v_tabla.LAST loop
		if p_codigogrupo=conflictosgrupo.v_tabla(i).codigogrupo
        AND p_codigoconflicto=conflictosgrupo.v_tabla(i).codigoconflicto 
        AND p_fechaincorporacion=conflictosgrupo.v_tabla(i).fechaincorporacion then
			conflictosgrupo.v_tabla.delete(i);
		end if;
	end loop;
end;
/

-----------------------------------------------------------------------------

create or replace procedure numeroconflictos(p_codigogrupo grupos_armados.codigo%TYPE,
                                             p_codigoconflicto conflictos.codigo%TYPE,
                                             p_fechaincorporacion his_intervenciones_armadas.fecha_incorporacion%TYPE)
is
	v_cont number := 0;
begin
    v_cont:=comprobarnumeroconflictos(p_codigogrupo,p_codigoconflicto,p_fechaincorporacion);
    if v_cont >= 5 then
        raise_application_error(-20001,'Ese grupo armado ya ha participado en 5 conflictos armados este año');
    end if;
end;
/

-----------------------------------------------------------------------------

CREATE OR REPLACE TRIGGER comprobarconflictos
before insert or update or delete on his_intervenciones_armadas
for each row 
declare
BEGIN
	case
        when inserting then
            numeroconflictos(:new.codigo_gruparmado, :new.codigo_conflicto, :new.fecha_incorporacion);
            insertarfila(:new.codigo_gruparmado, :new.codigo_conflicto, :new.fecha_incorporacion);

        when updating then
            numeroconflictos(:new.codigo_gruparmado, :new.codigo_conflicto, :new.fecha_incorporacion);
            borrarfila(:old.codigo_gruparmado, :old.codigo_conflicto, :old.fecha_incorporacion);
            insertarfila(:new.codigo_gruparmado, :new.codigo_conflicto, :new.fecha_incorporacion);

        when deleting then
            borrarfila(:old.codigo_gruparmado, :old.codigo_conflicto, :old.fecha_incorporacion);
    end case;
END;
/
```