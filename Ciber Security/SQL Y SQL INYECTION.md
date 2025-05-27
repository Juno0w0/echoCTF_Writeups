## ¿Qué es SQL?

**SQL (Structured Query Language)** es un lenguaje de programación diseñado para gestionar y manipular bases de datos relacionales.  
Su función principal es permitir a los usuarios:

- **Crear** bases de datos y tablas.
- **Insertar** datos.
- **Actualizar** datos existentes.
- **Eliminar** datos.
- **Consultar** o recuperar información específica.
## ¿Qué es una consulta SQL?

Una **consulta SQL** es una instrucción que se envía al sistema de gestión de bases de datos para que realice una operación sobre los datos almacenados.

Las consultas permiten desde obtener información específica hasta modificar o eliminar registros.

## Consultas basicas en SQL
La solicitud de información a una base de datos se realiza a través de un comando llamado SELECT, el cual permite seleccionar las columnas que se van a mostrar y en el orden en que lo van a hacer. Simplemente es la instrucción que la base de datos interpreta como que vamos a solicitar información.

La sintaxis básica de un comando SQL es la siguiente:

``` SQL
SELECT [ ALL / DISTINC ] [ * ] FROM Nombre_Tabla_Vista WHERE Condiciones ORDER BY ListaColumnas [ ASC / DESC ]
/* ejemplo*/ 
SELECT nombre, edad FROM usuarios WHERE id = 10;
```

- `SELECT nombre, edad`  
    Indica las columnas que queremos obtener, en este caso, `nombre` y `edad`.
- `FROM usuarios`  
    Especifica la tabla de donde se extraen los datos, aquí la tabla se llama `usuarios`.
- `WHERE id = 10`  
    Es una condición para filtrar los datos: solo devuelve filas donde el campo `id` sea igual a 10.

## Tipos comunes de consultas SQL

- **SELECT**: para leer o consultar datos.  
    Ejemplo: `SELECT * FROM productos;`
    
- **INSERT**: para agregar nuevos datos.  
    Ejemplo: `INSERT INTO usuarios (nombre, edad) VALUES ('Ana', 25);`
    
- **UPDATE**: para modificar datos existentes.  
    Ejemplo: `UPDATE usuarios SET edad = 26 WHERE nombre = 'Ana';`
    
- **DELETE**: para eliminar datos.  
    Ejemplo: `DELETE FROM usuarios WHERE edad < 18;`

## ¿Cómo funciona una consulta SQL?

Cuando una aplicación envía una consulta SQL a la base de datos, el motor SQL la interpreta y ejecuta, devolviendo los resultados o modificando los datos según lo indicado.

---

## ¿Qué es SQL Injection?

**SQL Injection** es una vulnerabilidad de seguridad que ocurre cuando una aplicación incluye datos proporcionados por el usuario directamente en una consulta SQL sin validarlos o escaparlos adecuadamente.

Esto permite a un atacante insertar código SQL malicioso dentro de la consulta legítima, alterando su comportamiento.

## ¿Cómo funciona una SQL Injection? (Ejemplo)

Supongamos que tienes este código vulnerable:
``` SQL
`SELECT * FROM usuarios WHERE nombre = 'usuario_input';`
```
Si el usuario introduce como nombre:

``` SQL
' OR '1'='1`
```

La consulta resultante será:
``` SQL
SELECT * FROM usuarios WHERE nombre = '' OR '1'='1';`
```
Esto es siempre verdadero (`'1'='1'`), por lo que la consulta devuelve **todas** las filas, pudiendo el atacante acceder a información que no debería.