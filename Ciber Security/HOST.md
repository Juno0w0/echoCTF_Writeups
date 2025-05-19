****
Host, tambien conocido como hosting, hospedaje o anfitrion, es cualquier computadora o maquina conectada a una red mediante un numero de IP definido y un dominio su funcion es proporcionarle recursos, informacion y servicios a los usuarios.

![](https://imgs.search.brave.com/_2PzKEH93fo5Vspz-M48WRrZbdNFl9B5sAuCUiMVbns/rs:fit:860:0:0:0/g:ce/aHR0cHM6Ly93d3cu/aG9zdGluZ2VyLmNv/bS9teC90dXRvcmlh/bGVzL3dwLWNvbnRl/bnQvdXBsb2Fkcy9z/aXRlcy8zOS8yMDE4/LzA2L0VTLWhvdy1k/b2VzLXdlYi1ob3N0/aW5nLXdvcmstbG9j/YWxzLTEwMjR4NDQ0/LmpwZw)

El servicio de HOSTING mas utlizado es el internet. Sin embargo cuando hablamos de HOST debemos recordar que existen otras formas de red ademas de Internet. 
Por ejemplo un erutador responsable de la red donde varias computadoras se conectan entre si a traves de una IP tambien puede considerarse un HOST

Cuando abres tu navegador e ingresas una URL se establece una conexion con el servidor en el que el sitio esta hospedado, gracias a este proceso el sitio web se convierte en una IP (cada IP es unica en el mundo), para poder cordinar esto existe una organizacion llamada ICANN. Esta entidad esta encargada de asignar los nombres de dominio y direcciones IP para que internet funcione a nivel mundial.

![host2](https://imgs.search.brave.com/-qerMwRLa-IDkzCtWP1paxfQsN9fEXlK4C5yH63Ckuc/rs:fit:860:0:0:0/g:ce/aHR0cHM6Ly9kdWFs/dGhpbmsuZXMvd3At/Y29udGVudC91cGxv/YWRzLzIwMjMvMDMv/JUMyJUJGUXVlLW9j/dXJyZS1jdWFuZG8t/c2UtYnVzY2EtdW4t/ZG9taW5pby5qcGc)

Usualmente el `HOST` presenta los siguientes servicios 

- cuentas de email,
- funciones de seguridad,
- [automatización del marketing](https://rockcontent.com/es/blog/marketing-automation/),
- optimización [SEO](https://rockcontent.com/es/blog/que-es-seo/),
- dominio,
- entre otros.

![host3](https://imgs.search.brave.com/ddBqYaNf9rPbC3Sm5uZ0mYjk5v5B0bTydY8Zm_ACokQ/rs:fit:860:0:0:0/g:ce/aHR0cHM6Ly9kdWFs/dGhpbmsuZXMvd3At/Y29udGVudC91cGxv/YWRzLzIwMjMvMDMv/VGlwb3MtZGUtaG9z/dGluZ3MuanBn)

### Que es un HOST virtual?
Es una tecnica uasda en servidores web (como Apache o Nginx) para servir diferentes contenidos o sitios web dependiendo del nombre de dominio solicitado por el cliente, esto se basa en el valor del `HOST` que el navegador envia.

**EJEMPLO**
Tienes una sola IP (por ejemplo `192.168.1.1`), pero quieres alojar:

- `www.miempresa.com`
    
- `blog.miempresa.com`
    
- `clientes.miempresa.com`
    

Cada uno es un **host virtual**, aunque todos se resuelvan en la misma IP.

#### Como funciona?
Cuando un navegador web hace una solicitud, por ejemplo 

``` HTTP
GET / HTTP/1.1
Host: blog.miempresa.com
```
El servidor revisa su configuracion y determina que sitio debe de servir dependiendo de ese `HOST`, esto sirve para **Alojar múltiples sitios web en un solo servidor** sin necesidad de múltiples IPs.

En el mundo real, muchas aplicaciones usan virtual hosts para alojar varios dominios en un mismo servidor. Cuando un atacante descubre que el servidor responde distinto segun el `HOST`, puede:

- **Acceder a contenido oculto**, como entornos de desarrollo (dev.site.com) 
- **Forzar redirecciones**, por ejemplo en aplicaciones que usan el `HOST`para construir enlaces o correos
