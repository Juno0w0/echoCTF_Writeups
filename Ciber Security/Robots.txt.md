## Que es el archivo robots.txt 

Un archivo robots.txt es un conjunto de instrucciones que los bots, es un archivo NO OBLIGATORIO pero que se incluye en los archivos fuente de la mayoría de los sitios web con la finalidad de gestionar la actividad de los "bots" de los motores de búsqueda o __rastreadores web__ 

Pensemos que un archivo robots.txt es como un cartel de "Código de conducta" colocado en la pared de un gimnasio, un bar o un centro comunitario: el cartel por sí mismo no tiene la capacidad de obligar a cumplir las normas enumeradas, pero los "buenos" clientes las cumplirán, mientras que es probable que los "malos" no las cumplan y sean expulsados

Los rastreadores web "rastrean" las paginas web e indexan el contenido para que aparezca en los resultados de los motores de búsqueda cuando un usuario hace una consulta, el robots.txt tiene instrucciones para indicarle al motor de búsqueda que sitios no indexar y evitar que salgan como sugerencias para el usuario. 

![robotsWA](robots_WA.png)

Aunque un archivo robots.txt proporciona instrucciones para los bots, no puede hacer que se cumplan las instrucciones. Un bot beneficioso, como un rastreador web o un bot de información, intentará visitar primero el archivo robots.txt antes de ver cualquier otra página de un dominio, y cumplirá con las instrucciones. Un bot perjudicial ignorará el archivo robots.txt o lo procesará para encontrar las páginas web que están prohibidas.

Una cosa importante que hay que tener en cuenta es que todos los subdominios necesitan su propio archivo robots.txt. Por ejemplo, mientras que www.cloudflare.com tiene su propio archivo, todos los subdominios de Cloudflare (blog.cloudflare.com, community.cloudflare.com, etc.) necesitan también el suyo.

## ¿Qué protocolos se utilizan en un archivo robots.txt?

En redes, un [protocolo](https://www.cloudflare.com/learning/network-layer/what-is-a-protocol/) es un formato para proporcionar instrucciones o comandos. Los archivos robots.txt utilizan un par de protocolos diferentes. El protocolo principal se llama Protocolo de exclusión de bots. Es una forma de indicar a los bots las páginas web y recursos que deben evitar. Las instrucciones formateadas para este protocolo están incluidas en el archivo robots.txt.

El otro protocolo utilizado para los archivos robots.txt es el protocolo Sitemaps. Este se puede considerar un protocolo de inclusión de bots. Sitemaps muestran a un rastreador web qué páginas puede rastrear. Esto ayuda a garantizar que un bot rastreador no se pierda ninguna página importante.