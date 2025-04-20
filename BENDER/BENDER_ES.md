![[bender1.png]]
## Introducción

La maquina Bender de EchoCTF oculta dos banderas en su pagina web. A través de la exploración del archivo [[robots.txt]] encontramos las rutas que nos que nos dirigen a las banderas 
IP: 10.0.41.1
## Reconocimiento y Escaneo 

Comenzamos el reconocimiento de la máquina **Bender** con un escaneo de puertos utilizando **Nmap**. El comando utilizado fue el siguiente:

	sheepsstealer@drogon:~/echoctf/bender$ nmap -sCS -vv -p 1337 10.0.41.1

RESPUESTA:
	PORT     STATE    SERVICE REASON  
	1337/tcp filtered waste   no-response

El puerto 1337 se encuentra filtrado, lo que indica que el servidor no estaba respondiendo a un escaneo común, al intentar una conexión HTTP a la ip  "10.0.41.1" con el puerto "1337" nos da la siguiente pagina web:

![[bender2.png]]

#### Fuzzing con dirsearch 
Después de eso realizamos fuzzing con **dirsearch** para identificar rutas y directorios adicionales en la pagina este archivo nos indicaba que el directorio /nogooglebot/ estaba desahabilitado para los motores de búsqueda, lo que sugiere que puede haber contenido oculto o sensible en ese directorio.

![[bender3.png]]

podemos ver como el único directorio con código 200 (OK) es el /[[Robots.txt]], al revisar el contenido de [[Robots.txt]] encontramos lo siguiente

	 well done ETSCTF_ae1764e05e046f3770ac7b396c4ab0a2
	User-agent: Googlebot
	Disallow: /nogooglebot/

Tenemos la primer bandera  y ademas este archivo nos indicaba que el directorio **`/nogooglebot/`** estaba desahabilitado para los motores de búsqueda, lo que sugiere que puede haber contenido oculto o sensible en ese directorio.
#### Acceso a **/nogooglebot/** 
Al ingresar al directorio __/nogooglebot/__ encontramos otra seccion de la pagina web que nos mostraba un mensaje **"Look Closer"** 

![[bender4.png]]

al inspeccionar el codigo fuente de la pagina justo en la seccion del mensaje **"Look Closer"** encontramos la segunda bandera 

	<!-- ETSCTF_e79422930da899e5634d7f9da9c601a2 -->

## Identificación de vulnerabilidades

El archivo [[Robots.txt]] no es un mecanismo de seguridad, pero en este caso, **reveló información crítica** sobre un directorio que no debía ser indexado por los motores de búsqueda. Aunque no es una vulnerabilidad grave por sí misma, es una mala práctica dejar información sensible accesible de esta forma.

## Medidas de mitigacion 

**Validación de directorios**: Asegurarse de que los directorios sensibles no sean accesibles, ya sea mediante configuraciones de seguridad adicionales o mediante el uso de herramientas como **.htaccess** en servidores web.

-Writeup by JunoGG-