# Que es el IP SPOOFING?
El **IP SPOOFING** o "suplantacion de IP" es la creacion de paquetes de internet o solicitudes con una direccion IP de origen modificada para poder ocultar la verdadera identidad del emisor, suplantar a otro sistema o ambas cosas. Asi el servidor cree que la solicitud viene de un lado del que no o que viene de un cliente confiable al cual le puede entregar informacion. 
Es una tecnica que ocupan los agentes maliciosos para generar diversos ataques entre ellos los famosos DDoS 

El envio de paquetes y solicitudes maquina-maquina y maquina-servidor es la principal forma de comunicacion entre los dispositivos de red hoy en dia. estos paquetes llevan un encabezado que le indica al receptor quien envia el paquete, en estos suele venir la informacion como la IP la cual puede ser modificada con fines maliciosos para suplantar ser otro emisor. 

![ips](https://www.zenarmor.com/docs/assets/images/ip-spoofing-181ecdcb3bd8364306cc5c2534764d10.png)

___
# Que tipos de ataques de Spoofing hay?

¿Qué tipos de ataques se pueden realizar mediante suplantación de IP? Por mencionar algunos, existen cuatro tipos de ataques de suplantación de IP:

* ***Non-Blind Spoofing**: El atacante envía numerosos paquetes al objetivo mediante suplantación ciega. Estos ciberatacantes suelen operar fuera de los límites de la red local y desconocen en gran medida cómo se transmiten los datos en ella. Por lo tanto, antes de lanzar un ataque, suelen centrarse en averiguar cómo se leen los paquetes en orden.

* ***Blind Spoofing:** El atacante y su objetivo se encuentran en la misma subred mediante suplantación no ciega. Esto les permite rastrear la red para averiguar la secuencia de los paquetes. Una vez que el hacker accede a la secuencia, puede eludir la autenticación haciéndose pasar por otra máquina conocida y legal.

* ***Man-in-the-Middle Attack** (MITM): Un ataque de intermediario (MITM) ocurre cuando un atacante se interpone en la comunicación entre un usuario y una aplicación, ya sea para espiar o imitar a una de las partes, simulando un flujo de información legítimo.

* ***Denial of Service Atack** (DDoS): En este tipo de ataques, el atacante envía paquetes y mensajes a su objetivo desde diversos dispositivos. Por lo tanto, identificar el origen de la suplantación de direcciones IP en ataques DoS se vuelve extremadamente difícil. Al no poder rastrear el origen del ataque, no pueden bloquearlo.

___

# Como se detecta el IP SPOOFING 

La suplantación de IP es difícil de detectar para los usuarios finales. Estos ataques ocurren en la capa de red, que es la capa 3 del modelo de comunicaciones de Interconexión de Sistemas Abiertos. De esta manera, no habrá rastros externos de intromisión. Externamente, las solicitudes de conexión falsas parecen ser solicitudes de conexión válidas.

Sin embargo, las empresas pueden utilizar tecnologías de monitorización de red para analizar el tráfico en los puntos finales de la red. El método más común es el filtrado de paquetes.

Los routers y firewalls suelen incorporar tecnologías de filtrado de paquetes. Buscan discrepancias entre la dirección IP del paquete y las direcciones IP deseadas en las listas de control de acceso (ACL). También pueden detectar paquetes falsificados.

Los dos tipos de filtrado de paquetes son el de entrada y el de salida:

El filtrado de entrada examina la cabecera IP de origen de los paquetes entrantes para verificar si coincide con una dirección de origen permitida. Cualquier paquete que no coincida o presente un comportamiento cuestionable se rechaza. Este filtrado crea una lista de control de acceso (ACL) que contiene las direcciones IP de origen permitidas.

Los datos salientes se examinan mediante el filtrado de salida. Este método busca direcciones IP de origen que no estén en la red de la empresa. Los usuarios internos no podrán lanzar un ataque de suplantación de IP con este método.

___
# EJEMPLO

Un buen ejemplo de esto fue el **ataque de denegación de servicio distribuido (DDoS) contra GitHub en 2018**, considerado uno de los mayores de su tipo hasta ese momento. Los atacantes **utilizaron direcciones IP falsificadas** para inundar los servidores de GitHub con tráfico, sobrecargándolos y haciéndolos inaccesibles para los usuarios durante aproximadamente 20 minutos.

![github](https://i0.wp.com/user-images.githubusercontent.com/579876/36830593-2ae2cf98-1cd9-11e8-944d-dde6248ac0e5.png?ssl=1)

[GitHub DDoS atack](https://github.blog/news-insights/company-news/ddos-incident-report/)

