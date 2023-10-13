# Web Cache Poisoning

## Table of contents

- [Web Cache Poisoning](#web-cache-poisoning)
  - [INTRODUCCION](#introduccion)
  - [Beneficios del almacenamiento en caché](#beneficios-del-almacenamiento-en-cach)
  - [¿Cómo funcionan los cachés web?](#cmo-funcionan-los-cachs-web)
  - [Identificacion de respuesta de cache](#identificacion-de-respuesta-de-cache)
    - [Almacenamiento en cache](#almacenamiento-en-cache)
    - [Tiempo en cache](#tiempo-en-cache)
  - [CACHE KEYS](#cache-keys)
  - [Identificación de parámetros unkeyed (MANUAL)](#identificacin-de-parmetros-unkeyed-manual)
    - [Parámetros GET unkeyed (MANUALMENTE)](#parmetros-get-unkeyed-manualmente)
    - [Headers unkeyed (MANUALMENTE)](#headers-unkeyed-manualmente)
  - [CACHE BUSTER (ENTORNO REAL)](#cache-buster-entorno-real)
  - [Identificación de parámetros unkeyed (AUTOMATIZADO)](#identificacin-de-parmetros-unkeyed-automatizado)
    - [PARAM MINER](#param-miner)
    - [BURP SUITE ACTIVE SCAN](#burp-suite-active-scan)
  - [PARAMETER CLOACKING (ENCUBRIMIENTO DE PARAMETROS)](#parameter-cloacking-encubrimiento-de-parametros)
  - [FAT GET Request](#fat-get-request)
  - [Normalized cache keys](#normalized-cache-keys)
  - [Explotacion](#explotacion)
    - [Unkeyed port](#unkeyed-port)
    - [Web cache poisoning with an unkeyed header](#web-cache-poisoning-with-an-unkeyed-header)
    - [Web cache poisoning with an unkeyed cookie](#web-cache-poisoning-with-an-unkeyed-cookie)
  - [Web cache poisoning with multiple headers](#web-cache-poisoning-with-multiple-headers)
    - [Targeted web cache poisoning using an unknown header](#targeted-web-cache-poisoning-using-an-unknown-header)
    - [Web cache poisoning via an unkeyed query string](#web-cache-poisoning-via-an-unkeyed-query-string)
    - [Web cache poisoning via an unkeyed query parameter](#web-cache-poisoning-via-an-unkeyed-query-parameter)
    - [Parameter cloaking](#parameter-cloaking)
    - [Web cache poisoning via a fat GET request](#web-cache-poisoning-via-a-fat-get-request)
    - [URL normalization](#url-normalization)


## INTRODUCCION

El envenenamiento de caché web es un vector de ataque avanzado que obliga a un caché web a proporcionar contenido malicioso a los usuarios desprevenidos que visitan el sitio vulnerable. Los cachés web se utilizan a menudo en la implementación de aplicaciones web por motivos de rendimiento, ya que reducen la carga en el servidor web. Al explotar el envenenamiento de caché web, un atacante puede entregar contenido malicioso a una gran cantidad de usuarios. 

El envenenamiento de caché web también se puede utilizar para aumentar la gravedad de las vulnerabilidades en la aplicación web. Por ejemplo, se puede usar para entregar cargas útiles de XSS reflejadas a todos los usuarios que visitan el sitio web, eliminando así la necesidad de la interacción del usuario que generalmente se requiere para explotar las vulnerabilidades de XSS reflejadas.

Los cachés web comúnmente utilizados incluyen [Apache](https://httpd.apache.org/) , [Nginx](https://nginx.org/) y [Squid](http://www.squid-cache.org/) .

## Beneficios del almacenamiento en caché

Cuando se proporciona un servicio web que es utilizado por una gran cantidad de usuarios, la escalabilidad es importante. Cuantos más usuarios utilicen el servicio, más carga se genera en el servidor web. Para reducir la carga del servidor web y distribuir usuarios entre servidores web redundantes, se pueden usar `Content Delivery Networks (CDNs)` y proxies inversos.

## ¿Cómo funcionan los cachés web?

Los cachés web almacenan recursos para reducir la carga en el servidor web. Estos pueden ser recursos estáticos, como hojas de estilo o archivos de secuencias de comandos, pero también respuestas dinámicas generadas por el servidor web en función de los datos proporcionados por el usuario, como consultas de búsqueda. Para servir recursos almacenados en caché, el caché web debe poder distinguir entre solicitudes para determinar si dos solicitudes pueden recibir la misma respuesta almacenada en caché o si es necesario obtener una respuesta nueva del servidor web.

Están ubicados entre el cliente y el servidor y brindan contenido desde su almacenamiento local en lugar de solicitarlo al servidor web.

![[Pasted image 20230216103657.png]]

Para evitar estos problemas, los cachés web solo usan un subconjunto de todos los parámetros de solicitud para determinar si dos solicitudes deben recibir la misma respuesta. Este subconjunto se llama `Cache Key`. En la mayoría de las configuraciones predeterminadas, esto incluye la ruta de la solicitud (URL), los parámetros GET y el header `Host`. 

Sin embargo, las claves de caché se pueden configurar individualmente para incluir o excluir cualquier parámetro o encabezado HTTP, para que funcionen de manera óptima para una aplicación web específica.

Suponiendo que la caché web está configurada para usar la configuración predeterminada de la ruta de solicitud, los parámetros GET y el header del host como clave de caché, las siguientes dos solicitudes recibirán la misma respuesta.

primera solicitud:

```http
GET /index.html?language=en HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.5359.125 Safari/537.36
Accept: text/html
```

La segunda solicitud tiene la misma clave de caché y, por lo tanto, recibe la misma respuesta.

```http
GET /index.html?language=en HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 13_1) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.1 Safari/605.1.15
Accept: text/html,application/xhtml+xml,application/xml
```

Las solicitudes difieren en el header  del `user-agent` debido a los diferentes sistemas operativos y `Accept` también contienen un header diferente.

Sin embargo, si un tercer usuario solicita la misma página en un idioma diferente a través del parámetro GET `language`, la clave de caché es diferente (ya que los parámetros GET son parte de la clave de caché), por lo que el caché web ofrece una respuesta diferente.

```http
GET /index.html?language=de HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 13_1) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.1 Safari/605.1.15
Accept: text/html
```

**_Distinguimos entre parámetros `keyed` y parámetros `unkeyed`. Se llaman todos los parámetros que forman parte de la clave de caché `keyed`, mientras que todos los demás parámetros lo son `unkeyed`. En el ejemplo anterior, los encabezados User-Agent y Accept son `unkeyed`._**

## Identificacion de respuesta de cache

### Almacenamiento en cache

Podemos ver que la primera respuesta no se almacena en caché:

![[Pasted image 20230216111138.png]]

mientras que la segunda respuesta se sirve desde el caché, como se indica en el encabezado `X-Cache-Status` en la respuesta:

![[Pasted image 20230216111206.png]]

>[!tip]
>El nombre del header puede variar segun el servidor web: `X-Cache` o `X-Cache-Status`

### Tiempo en cache

El header `age` contiene el tiempo en segundos que el objeto estuvo en un caché de proxy.
Si es `Age: 0`, probablemente se obtuvo del servidor de origen.

>[!success]
>`age` expresa el valor en segundos de cuanto tiempo esta la respues en la cache, va incrementando en 1 segundo hasta llegar al valor de `max-age`, una vez que alcanza este valor se reinicia la cache por lo que la consulta que cae en ese momento es consultada en el servidor y no la cache, entonces es el momento para envenenar la cache.

La directiva `max-age` establece la cantidad máxima de tiempo en segundos que las respuestas obtenidas pueden usarse nuevamente (desde el momento en que se realiza una solicitud). Por ejemplo, `max-age=90` indica que un activo se puede reutilizar (permanece en la memoria caché del navegador) durante los próximos 90 segundos.

![[Pasted image 20230216153259.png]]

![[Pasted image 20230216154607.png]]

Este ultimo nos indica cuanto tiempo puede durar nuestro envenenamiento en cache, por lo que tenemos ese tiempo para que lo que logremos cargar en cache envenene a otrs usuarios, siempre y cuando otro usuario no envie envie algo al cache.

## CACHE KEYS

**_Distinguimos entre parámetros `keyed` y parámetros `unkeyed`. Se llaman todos los parámetros que forman parte de la clave de caché `keyed`, mientras que todos los demás parámetros lo son `unkeyed`_**

las clave de cache (parametros `keyed`) normalmente son valores que identifican a una request como:

- el path (URL+URL+PARAMETERS), en algunos casos no incluye parametros
- el header `host`

![[Pasted image 20230217115710.png]]

>[!note]
>Estos 2 valores son los mas comunes, pero no los unicos depende de cada pagina que define como una clave de cache.

cuando llega una request se consulta a la cache por estos parametros `keyed` para devolver la respuesta almacenada si se encuentra en cache.

Podemos usar el header `Pragma: x-get-cache-key` que si esta configurado nos indica en la respuesta cuales son los headers, cookies o pearametros que con clave cache `keyed`.

los valores restantes son llamados `unkeyed` ya que si los modificamos se guardan en cache pero no son comparados al consultar la cache, es quiere decir que un usuario que consulta los mismos calores `keyed` puede recibir nuestro valores `unkeyed`, siempre y cuando estos se reflejen en la respuesta.

## Identificación de parámetros unkeyed (MANUAL)

### Introduccion

En un ataque de envenenamiento de caché web, queremos forzar el caché web para que proporcione contenido malicioso a otros usuarios. Para hacerlo, necesitamos identificar parámetros sin clave que podamos usar para inyectar una carga útil maliciosa en la respuesta. El parámetro para entregar la carga útil debe estar sin clave, ya que los parámetros con clave deben ser los mismos cuando la víctima accede al recurso.

En este caso, podemos entregar la carga útil mediante la URL `/index.php?language=en&ref="><script>alert(1)</script>`. Si el caché web luego almacena la respuesta, envenenamos con éxito el caché, lo que significa que todos los usuarios posteriores que soliciten el recurso `/index.php?language=en` recibirán nuestra carga útil XSS. Si se tecló el parámetro ref, la solicitud de la víctima daría como resultado una clave de caché diferente, por lo que la caché web no entregaría la respuesta envenenada.

**_Por lo tanto, el primer paso para identificar vulnerabilidades de envenenamiento de caché web es identificar parámetros sin clave. _**

Otra conclusión importante es que el envenenamiento de caché web en la mayoría de los casos solo ayuda a facilitar la explotación de otras vulnerabilidades que ya están presentes en la aplicación web subyacente, como XSS reflejado o host header attack.

Los parámetros de solicitud sin clave se pueden identificar al observar si recibimos una respuesta en caché o nueva. Como se discutió anteriormente, los parámetros de solicitud incluyen la ruta, los parámetros GET y los encabezados HTTP. Determinar si una respuesta se almacenó en caché o no puede ser difícil. En nuestro ejemplo, el servidor nos informa a través del header `X-Cache-Status`; sin embargo, en un entorno real, este puede no ser el caso.

### Parámetros GET unkeyed (MANUALMENTE)

si identificamos un parametro GET y la evidencia en la respuesta del header `X-Cache-Status` o algun otro que indique que se cargo la respuesta de la cache, tenemos que hacer lo siguiente:

Ejemplo parametro `?ref=`:

vamos a consultar este parametro 3 veces:

- Las 2 primeras solicitudes se envia un mismo valor en el parametro, la 3 solicitud se envia un valor diferente.
- La primera solicitud la cache debe fallar, los siguientes 2 debe retornar de la cache.

- **SOLICITUD 1**: Solicitud de `/index.php?language=valuewedidnotusebefore&ref=test123` **RESPUESTA** resulta en una falla de caché `X-Cache-Status: MISS`. (**Sale MISS por que se guarda en cache**)
- **SOLICITUD 2**: La misma solicitud da como `/index.php?language=valuewedidnotusebefore&ref=test123`**RESPUESTA** resultado un acierto de caché `X-Cache-Status: HIT`. (**Sale HIT porque en la solicitud 1 se guardó en cache**)
- **SOLICITUD 3**: La solicitud `/index.php?language=valuewedidnotusebefore&ref=Hello` **RESPUESTA** también da como resultado un acierto de caché `X-Cache-Status: HIT`. (**Sale HIT porque se guarda en cache**)

en este ejemplo el parametro `ref` es `unkeyed` y puede ser vulnerable a **web cache poisoning**.

Ahora, para determinar si elparámetro  `ref` es explotable o no, necesitamos determinar cómo este parámetro influye en el contenido de la respuesta. Podemos ver que su valor se refleja en el formulario de envío:

![[Pasted image 20230216115207.png]]

Dado que no se aplica desinfección, podemos salir fácilmente del elemento HTML y activar un XSS reflejado así:

```http
GET /index.php?language=unusedvalue&ref="><script>alert(1)</script> HTTP/1.1
Host: webcache.htb
```

>[!tip]
>**Esto funciona bien en nuestro entorno de prueba; sin embargo, en un compromiso real en el que una gran cantidad de usuarios acceden potencialmente al sitio de destino durante nuestra prueba, es casi imposible determinar si recibimos una respuesta almacenada en caché debido a un parámetro sin clave o porque la solicitud de otro usuario hizo que la respuesta se almacenara en caché. Por lo tanto, siempre debemos usar `Cache Busters` en un compromiso del mundo real.**

### Headers unkeyed (MANUALMENTE)

**_Podemos aplicar la misma metodología de antes para determinar que este encabezado no tiene clave `unkeyed`._**

es bastante común encontrar encabezados HTTP sin clave que influyen en la respuesta del servidor web. En la aplicación web actual, supongamos que encontramos el encabezado HTTP personalizado `X-Backend-Server`que parece ser un encabezado de depuración sobrante que influye en la ubicación desde la que se carga un script de depuración: 

![[Pasted image 20230216115328.png]]

Dado que este encabezado se refleja sin saneamiento, también podemos usarlo para explotar una vulnerabilidad XSS. Por lo tanto, podemos usar el encabezado para entregar la misma carga útil que antes con la siguiente solicitud:

```http
GET /index.php?language=de HTTP/1.1
Host: webcache.htb
X-Backend-Server: testserver.htb"></script><script>var xhr=new XMLHttpRequest();xhr.open('GET','/admin.php?reveal_flag=1',true);xhr.withCredentials=true;xhr.send();//
```

>[!tip]
>**Esto funciona bien en nuestro entorno de prueba; sin embargo, en un compromiso real en el que una gran cantidad de usuarios acceden potencialmente al sitio de destino durante nuestra prueba, es casi imposible determinar si recibimos una respuesta almacenada en caché debido a un parámetro sin clave o porque la solicitud de otro usuario hizo que la respuesta se almacenara en caché. Por lo tanto, siempre debemos usar `Cache Busters` en un compromiso del mundo real.**

## CACHE BUSTER (ENTORNO REAL)

En escenarios del mundo real, debemos asegurarnos de que nuestra respuesta envenenada no se sirva a ningún usuario real de la aplicación web. Podemos lograr esto agregando un `cache buster` a todas nuestras solicitudes. 

Un `cache buster` es un valor de parámetro único que solo usamos para garantizar una clave de caché única. Dado que tenemos una clave de caché única, solo recibimos la respuesta envenenada y ningún usuario real se ve afectado.

por ejemplo:

una pagina web puede tener un parametro que controla el lenguaje de la pagina `?language=`, la opciones mas comunes son **en, es, de, fr, br**. Nuestro `cache buster` podria ser un valor unico que no se usaria por otro usuario.

Entonces supongamos que el parametro `?language=` no `unkeyed`, pero la pagina tiene otro parametro que es `?ref=`, si queremos probar este parametro manualmente y no influya la cache de otros usuarios a nuestras pruebas podemos pasar en el primer parametro un valor unico y probar con el segundo parametro:

```http
GET /index.php?language=unusedvalue&ref=test HTTP/1.1
Host: webcache.htb
```

ese `unusedvalue` puede ser cualquier valor que no se espero sin que afecte a la pagina `pj language=hola`

>[!tip]
>La validacion manual consiste en hacer 3 peticiones. Tenga en cuenta que tenemos que usar un destructor de caché diferente en la **TERCERA** solicitud de prueba, ya que la clave de caché con el parámetro `language=unusedvalue` ya existe debido a nuestra solicitud anterior. Por lo tanto, tendríamos que ajustar ligeramente el valor del parámetro de idioma para obtener una nueva clave de caché única y sin usar.

## Identificación de parámetros unkeyed (AUTOMATIZADO)

### PARAM MINER

>[!danger]
>Cuando realice el ataque ya validado y todo, tendra que desactivar el param miner para que no este envenenando en segundo plano y haga que no funcione su exploit.

- [https://forum.portswigger.net/thread/is-there-a-way-to-stop-a-param-miner-session-fb215440](https://forum.portswigger.net/thread/is-there-a-way-to-stop-a-param-miner-session-fb215440)

### BURP SUITE ACTIVE SCAN

![[Pasted image 20230216152907.png]]

**SOLO REPORTA CON ALGUNOS HEADERS, ES MEJOR COMBINARNO CON EL PARAM MINER**

## PARAMETER CLOACKING (ENCUBRIMIENTO DE PARAMETROS)

Pueden surgir problemas de **parameter cloacking** donde el back-end identifica parámetros distintos que la memoria caché no identifica. El marco **Ruby on Rails**, por ejemplo, interpreta tanto los signos (`&`) y (`;`) como delimitadores. 

Cuando se usa junto con un caché que no permite esto, puede explotar potencialmente otra peculiaridad para anular el valor de un parámetro clave en la lógica de la aplicación.

Considere la siguiente solicitud:

```html
GET /?keyed_param=abc&excluded_param=123;keyed_param=cba
```

Muchos cachés solo interpretarán esto como dos parámetros, delimitados por (`&`):

1.  `keyed_param=abc`
2.  `excluded_param=123;keyed_param=cba`

Una vez que el algoritmo de análisis elimina el `excluded_param`, la clave de caché solo contendrá `keyed_param=abc`. Sin embargo, en el back-end, Ruby on Rails ve el (`;`) y divide la cadena de consulta en tres parámetros separados:

1.  `keyed_param=abc`
2.  `excluded_param=123`
3.  `keyed_param=cba`

Pero ahora hay un nombre de parametro duplicado `keyed_param`. 

Aquí es donde entra en juego la segunda peculiaridad. Si hay parámetros duplicados, cada uno con valores diferentes, Ruby on Rails da prioridad a la ocurrencia final. El resultado final es que la clave de caché contiene un valor de parámetro esperado e inocente, lo que permite que la respuesta almacenada en caché se sirva normalmente a otros usuarios.

Sin embargo, en el back-end, el mismo parámetro tiene un valor completamente diferente, que es nuestra carga útil inyectada. Es este segundo valor el que se pasará al dispositivo y se reflejará en la respuesta envenenada.

Este exploit puede ser especialmente poderoso si le da control sobre una función que se ejecutará. Por ejemplo, si un sitio web utiliza JSONP para realizar una solicitud entre dominios, a menudo contendrá un parámetro `callback` para ejecutar una función determinada en los datos devueltos:

```html
/jsonp?callback=innocentFunction
```

## FAT GET Request

Las solicitudes Fat GET son solicitudes HTTP GET que contienen un cuerpo de solicitud. Dado que los parámetros GET se envían por especificación como parte de la cadena de consulta (URL), puede ser extraño pensar que las solicitudes GET pueden contener un cuerpo de solicitud. 

Sin embargo, cualquier solicitud HTTP puede contener un cuerpo de solicitud, sin importar cuál sea el método. En el caso de una solicitud GET, el cuerpo del mensaje no tiene significado, por lo que **nunca se usa**.

Por lo tanto, se permite explícitamente un cuerpo de solicitud, sin embargo, no debería tener ningún efecto. Por lo tanto, las siguientes dos solicitudes GET son semánticamente equivalentes, ya que el cuerpo debe ignorarse en la segunda:

**solicitud 1**

```http
GET /index.php?param1=Hello&param2=World HTTP/1.1
Host: fatget.wcp.htb
```

**solicitud 2**

```http
GET /index.php?param1=Hello&param2=World HTTP/1.1
Host: fatget.wcp.htb
Content-Length: 10

param3=123
```

En este caso, la clave de caché se basaría en el parametro GET pero del lado del servidor se tomaría del cuerpo.

![[Pasted image 20230217175004.png]]

## Normalized cache keys

Algunas implementaciones de almacenamiento en caché normalizan la entrada con clave al agregarla a la clave de caché. En este caso, las dos solicitudes siguientes tendrían la misma clave:

```html
GET /example?param="><test> 
GET /example?param=%22%3e%3ctest%3e
```

Este comportamiento puede permitirle explotar estas vulnerabilidades XSS que de otro modo serían "no explotables". Si envía una solicitud maliciosa usando Burp Repeater, puede envenenar el caché con una carga útil XSS sin codificar. Cuando la víctima visita la URL maliciosa, su navegador seguirá codificando la carga útil; sin embargo, una vez que la memoria caché normaliza la URL, tendrá la misma clave de memoria caché que la respuesta que contiene su carga no codificada.

Como resultado, el caché entregará la respuesta envenenada y la carga útil se ejecutará en el lado del cliente. Solo debe asegurarse de que el caché esté envenenado cuando la víctima visite la URL.

## Explotacion

>[!success]
>El objetivo es: 
>- determinar si el servidor usa cache
>- encontrar un parametro `unkeyed` (puede ser header, cookie o parametro GET)
>- los parametros `keyed` nos serviran con `cache buster` para que el usuario no sospeche
>- si el parametro `unkeyed` se refleja en el body de la respuesta, podemos inyectar algo malicioso en la cache
>- antes de lanzar el ataque es necesario quitar el `cache buster`

### Unkeyed port

El header `Host` suele ser parte de la clave de caché y, como tal, inicialmente parece un candidato poco probable para inyectar cualquier tipo de carga útil. Sin embargo, algunos sistemas de almacenamiento en caché analizarán el encabezado y **excluirán el puerto** de la clave de caché.

En este caso, puede usar potencialmente este encabezado para el envenenamiento de caché web. Por ejemplo,  una URL de redireccionamiento se generó dinámicamente en función del header `Host`. Esto podría permitirle construir un ataque de denegación de servicio simplemente agregando un puerto arbitrario a la solicitud.

### Web cache poisoning with an unkeyed header

en el inicio de la pagina `/` vamos a realizar un escaneo de headers ocultos con **param miner**:

![[Pasted image 20230216225439.png]]

podemos ver despues de un tiempo en la parte de  **target > issues** que encuentra un valor:

![[Pasted image 20230216225535.png]]

otra forma de descubrir es:

**PETICION 1**

- enviamos un **cache buster** `?a=k`
- enviamos el header que parece vulnerable `X-Forwarded-host` con un valor www.hacker1.com
- El valor del header se ve reflejada en el body
- la respuesta retorna un `X-Cache: miss`


![[Pasted image 20230216230209.png]]

**PETICION 2**

- enviamos un **cache buster** `?a=k`
- enviamos el header que parece vulnerable `X-Forwarded-host` con un valor www.hacker1.com
- El valor del header se ve reflejada en el body
- la respuesta retorna un `X-Cache: hit`

![[Pasted image 20230216230301.png]]

**PETICION 3**

- enviamos un **cache buster** `?a=k`
- enviamos el header que parece vulnerable `X-Forwarded-host` con un valor www.hacker2.com (OTRO VALOR)
- El valor del header ANTERIOR se ve reflejada en el body (www.hacker1.com)
- la respuesta retorna un `X-Cache: hit`

![[Pasted image 20230216230442.png]]

como vemos que el valor de header se refleja en el body y consulta un script:

![[Pasted image 20230216230944.png]]

podemos usar un servidor, crear una ruta `/resources/js/tracking.js` con el contenido malicioso y en el header `X-Forwarded-Host` colocar la direccion del servidor, esto hara que se ejecute o se acrge en codigo fuente nuestro JS.

>[!note]
> - No olvide volver a hacer las 2 primeras peticiones de la identificacion para que se guarde en cache.
> - hagalas ahora sin el `cache buster`
> - verifique que la respuesta contenta su servidor y el header `X-Cacha: hit`
> - apenas tenga eso recargue la pagina principal en su navegador y debera ver el alert
> - tenga en cuenta que hay un lapso de tiempo valido antes de que la cache se reinicie porque un usurio visita la pagina

payload JS:

```javascript
alert('Web Cache Poisoning')
```

![[Pasted image 20230216232823.png]]

### Web cache poisoning with an unkeyed cookie

cuando se trata de cookies es mas complejo que **param miner** lo descubra, si puede descubrirlo usted mismo mucho mejor.

Hay una cookie que se refleja en el body de la respuesta, la idea es similar al anterior, cargar el payload sin el `cache buster` hasta que `X-Cache: hit`

![[Pasted image 20230217002457.png]]

## Web cache poisoning with multiple headers

Puede darse el caso que al juntar 2 headers diferentes pueda ver un reflejo en el body:

![[Pasted image 20230217002626.png]]

los headers `x-forwarded-host` y `x-forwarded-scheme` tiene que estar juntos y apuntar a un mismo servidor.

este caso es mas dificil de detectar ya que un header puede depender de la otra, de ser asi **param miner** puede reportar solo 1:

![[Pasted image 20230217002733.png]]

puede que tengmos que colocar otras opciones en el escaneo de **param miner**:

![[Pasted image 20230217004029.png]]

y quizas tengamos suerte:

![[Pasted image 20230217004108.png]]

la idea es similar al anterior:

- cargar el payload en el servidor con el nombre del script
- crear el payload malicioso como contenido del scrpt
- colocar en los headers el servidor
- consultar el recurso sin el `cache buster` 
- repetir esto hasta que `X-Cache: hit`

### Targeted web cache poisoning using an unknown header

El header `Vary` especifica una lista de encabezados adicionales que deben tratarse como parte de la clave de caché, incluso si normalmente no tienen clave. 
 
Se usa comúnmente para especificar que el header `User-Agent` está codificado, por ejemplo, de modo que si la versión móvil de un sitio web se almacena en caché, esto no se mostrará a los usuarios que no son móviles por error.

Esta información también se puede utilizar para construir un ataque de varios pasos para apuntar a un subconjunto específico de usuarios. (Segun user-agent)

### Web cache poisoning via an unkeyed query string

En este ejemplo aun un parametro GET que se refleja en el body de la respuesta, por lo que no podemos usar `cache buster`, usando paraminer vamos a tratar de descubrir que headers ocultos hay:

![[Pasted image 20230217111929.png]]

parece que existe el header `origin`, podemos colocarlo, ademas podemos usar el header `Pragma: x-get-cache-key` que si esta configurado nos indica en la respuesta cuales son los headers con clave cache:

![[Pasted image 20230217112301.png]]

la respuesta nos muestra que el header origin un valor `keyed` en la cache, podemos agregar cualquier parametro en el GET y vemos que se refleja en el frontend:

![[Pasted image 20230217112741.png]]

```html
?evil='/><script>alert(1)</script>
```

![[Pasted image 20230217112433.png]]

>[!tip]
>la clave cache es el header `origin` por lo que lo podemos usar como `cache buster`

los parametros GET no son parte de la clave cache por lo que podemos colocar nuestro payload y recargar hasta que se guarde en cache, obviamente quitando la clave cache `origin`.

### Web cache poisoning via an unkeyed query parameter

una pagina admite parametros GET que se reflejan en el body:

![[Pasted image 20230217121525.png]]

podemos agregar el header `pragma: x-get-cache-key` para ver su nos indica que si tenemos un valor en nuestra request que es un parametro `keyed`:

![[Pasted image 20230217145553.png]]

como es un parametro `keyed` no podemos usarlo para el envenenamiento, pero podemos buscar algun otro parametro con **param miner** que quizas no lo toma como `keyed`:

>[!note]
>no agregamos `cache buster`

![[Pasted image 20230217151606.png]]

el resultado ni indica despues de unos minutos:

![[Pasted image 20230217151711.png]]

el parametro `utm_content=` parece que no se almacena en cache, probemos con el resultado del header `pragma: x-get-cache-key`:

![[Pasted image 20230217152544.png]]

el parametro efectivamente es `unkeyed` y se refleja en el body, por lo que podemos inyectar un payload en la cache hasta que nos retorne el header `X-Cache: miss`

```bash
/?utm_content=test'<script>alert(1)</script>
```

![[Pasted image 20230217152933.png]]

entonces si consultamos la peticion original veremos que se envenenó la cache:

![[Pasted image 20230217153108.png]]

y si lo consultamos desde un browser:

![[Pasted image 20230217153334.png]]

### Parameter cloaking

en el inicio de una pagina descubrimos que cualquier parametro que enviemos es un parametro `keyed`:

![[Pasted image 20230217170326.png]]

esta entrada se refleja en el body. Con **param miner** descubrimos que existe un parametro que no es `keyed`:

![[Pasted image 20230217170557.png]]

![[Pasted image 20230217170626.png]]

![[Pasted image 20230217170455.png]]

vamos a validar:

![[Pasted image 20230217170805.png]]

por lo que tenemos un valor potencial, adicionalmente encontramos un script JS que tiene un parametro **callback**

![[Pasted image 20230217170955.png]]

el valor del parametro **callback** se refleja en el script, podemos utilizar el parametro **utm_content**  que es `unkeyed` para ver que no afecta:

![[Pasted image 20230217171205.png]]

vamos a aprovechar **parameter cloacking** con este parametro `unkeyed` para agregar un parametro **callback** arbitrario:

```html
/js/geolocate.js?callback=setCountryCookie&utm_content=test;callback=alert(1)
```

la cache vera 2 parametros:

1. `callback=setCountryCookie`
2. `utm_content=test;callback=alert(1)`

el backend vera 3:

1. `callback=setCountryCookie`
2. `utm_content=test`
3. `callback=alert(1)`

como hay un parametro llamado **callback** repetido, el servidor solo considera el ultimo, por lo que podriamos inyectar codigo JS:

![[Pasted image 20230217171652.png]]

si consultamos el recurso original, se enveneno la cache:

![[Pasted image 20230217171735.png]]

si recargamos el navegador:

![[Pasted image 20230217171616.png]]

**PODEMOS DETECTAR ESTO CON PARAM MINER PERO DEBEMOS ESPECIFICAR EL PARAMETRO UNKEYED ENCONTRADO**

![[Pasted image 20230217172008.png]]

![[Pasted image 20230217172048.png]]

![[Pasted image 20230217172151.png]]

### Web cache poisoning via a fat GET request

detectamos que al llamar un script se le pasa un parametro GET llamado **callback** que se refleja en la respuesta:

![[Pasted image 20230217174507.png]]

si fuera vulnerable a **fat get request** sabemos que el parametro vulnerable es el mismo, podemos validar esto con **param miner**:

![[Pasted image 20230217174622.png]]

y le pasamos el parametro vulnerable:

![[Pasted image 20230217174719.png]]

resultado:

![[Pasted image 20230217174809.png]]

la forma de validar manualmente es mandar en el body el mismo parametro con otro valor y ver cual se refleja:

![[Pasted image 20230217175004.png]]

podemos porner nuestro payload para que se refleje:

```javascript
alert(1)
```

![[Pasted image 20230217175448.png]]

si cargamos la peticion original se envenena la cache:

![[Pasted image 20230217175531.png]]

recargamos el navegador y veremos el **alert()**

### URL normalization

si consultamos una URI cualquiera la pagina nos muestra un error y la URI se refleja en la respuesta:

![[Pasted image 20230217201512.png]]

esto parece facil de explotar ya que podemos realizar un XSS:

![[Pasted image 20230217201714.png]]

si cargamos esto desde el browser nos lo muestra URL encoded:

![[Pasted image 20230217201815.png]]

resulta que en la cache se guarda el payload bien, pero al expirar la cache se muestra URL encoded, vemos que la cache se reestablece cada 10 segundos:

![[Pasted image 20230217202054.png]]

por lo que vamos a enviar desde el **repiter** el payload y rapidamente cargamos la misma URL en el navegador para que nos muestre de la cache, si consultamos antes de que pase 10 segundos veremos el popup:

![[Pasted image 20230217202231.png]]

