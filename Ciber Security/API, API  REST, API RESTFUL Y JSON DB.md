# Seguridad en APIs RESTful y json-server (Basado en Argonauts)

## Que es una API?
___
El principal concepto con el cual estar familiarizado es el de __API__ (Aplication Programing Interface) es es una "Interfaz", o bien en palabras mas simples es un mecanismo que permite a dos componentes de software comunicarse entre si mediante un conjunto de definiciones y protocolos.

Por ejemplo, si quisieras crear una APP de viaje tipo UBER necesitarias los mapas minimo de toda tu ciudad, crear esos mapas por tu cuenta es costoso y poco practico, por este motivo mejor buscas una empresa que ya haya desarrollado todo ese software que contenga la informacion de todos los mapas y contratas esa informacion para agregarla a tu app, por ejemplo google. Google Maps ya tiene toda la informacion de mapas en el mundo, por lo cual tu puedes agregar esa informacion a tu app con una API. 

![api1](https://imgs.search.brave.com/YOBPmur0n1lMptfzNWSe_R-SEelRHqhaEk5iSYy39ZI/rs:fit:860:0:0:0/g:ce/aHR0cHM6Ly9vdXR2/aW8uY29tL3N0YXRp/Yy9iNjQ5ZWVmZDY4/N2IxZTE3NWY2Y2Iy/MjI5ODkyOTkwMi8x/YjRhNi9jbDJ0MGxj/NGwwMDB3N2I3dDB5/bGkzNTZqLmpwZw)

Como podemos ver en la imagen anterior y basandonos en el ejemplo de la APP la API seria codigo o una interfaz que conectaria la APP con la informacion de los mapas que google ya recopilo anteriormente.

En un ejemplo mas practico, imagina que vas a conducir un coche, entonces la API seria un volante, tu controlas el coche atravez del volante (la API), realmente tu no sabes como funciona el coche por dentro a detalle, y no importa, solo necesitas el volante para manejarlo, entonces la API permite la comunicacion y control entre tu y el coche.

#### Interfaz 
Al hablar de "Interfaz" nos referimos a un 'contrato' de servicio entre dos APPs, este contrato va a definir como se comunican estas apps, mediante protocolos y solicitudes.

## Como funciona una API? 
___
Las API funcionan mediante arquitecturas, esto quiere decir la forma en la que se configuran para hacer determinadas tareas en determinadas situaciones y hay 4 principales.

* API DE SOAP
* API DE RPC
* API DE WEBSOCKET
* API DE REST
En esta ocasion nos vamos a enfocar en las API DE REST pero si te interesa profundizar mas en las API puedes visitar el siguiente enlace: [Que es una API?](https://aws.amazon.com/es/what-is/api/)

## API REST
___
REST significa __Transferencia de estado representacional__. REST es una arquitectura, entonces tiene ya definida ciertas funciones para funcionar, por ejemplo, REST define un conjunto de funciones como GET, POST, PULL, DELETE que los clientes ocupan para intercambiar informacion con el servidor. Los clientes intercambian esta informacion mediante metodos HTTP.

 REST se creó como una guía para administrar la comunicación en una red compleja como Internet. Puede implementarla y modificarla fácilmente, lo que brinda visibilidad y portabilidad entre plataformas a cualquier sistema de API.
Los servicios web que implementan una arquitectura de REST son llamados servicios web RESTful. El término API RESTful suele referirse a las API web RESTful. Sin embargo, los términos API REST y API RESTful se pueden utilizar de forma intercambiable.

**SI TE INTERESA SABER MAS A PROFUNDIDAD SOBRE LAS APIREST TE RECOMIENDO EL SIGUIENTE LINK [AWS QUE ES UNA API REST](https://aws.amazon.com/es/what-is/restful-api/)**

## API RESTFUL 
___
Ya vimos que una API RESTful es un diseño de API que permite interactuar con recursos a través de la web usando métodos y URLs estándar, facilitando la interoperabilidad entre sistemas.

1- **Recursos Identificados**: Cada dato, recurso o producto dentro de la base de datos tiene un identificador que la API tiene que usar para hallarlo y esta a su vez lo obtiene de la URL, lo que conocemos con un _endpoint_ 

2- **Uso de metodos HTTP**: Ocupa metodos HTTP estandar como GET,POST,PUT,DELTE etc.

3- **Sin estado**: Cada peticion del cliente al servidor debe de contener toda la informacion necesaria para que el servidor pueda entender y procesar la solicitud 

![API3](https://imgs.search.brave.com/UPMQt4h6yS8TNiAVD4Aj9aippY8kA5sTZ2oJy0cOY10/rs:fit:860:0:0:0/g:ce/aHR0cHM6Ly91cGxv/YWRzLnNpdGVwb2lu/dC5jb20vd3AtY29u/dGVudC91cGxvYWRz/LzIwMjIvMDgvMTY2/MTc0OTEyNVJFU1Qt/QVBJLVJlcXVlc3Qu/cG5n)

## JSON SERVER
___

