## Que son los HTTP HEADERS? 

Son lineas de texto que se envian entre el cliente (Por ejemplo el navegador WEB o una herramienta como CURL) y el servidor web cuando hacen una solicitud o responden a ella. 

![header1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Ciber%20Security/imagenes/cabecera.png)

## Header de solicitud (cliente -> servidor)
Se usa para indicar que quiere el cliente y como puede recibir la respuesta 

- `Host`: el dominio solicitado.
    
- `User-Agent`: tipo de navegador o cliente.
    
- `Accept`: qué tipo de contenido acepta (ej. `text/html`, `application/json`).
    
- `Cookie`: cookies enviadas al servidor.


## Header de respuesta (Cliente <- Servidor)
El servidor las usa para informar sober la respuesta 

- `Content-Type`: tipo de contenido devuelto (HTML, JSON, imagen, etc.).
    
- `Server`: tecnología del servidor (`nginx`, `Apache`, etc.).
    
- `Set-Cookie`: cookies que el cliente debe guardar.
    
- `X-Powered-By`: tecnologías del backend (como `PHP/7.4.23`).
    
- `X-XSS-Protection`, `Strict-Transport-Security`, `Content-Security-Policy`: cabeceras de seguridad.

## Importancia de los Headers 
- Pueden revelar informacion importante o sensible sobre el servidor
- Permiten ataques si estan mal configuradas
-  Te permiten personalizar o manipular la interaccion HTTP 

## Ejemplo de cabecera 

`PETICION`

```http 
GET /capital/francia HTTP/1.1
Host: chat.openai.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64) Chrome/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language: es-MX,es;q=0.9
Connection: keep-alive

```

Esto le dice al servidor "Hola soy un navegador Chrome, quiero que me des info sobre `/Capital/francia` y prefiero contenido en espanol"

`RESPUESTA`
``` HTTP
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 46
Server: gpt-4.openai.com
X-Powered-By: GPT-4
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
Date: Sat, 17 May 2025 19:28:00 GMT
```

Lo cual daria el siguiente mensaje en el cuerpo del mensaje 

``` HTML
<p>La capital de Francia es <strong>París</strong>.</p>
```

## Header HTTP mal configurado

``` HTTP
HTTP/1.1 200 OK
Content-Type: text/html
Server: Apache/2.4.41 (Ubuntu)
X-Powered-By: PHP/5.6.40
X-Backend-Server: internal-dev01.local
X-Debug-Mode: true
X-Admin-Email: admin@empresa.com
```

### ⚠️ **¿Qué problemas hay aquí?**

1. **`Server: Apache/2.4.41 (Ubuntu)`**  
    Revela la versión exacta del servidor web y el sistema operativo. Un atacante puede buscar exploits conocidos para esa versión.
    
2. **`X-Powered-By: PHP/5.6.40`**  
    Indica que el servidor usa una versión vieja y vulnerable de PHP (5.6). Este dato es oro para un atacante.
    
3. **`X-Backend-Server: internal-dev01.local`**  
    Filtra la dirección o nombre del servidor interno, útil para ataques dirigidos o escaneos internos (si logra entrar a la red).
    
4. **`X-Debug-Mode: true`**  
    Indica que el modo debug está activo, lo cual puede permitir errores detallados o respuestas extendidas.
    
5. **`X-Admin-Email: admin@empresa.com`**  
    Filtra el correo del administrador del sistema. Útil para ataques de phishing, ingeniería social o enumeración de usuarios


## Header corregida 
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: no-referrer
Permissions-Policy: geolocation=()

```

Aquí no se expone ninguna tecnología interna, y además se agregan **cabeceras de seguridad** recomendadas por OWASP para proteger contra XSS, clickjacking, filtrado de tipos de contenido y más.

