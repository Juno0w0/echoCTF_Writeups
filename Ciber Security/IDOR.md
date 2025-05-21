# Insecure Direct Object References

Estas vulnerabilidades son un tipo de fallo de seguridad en aplicaciones web y API's donde se le permite a un usuario acceder directamente a objetos (como recursos, funciones o archivos) en funcion de la consulta que este realice simplemente modificando un identificador en la URL, peticion o formulario.

Esta vulnerabilidad tiene que ver con ___errores de implementacion de control de acceso___ y se ubica en la lista [TOP 10 de OWASP](https://owasp.org/www-project-top-ten/2017/Top_10) sobre riesgos de seguridad mas comunes en aplicaciones WEB (Broken Access Control)

## Como funciona?
* La aplicacion expone un parametro como __ID__ de Usuario, Documento, Archivo etc. que referencia a un recurso interno. 
* No valida adecuadamente que el usuario tenga permiso para acceder a ese recurso
* Al cambiar ese identificador (por ejemplo, un numero en la URL ), un atacante puede acceder a datos o funcionalidades que no le corresponden 

![idor](https://i0.wp.com/lab.wallarm.com/wp-content/uploads/2024/08/283.2-min.jpg?w=770&ssl=1)
## Nivel de Impacto

Es casi imposible definir cual es su impacto real pues esto varea dependiendo del nivel de sensibilidad de la informacion o nivel del control y funciones que adquiera el atacante, sin embargo su presencia puede desencadenar:

- __Brechas de informacion__ : Al acceder a informacion sensible y tener la posibilidad de modificar esta informacion
- __Modificacion de privilegios__ : Al tener nuevas funcionalidades que puedan comprometer el correcto funcionamiento del servidor

## Como se veria? 

``` c
https://ejemplo.com/perfil?user_id=123
```

Un atacante cambia el `user_id` a otro valor 

``` c
https://ejemplo.com/perfil?user_id=124
```

Si el sistema no valida que el usuario tiene permisos para ver ese contenido, el atacante puede ver la informacion provada de otro usuario.

## Que es un endpoint? 

Ya vimos que el IDOR es una vulnerabilidad donde se modifican los __endpoints__ pero, que es un endpoint? 
Un ednpoint es una URL o direccion especifica en un servidor que busca acceder a un recurso o servicio en particular. Es el punto final de la comunicacion donde un cliente envia solicitudes para obtener datos.

- En una **API RESTful**, cada endpoint corresponde a un recurso o conjunto de operaciones. Por ejemplo:
    
    - `GET /users` → obtiene la lista de usuarios.
        
    - `POST /users` → crea un nuevo usuario.
        
    - `GET /users/{id}` → obtiene un usuario específico.
        
    - `DELETE /users/{id}` → elimina un usuario.
        
- En aplicaciones web tradicionales, el endpoint puede ser simplemente una ruta o página, como:
    
    - `https://miapp.com/login`
        
    - `https://miapp.com/profile/edit`

#### __En resumen:__
Los endpoints son las zonas de la URL donde se reciben datos de entrada y se ejecutan funciones del backend 
La vulnerabilidad IDOR es muy común debido a errores en la implementación de controles de acceso y también debido a la falta de software capacitado para identificarlo.

Por otro lado, y a pesar de su simpleza, puede tener un impacto grave en tu empresa, causando brechas de información o comprometiendo la integridad de los datos almacenados.