# Host Header Attacks

## INTRODUCCION

El header `host` es obligatorio a partir de HTTP/1.1. Especifica el nombre de dominio al que el cliente quiere acceder. Por ejemplo, cuando un usuario visita `https://portswigger.net/web-security`, su navegador redactará una solicitud que contiene un encabezado de Host de la siguiente manera:

```http
GET /web-security HTTP/1.1
Host: portswigger.net
```

## ¿Cuál es el propósito del header host HTTP?

El propósito del header del host HTTP es ayudar a **identificar con qué componente de back-end desea comunicarse el cliente**. Si las solicitudes no contenían encabezados de Host, o si el encabezado de Host tenía un formato incorrecto de alguna manera, esto podría generar problemas al enrutar las solicitudes entrantes a la aplicación deseada.

### VIRTUAL HOSTING

Un escenario posible es cuando un único servidor web alberga varios sitios web o aplicaciones. Esto podría ser varios sitios web con un solo propietario, pero también es posible que los sitios web con diferentes propietarios se alojen en una única plataforma compartida.

En cualquier caso, aunque cada uno de estos sitios web distintos tendrá un nombre de dominio diferente, todos comparten una dirección IP común con el servidor. Los sitios web alojados de esta manera en un único servidor se conocen como "hosts virtuales".

### Enrutamiento del tráfico a través de un intermediario

Otro escenario común es cuando los sitios web están alojados en distintos servidores back-end, pero todo el tráfico entre el cliente y los servidores se enruta a través de un sistema intermediario. Esto podría ser un balanceador de carga simple o un servidor proxy inverso de algún tipo. Esta configuración es especialmente frecuente en los casos en que los clientes acceden al sitio web a través de una red de entrega de contenido (CDN).

## ¿Qué es un ataque de header host HTTP?

Los ataques de encabezado de Host HTTP explotan sitios web vulnerables que manejan el valor del encabezado de Host de una manera no segura. Si el servidor confía implícitamente en el encabezado del Host y no lo valida o escapa correctamente, un atacante puede usar esta entrada para inyectar cargas dañinas que manipulan el comportamiento del lado del servidor. Los ataques que implican la inyección de una carga útil directamente en el encabezado del host se conocen a menudo como **"Host header injection Attacks"**

Algunas paginas hacen referencia a este header para concatenarlo a alguna funcionalidad:

```html
<a href="https://_SERVER['HOST']/support">Contact support</a>
```

Como el encabezado del host es, de hecho, controlable por el usuario, esta práctica puede generar una serie de problemas. Si la entrada no se escapa o valida correctamente, el encabezado del Host es un vector potencial para explotar una variedad de otras vulnerabilidades, en particular:

-   Web cache poisoning
-   Business logic flaws in specific functionality
-   Routing-based SSRF
-   Classic server-side vulnerabilities, such as SQL injection

## Cómo probar vulnerabilidades utilizando el header host HTTP

Para probar si un sitio web es vulnerable a un ataque a través del encabezado del host HTTP, necesitará un proxy de interceptación, como Burp Proxy, y herramientas de prueba manuales como Burp Repeater y Burp Intruder.

Si puede modificar el encabezado del host y aun así llegar a la aplicación de destino con su solicitud. Si es así, puede usar este encabezado para sondear la aplicación y observar qué efecto tiene esto en la respuesta.

### Proporcione un encabezado de Host arbitrario

Al buscar vulnerabilidades de inyección de encabezado de Host, el primer paso es probar qué sucede cuando proporciona un nombre de dominio arbitrario y no reconocido a través del encabezado de Host.

### Inyectar encabezados de host duplicados

Un enfoque posible es intentar agregar encabezados de host duplicados. Es cierto que esto a menudo resultará en el bloqueo de su solicitud. Sin embargo, como es poco probable que un navegador envíe una solicitud de este tipo, es posible que ocasionalmente los desarrolladores no hayan previsto este escenario.

Diferentes sistemas y tecnologías manejarán este caso de manera diferente, pero es común que uno de los dos encabezados tenga prioridad sobre el otro, anulando efectivamente su valor. Cuando los sistemas no están de acuerdo sobre qué encabezado es el correcto, esto puede generar discrepancias que puede aprovechar. Considere la siguiente solicitud:

```http
GET /example HTTP/1.1
Host: vulnerable-website.com
Host: bad-stuff-here
```

Digamos que el front-end da prioridad a la primera instancia del encabezado, pero el back-end prefiere la instancia final. Dado este escenario, podría usar el primer encabezado para asegurarse de que su solicitud se enruta al destino deseado y usar el segundo encabezado para pasar su carga útil al código del lado del servidor.

### Proporcione una URL absoluta

Aunque la línea de solicitud generalmente especifica una ruta relativa en el dominio solicitado, muchos servidores también están configurados para comprender las solicitudes de URL absolutas.

La ambigüedad causada por proporcionar una URL absoluta y un encabezado de host también puede generar discrepancias entre diferentes sistemas.

```http
GET https://vulnerable-website.com/ HTTP/1.1
Host: bad-stuff-here
```

Tenga en cuenta que es posible que también deba experimentar con diferentes protocolos. Los servidores a veces se comportarán de manera diferente dependiendo de si la línea de solicitud contiene una URL HTTP o HTTPS.

### Agregar ajuste de línea

También puede descubrir comportamientos extraños al sangrar los encabezados HTTP con un carácter de espacio. Algunos servidores interpretarán el encabezado sangrado como una línea envuelta y, por lo tanto, lo tratarán como parte del valor del encabezado anterior. Otros servidores ignorarán el encabezado con sangría por completo.

```http
GET /example HTTP/1.1
    Host: bad-stuff-here
Host: vulnerable-website.com
```

## Inyectar encabezados de anulación de host (Override Headers)

Incluso si no puede anular el encabezado Host mediante una solicitud ambigua, existen otras posibilidades para anular su valor y dejarlo intacto. Esto incluye inyectar su carga útil a través de uno de varios otros encabezados HTTP que están diseñados para cumplir este propósito, aunque para casos de uso más inocentes.

A veces puede usar `X-Forwarded-Host` para inyectar su entrada maliciosa mientras elude cualquier validación en el encabezado del Host.

```http
GET /example HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: bad-stuff-here
```

 es posible que encuentre otros encabezados que tengan un propósito similar, que incluyen:

- `X-Host`
- `X-Forwarded-Host`
- `X-Forwarded-Server`
- `X-HTTP-Host-Override`
- `Forwarded`

>[!tip]
>En Burp Suite, puede usar la función "Adivinar encabezados" de la extensión [Param Miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943) para sondear automáticamente los encabezados admitidos utilizando su extensa lista de palabras integrada.

## EXPLOTACION

### Basic password reset poisoning

una aplicacion tiene la opcion de reestablecimeinto de contraseña a partir de un **username** o **email** dado:

![[Pasted image 20230221215609.png]]

al enviar vemos que llega un correo en el buzon asi:

![[Pasted image 20230221215658.png]]

el email contiene el link para hacer el reset de la contraseña y hace la validacion con un valor `temp-forgot-password-token` para identificar al usuario, esta pagina tiene vulnerabilidad en el header `host` porque usa este header para generara el link de reseteo de contraseña:

![[Pasted image 20230221220534.png]]

![[Pasted image 20230221220550.png]]

podemos solicitar el reseteo para otro usuario que conozcamos e inyectar nuestro servidor en el header `host`, cuando el acceda a la URL en los logs del servidor podemos ver el **token temporal**:

![[Pasted image 20230221221908.png]]

![[Pasted image 20230221221755.png]]

si revisamos los logs en el servidor:

![[Pasted image 20230221221821.png]]

ahora podemos consultar un link verdadero y cambiar el token por el que obtuvimos y podremos modificar la contraseña del usuario victima.

### Host header authentication bypass

Algunas paginas pueden tener un panel de administrador solo para usuarios administradores `/admin`, la pagina puede hacer una validacion por el header `host` y avisa este mensaje:

![[Pasted image 20230221222624.png]]

podemos engañar a la pagina colocando `host: localhost` para que crea que somos un usuario local:

>[!tip]
>puede probar con los otros headers que suplantan el header `host`:
>-   `X-Host`
>-   `X-Forwarded-Server`
>-   `X-HTTP-Host-Override`
>-   `Forwarded`

![[Pasted image 20230221222901.png]]

![[Pasted image 20230221222911.png]]

para eliminar un usuario o realizar cualquier accion en ese panel, debemos modificar tambien el header `host`:

![[Pasted image 20230221223003.png]]

### Acceso a sitios web internos con fuerza bruta de host virtual

Las empresas a veces cometen el error de alojar sitios web de acceso público y sitios internos privados en el mismo servidor. Los servidores suelen tener una dirección IP pública y una privada. Como el nombre de host interno puede resolverse en la dirección IP privada, este escenario no siempre se puede detectar simplemente mirando los registros DNS:

```bash
www.example.com: 12.34.56.78
intranet.example.com: 10.0.0.132
```

 Si han descubierto un nombre de dominio oculto por otros medios, como la divulgación de información , simplemente pueden solicitarlo directamente.

### Web cache poisoning via ambiguous requests

Una pagina hace uso de cache para cargar contenido, ademas carga contenido JS llamando a una ruta y toma el header `host` como URL de `src` y cuando se le envia 2 header `host` podemos inyectar nuestro servidor malicioso:

>[!note]
>La peticion tiene que estar con el protocolo **HTTP 1.1**

![[Pasted image 20230221225038.png]]

vamos a crear en nuestro servidor malicioso un script JS que realice un `alert(document.cookie)` y que se llame como lo carga la pagina `/resources/js/tracking.js`:

![[Pasted image 20230221225151.png]]

ahora quitamos el **cache buster** y vamos a envenenar la cache hasta que encontremos en la respuesta el header `X-Cache: miss`:

![[Pasted image 20230221225325.png]]

si recargamos la pagina:

![[Pasted image 20230221225421.png]]

si la victima tambien lo recarga vera el `alert()`.

>[!success]
>se almacena en cache por 30 segundo por lo que para ver el popup apenas tengamos la respuesta `X-Cache: miss` tenemos que actualizar la pagina en los siguientes 30 segundos.

### Routing-Based SSRF

Puede que conozcamos un segemento de la red interna de una aplicacion, a traves de una mala configuracion podemos acceder a contenido interno.

Si están configurados de manera insegura para reenviar solicitudes basadas en el header de host no validado, pueden manipularse para desviar solicitudes a un sistema arbitrario elegido por el atacante.

podemos usar el **burp collaborator** para verificar que el header `host` consulta el valor que le colocamos, modificamos su valor con nuestro **collaborator**:

![[Pasted image 20230221231429.png]]

![[Pasted image 20230221231454.png]]

entonces, supongamos que conocemos un segmento interno `192.168.0.0/24` por lo que podemos usar el intruder para ver si alguna pagina nos responde a `/`:

![[Pasted image 20230221231621.png]]

si filtramos por codigo de estado:

![[Pasted image 20230221231708.png]]

podemos consultar el panel administrativo:

![[Pasted image 20230221231840.png]]

![[Pasted image 20230221231850.png]]

podemos realizar las acciones de ese panel administrativo interno pero tenemos que modificar el header `host` nuevamente:

![[Pasted image 20230221232106.png]]

### SSRF via flawed request parsing

Una pagina permite colocar la ruta absoluta en la consulta:

![[Pasted image 20230221233500.png]]

si modificamos el header `host` para consultar nuestro **collaborator** vemos que responde, pero tenemos que dejar la ruta absoluta:

![[Pasted image 20230221233706.png]]

al igual que el anterior caso, podemos intentar descubrir servidores internos si conocemos un segmento valido, por ejemplo `192.168.0.0/24`:

![[Pasted image 20230221233927.png]]

![[Pasted image 20230221233954.png]]

podemos consultar el panel administrativo ya una vez que descubrimos el host (**pero debemos dejar la ruta absoluta**)

![[Pasted image 20230221234145.png]]

para realizar las acciones del panel administrativo igual debemos dejar la ruta absoluta:

![[Pasted image 20230221234356.png]]

### Host validation bypass via connection state attack

Por motivos de rendimiento, muchos sitios web reutilizan las conexiones para múltiples ciclos de solicitud/respuesta con el mismo cliente.

Por ejemplo, en ocasiones puede encontrarse con servidores que solo realizan una validación exhaustiva en la primera solicitud que reciben a través de una nueva conexión. En este caso, puede eludir potencialmente esta validación enviando una solicitud inicial de aspecto inocente y luego siguiendo con la maliciosa por la misma conexión.

