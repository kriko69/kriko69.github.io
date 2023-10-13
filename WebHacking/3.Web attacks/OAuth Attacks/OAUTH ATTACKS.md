# OAuth Attacks

## Table of contents

- [OAuth Attacks](#oauth-attacks)
  - [¿Qué es OAuth?](#qu-es-oauth)
  - [¿Cómo funciona OAuth 2.0?](#cmo-funciona-oauth-20)
  - [Etapas de OAuth](#etapas-de-oauth)
  - [Grant types (OAuth Flows) - Tipo de Acceso](#grant-types-oauth-flows---tipo-de-acceso)
    - [Que es un OAuth grant type?](#que-es-un-oauth-grant-type)
    - [OAuth Scope](#oauth-scope)
    - [Grant Type - Codigo de autorizacion](#grant-type---codigo-de-autorizacion)
      - [1. Solicitud de autorización](#1-solicitud-de-autorizacin)
      - [2. Inicio de sesión y consentimiento del usuario](#2-inicio-de-sesin-y-consentimiento-del-usuario)
      - [3. Otorgamiento de código de autorización](#3-otorgamiento-de-cdigo-de-autorizacin)
      - [4. Solicitud de token de acceso](#4-solicitud-de-token-de-acceso)
      - [5. Concesión de token de acceso](#5-concesin-de-token-de-acceso)
      - [6. Llamada a la API](#6-llamada-a-la-api)
      - [7. Donación de recursos](#7-donacin-de-recursos)
    - [Grant Type - Implicita](#grant-type---implicita)
      - [1. Solicitud de autorización](#1-solicitud-de-autorizacin)
      - [2. Inicio de sesión y consentimiento del usuario](#2-inicio-de-sesin-y-consentimiento-del-usuario)
      - [3. Concesión de token de acceso](#3-concesin-de-token-de-acceso)
      - [4. Llamada API](#4-llamada-api)
      - [5. Donación de recursos](#5-donacin-de-recursos)
  - [OPENID Connect](#openid-connect)
    - [¿Qué es Open ID Connect?](#qu-es-open-id-connect)
    - [Funciones de OpenID Connect](#funciones-de-openid-connect)
    - [OpenID Connect claims and scopes](#openid-connect-claims-and-scopes)
    - [Identificación de OpenID Connect](#identificacin-de-openid-connect)
  - [Autenticación OAuth](#autenticacin-oauth)
  - [Identificación de la autenticación OAuth](#identificacin-de-la-autenticacin-oauth)
  - [Reconocimiento](#reconocimiento)
  - [OAuth Attacks](#oauth-attacks)
    - [Authentication bypass via OAuth implicit flow](#authentication-bypass-via-oauth-implicit-flow)
    - [Forced OAuth profile linking (CSRF Attack - no state parameter)](#forced-oauth-profile-linking-csrf-attack---no-state-parameter)
    - [OAuth account hijacking via redirect_uri](#oauth-account-hijacking-via-redirect_uri)
    - [Stealing OAuth access tokens via an open redirect](#stealing-oauth-access-tokens-via-an-open-redirect)
    - [SSRF via OpenID dynamic client registration](#ssrf-via-openid-dynamic-client-registration)

![[Pasted image 20230209152143.png]]

## ¿Qué es OAuth?

OAuth es un marco de autorización de uso común que permite que los sitios web y las aplicaciones web soliciten acceso limitado a la cuenta de un usuario en otra aplicación. Fundamentalmente, OAuth permite al usuario otorgar este acceso sin exponer sus credenciales de inicio de sesión a la aplicación solicitante. Esto significa que los usuarios pueden ajustar qué datos quieren compartir en lugar de tener que ceder el control total de su cuenta a un tercero.

El proceso básico de OAuth se usa ampliamente para integrar funciones de terceros que requieren acceso a ciertos datos de la cuenta de un usuario. Por ejemplo, mientras navega por la web, es casi seguro que se ha topado con sitios que le permiten iniciar sesión con su cuenta de redes sociales. Lo más probable es que esta función se cree utilizando el popular marco OAuth 2.0.

## ¿Cómo funciona OAuth 2.0?

OAuth 2.0 se desarrolló originalmente como una forma de compartir el acceso a datos específicos entre aplicaciones. Funciona definiendo una serie de interacciones entre tres partes distintas:

- **Aplicación cliente**: El sitio web o aplicación web que quiere acceder a los datos del usuario.
- **Propietario del recurso**: el usuario a cuyos datos desea acceder la **aplicación cliente**.
- **Proveedor de servicios OAuth**: El sitio web o aplicación que controla los datos del usuario y el acceso a los mismos. Admiten OAuth al proporcionar una API para interactuar tanto con un servidor de autorización como con un servidor de recursos.

## Etapas de OAuth

En términos generales, OAuth implica las siguientes etapas:

1. La aplicación cliente solicita acceso a un subconjunto de los datos del usuario, especificando qué tipo de concesión quieren usar y qué tipo de acceso quieren.
2. Se solicita al usuario que inicie sesión en el servicio OAuth y dé explícitamente su consentimiento para el acceso solicitado (Cuando ya esta logueado).
3. La aplicación cliente recibe un token de acceso único que prueba que tiene permiso del usuario para acceder a los datos solicitados. Exactamente cómo sucede esto varía significativamente según el tipo de concesion (Grant Type). 
4. La aplicación cliente utiliza este token de acceso para realizar llamadas a la API y obtener los datos relevantes del servidor de recursos.

## Grant types (OAuth Flows) - Tipo de Acceso

### Que es un OAuth grant type?

El tipo de concesión de OAuth (OAuth grant type) determina:

- La secuencia exacta de pasos que intervienen en el proceso de OAuth. 
- La forma en que la aplicación cliente se comunica con el servicio OAuth en cada etapa, incluida la forma en que se envía el token de acceso. 

Por esta razón, los tipos de concesión a menudo se denominan "OAuth flows".

Un servicio OAuth debe configurarse para admitir un tipo de concesión en particular antes de que una aplicación cliente pueda iniciar el flujo correspondiente. La aplicación cliente especifica qué tipo de concesión desea usar en la solicitud de autorización inicial que envía al servicio OAuth.

### OAuth Scope

Para cualquier tipo de concesión de OAuth, la aplicación cliente debe especificar a qué datos desea acceder y qué tipo de operaciones desea realizar. Lo hace utilizando el parámetro `scope` de la solicitud de autorización que envía al servicio OAuth.

### Grant Type - Codigo de autorizacion

La aplicación cliente y el servicio OAuth primero usan redireccionamientos para intercambiar una serie de solicitudes HTTP basadas en navegador que inician el flujo. Se pregunta al usuario si da su consentimiento para el acceso solicitado. Si aceptan, a la aplicación cliente se le otorga un "código de autorización". Luego, la aplicación del cliente intercambia este código con el servicio OAuth para recibir un "token de acceso", que pueden usar para realizar llamadas API para obtener los datos de usuario relevantes.

Toda la comunicación que tiene lugar desde el intercambio de código/token en adelante se envía de servidor a servidor a través de un canal secundario seguro y preconfigurado y, por lo tanto, es invisible para el usuario final.

Este canal seguro se establece cuando la aplicación cliente se registra por primera vez en el servicio OAuth. En este momento, un `client_secret` es tambien generado, el cual la aplicacion cliente debe usar para autenticarse cuando envía estas solicitudes de servidor a servidor.

>[!note]
>Dado que los datos más confidenciales (el token de acceso y los datos del usuario) no se envían a través del navegador, este tipo de concesión es posiblemente el más seguro. Idealmente, las aplicaciones del lado del servidor siempre deberían usar este tipo de concesión si es posible.

![[Pasted image 20230209161143.png]]

#### 1. Solicitud de autorización

La aplicación cliente envía una solicitud al endpoint `/authorization` del servicio OAuth solicitando permiso para acceder a datos de usuario específicos.

>[!note]
>Tenga en cuenta que el mapeo de puntos finales puede variar entre proveedores. Sin embargo, siempre debería poder identificar el punto de conexión en función de los parámetros utilizados en la solicitud.

Ejemplo:

```http
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

Esta solicitud contiene los siguientes parámetros GET:

- **client_id**: Parámetro obligatorio que contiene el identificador único de la aplicación cliente. Este valor se genera cuando la aplicación cliente se registra con el servicio OAuth.   

- **redirect_uri**: El URI al que se debe redirigir el navegador del usuario al enviar el código de autorización a la aplicación cliente. **Muchos ataques de OAuth se basan en la explotación de fallas en la validación de este parámetro.**

- **response_type**: Determina qué tipo de respuesta espera la aplicación cliente y, por lo tanto, **qué flujo desea iniciar**. _Para el tipo de concesión de código de autorización, el valor debe ser `code`.

- **scope**: Se utiliza para especificar a qué subconjunto de datos del usuario desea acceder la aplicación cliente. Tenga en cuenta que estos pueden ser ámbitos personalizados establecidos por el proveedor de OAuth o ámbitos estandarizados definidos por la especificación de OpenID Connect. [[#OPENID Connect]]

- **state**: Almacena un valor único e imposible de adivinar que está vinculado a la sesión actual en la aplicación cliente. El servicio OAuth debería devolver este valor exacto en la respuesta, junto con el código de autorización. Este parámetro sirve como una forma de token **CSRF** para la aplicación cliente al asegurarse de que la solicitud a su endpoint `/callback`  sea de la misma persona que inició el flujo de OAuth.

#### 2. Inicio de sesión y consentimiento del usuario

Cuando el servidor de autorización recibe la solicitud inicial, redirigirá al usuario a una página de inicio de sesión, donde se le pedirá que inicie sesión en su cuenta con el proveedor de OAuth. Por ejemplo, **esta suele ser su cuenta de redes sociales**.

Luego se les presentará una lista de datos a los que la aplicación cliente desea acceder. Esto se basa en los alcances definidos en la solicitud de autorización. El usuario puede elegir si consentir o no este acceso.

![[Pasted image 20230209163214.png]]

>[!note]
> La primera vez que el usuario selecciona "Iniciar sesión con las redes sociales", deberá iniciar sesión manualmente y dar su consentimiento, pero si vuelve a visitar la aplicación del cliente más tarde, a menudo podrá volver a iniciar sesión con un un solo click.

#### 3. Otorgamiento de código de autorización

Si el usuario acepta el acceso solicitado, su navegador será redirigido al endpoint `/callback` que se especificó en el parámetro `redirect_uri` de la solicitud de autorización. La solicitud resultante `GET` contendrá el código de autorización como parámetro de consulta. 

Dependiendo de la configuración, también puede enviar el parámetro `state` con el mismo valor que en la solicitud de autorización.

Ejemplo:

```bash
GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1 
Host: client-app.com
```

#### 4. Solicitud de token de acceso

Una vez que la aplicación cliente recibe el código de autorización, debe cambiarlo por un token de acceso. Para hacer esto, envía una solicitud de servidor a servidor a traves de una peticion `POST` al endpoint  `/token` del servicio OAuth. 

**_Toda la comunicación a partir de este punto se lleva a cabo en un canal trasero seguro y, por lo tanto, normalmente un atacante no puede observarla ni controlarla._**

Ejemplo.

```bash
POST /token HTTP/1.1 
Host: oauth-authorization-server.com

…
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8
```

Ademas de los parametros `client_id` y `code`, notará los siguientes parámetros nuevos:

- **client_secret**: La aplicación cliente debe autenticarse incluyendo la clave secreta que se le asignó al registrarse en el servicio OAuth. (Cambia clave por token)

- **grant_type**: Se usa para asegurarse de que el nuevo extremo sepa qué tipo de concesión desea usar la aplicación cliente. En este caso, esto debe establecerse en `authorization_code`.

#### 5. Concesión de token de acceso

El servicio OAuth validará la solicitud del token de acceso. Si todo es como se esperaba, el servidor responde otorgando a la aplicación cliente un token de acceso con el alcance solicitado.

Ejemplo:

```bash
{
    "access_token": "z0y9x8w7v6u5",
    "token_type": "Bearer",
    "expires_in": 3600,
    "scope": "openid profile",
    …
}
```

#### 6. Llamada a la API

Ahora que la aplicación cliente tiene el código de acceso, finalmente puede obtener los datos del usuario del servidor de recursos. Para ello, realiza una llamada API al endpoint `/userinfo` del servicio OAuth. 

El token de acceso se envía en el header `Authorization: Bearer` para demostrar que la aplicación cliente tiene permiso para acceder a estos datos.

Ejemplo:

```bash
GET /userinfo HTTP/1.1
Host: oauth-resource-server.com
Authorization: Bearer z0y9x8w7v6u5
```

#### 7. Donación de recursos

El servidor de recursos debe verificar que el token sea válido y que pertenezca a la aplicación cliente actual. Si es así, responderá enviando el recurso solicitado, es decir, los datos del usuario según el alcance del token de acceso.

Ejemplo:

```bash
{
    "username":"carlos",
    "email":"carlos@carlos-montoya.net",
    …
}
```

La aplicación cliente finalmente puede usar estos datos para su propósito previsto.

### Grant Type - Implicita

El tipo de concesión implícita es mucho más simple. En lugar de obtener primero un código de autorización y luego cambiarlo por un token de acceso, la aplicación cliente recibe el token de acceso inmediatamente después de que el usuario da su consentimiento.

Cuando se usa el tipo de concesión implícita, toda la comunicación se realiza a través de redireccionamientos del navegador; no hay un canal secundario seguro como en el flujo del código de autorización. Esto significa que el token de acceso confidencial y los datos del usuario están más expuestos a posibles ataques.

>[!note]
>El tipo de concesión implícita es más adecuado para aplicaciones de una sola página y aplicaciones de escritorio nativas, que no pueden almacenarlas fácilmente en el back-end el valor `client_secret`.

![[Pasted image 20230209190841.png]]

#### 1. Solicitud de autorización

**El tipo de concesión implícita** comienza de la misma manera que el **El tipo de concesión mediante codigo de autorizacion**. La única diferencia importante es que el parámetro `response_type` debe establecerse en `token`.

Ejemplo:

```bash
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1

Host: oauth-authorization-server.com
```

#### 2. Inicio de sesión y consentimiento del usuario

El usuario inicia sesión y decide si acepta o no los permisos solicitados. Este proceso es exactamente el mismo que para **El tipo de concesión mediante codigo de autorizacion**.

#### 3. Concesión de token de acceso

Si el usuario da su consentimiento para el acceso solicitado, el servicio OAuth redirigirá el navegador del usuario al parametro `redirect_uri` especificado en la solicitud de autorización (PASO 1). Enviará el token de acceso y otros datos específicos del token como un fragmento de URL.

Ejemplo:

```bash
GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1

Host: client-app.com
```

>[!note]
>Como el token de acceso se envía en un fragmento de URL, nunca se envía directamente a la aplicación cliente. En su lugar, la aplicación cliente debe utilizar un script adecuado para extraer el fragmento y almacenarlo.

#### 4. Llamada API

Una vez que la aplicación cliente extrajo con éxito el token de acceso del fragmento de URL, puede usarlo para realizar llamadas API al endpoint `/userinfo` del servicio OAuth. 

**_A diferencia del flujo del código de autorización, esto también ocurre a través del navegador._**

Ejemplo:

```bash
GET /userinfo HTTP/1.1
Host: oauth-resource-server.com
Authorization: Bearer z0y9x8w7v6u5
```

#### 5. Donación de recursos

El servidor de recursos debe verificar que el token sea válido y que pertenezca a la aplicación cliente actual. Si es así, responderá enviando el recurso solicitado, es decir, los datos del usuario en función del alcance asociado con el token de acceso.

```bash
{
    "username":"carlos",
    "email":"carlos@carlos-montoya.net"
}
```

## OPENID Connect

### ¿Qué es Open ID Connect?

OpenID Connect amplía el protocolo OAuth para proporcionar una capa de identidad y autenticación, agrega una funcionalidad simple que permite un mejor soporte para el caso de uso de autenticación de OAuth.

### Funciones de OpenID Connect

Las funciones de OpenID Connect son esencialmente las mismas que las de OAuth estándar. La principal diferencia es que la especificación utiliza una terminología ligeramente diferente.

- **Parte de confianza**: la aplicación que solicita la autenticación de un usuario. Esto es sinónimo de la aplicación de cliente OAuth.
- **Usuario final**: el usuario que se está autenticando. Esto es sinónimo del propietario del recurso OAuth.
- **Proveedor de OpenID**: un servicio de OAuth que está configurado para admitir OpenID Connect.

### OpenID Connect claims and scopes

El término "claims" (reclamaciones) hace referencia a los pares `key:value` que representan información sobre el usuario en el servidor de recursos. Un ejemplo de un reclamo podría ser:

```json
"family_name":"Montoya"
```

Todos los servicios de OpenID Connect utilizan un conjunto idéntico de ámbitos. Para usar OpenID Connect, la aplicación cliente debe especificar el scope `openid` en la solicitud de autorización. Luego pueden incluir uno o más de los otros ámbitos estándar:

-   `profile`
-   `email`
-   `address`
-   `phone`

Cada uno de estos ámbitos corresponde al acceso de lectura para un subconjunto de notificaciones sobre el usuario que se definen en la especificación de OpenID. Por ejemplo, solicitar el ámbito `openid profile` otorgará a la aplicación cliente acceso de lectura a una serie de notificaciones relacionadas con la identidad del usuario, como `family_name`, `given_name`, `birth_date`, etc

### Identificación de OpenID Connect

Si la aplicación cliente está utilizando OpenID connect activamente, esto debería ser obvio a partir de la solicitud de autorización. La forma más infalible de verificar es buscar el parametro scope `openid` obligatorio.

Incluso si el proceso de inicio de sesión no parece utilizar inicialmente OpenID Connect, vale la pena comprobar si el servicio OAuth lo admite. Simplemente puede intentar agregar el scope `openid` o cambiar el tipo de respuesta `id_token` y observar si esto da como resultado un error.

También puede acceder al archivo de configuración desde el terminal estándar `/.well-known/openid-configuration`.

## Autenticación OAuth

Aunque originalmente no estaba diseñado para este propósito, OAuth también se ha convertido en un medio para autenticar a los usuarios. Para los mecanismos de autenticación de OAuth, los flujos básicos de OAuth siguen siendo prácticamente los mismos; la principal diferencia es cómo la aplicación cliente utiliza los datos que recibe.

## Identificación de la autenticación OAuth

Reconocer cuándo una aplicación utiliza la autenticación OAuth es relativamente sencillo. 

>[!tip]
>Si ve una opción para iniciar sesión con su cuenta desde un sitio web diferente, esta es una fuerte indicación de que se está utilizando OAuth.

La forma más confiable de identificar la autenticación OAuth es interceptar la comunicacion con un proxy y verificar las peticiones, la primera solicitud del flujo siempre será una solicitud al endpoint `/authorization` que contiene una serie de parámetros de consulta que se utilizan específicamente para OAuth.

Verifique el punto [[#1. Solicitud de autorización]]

## Reconocimiento

Si se utiliza un servicio OAuth externo, debería poder identificar el proveedor específico a partir del nombre de host al que se envía la solicitud de autorización.

Una vez que conozca el nombre de host del servidor de autorización, siempre debe intentar enviar una solicitud `GET` a los siguientes puntos finales estándar:

-   `/.well-known/oauth-authorization-server`
-   `/.well-known/openid-configuration`

Estos a menudo devolverán un archivo de configuración JSON que contiene información clave, como detalles de funciones adicionales que pueden ser compatibles. Esto a veces le informará sobre una superficie de ataque más amplia y funciones compatibles que pueden no mencionarse en la documentación.

## OAuth Attacks

### Authentication bypass via OAuth implicit flow

Si tenemos unas credenciales de usuario validas nuestras podemos interceptar y ver el flujo de OAuth con burpsuite:

![[Pasted image 20230209222530.png]]

podemos ver todos los pasos del flujo de OAuth implicito. La peticion POST `/authenticate` envia la informacion del usuario junto con su token otorgado por OAuth, estos datos enviados con como los datos enviados en un inicio de sesion cualquiera. 

Ahora si no se realiza una correcta validacion del token para ese usuario podemos modificar los valores de esa peticion para intentar evadir la autenticacion y entrar como otro usuario:

Peticion original:

![[Pasted image 20230209223014.png]]

Peticion con suplantacion de identidad (conocemos el email de la victima):

![[Pasted image 20230209223108.png]]

podemos representar esta peticion en nuestro navegador:

![[Pasted image 20230209223200.png]]

![[Pasted image 20230209223215.png]]

esta URL la pegamos en nuestro navegador con el **FoxyProxy** a la escucha con Burpsuite, pero el intercept apagado:

![[Pasted image 20230209223326.png]]

![[Pasted image 20230209223441.png]]

y podemos ver que ingresamos con otro usuario.

### Forced OAuth profile linking (CSRF Attack - no state parameter)

En una pagina podemos ver la funcionalidad de login normal y la opcion de agregar una red social (mediante OAuth):

Login normal:

![[Pasted image 20230210065309.png]]

una vez dentro en nuestro perfil:

![[Pasted image 20230210065345.png]]

lo cual conlleva a todo el flujo de oauth. intercepctando todo el flujo vemos que el el primer paso de autoriazcion (PASO 1 del flujo) no predenta el parametro **state** que proteje contra **CSRF Attacks**.

![[Pasted image 20230210065535.png]]

la siguiente peticon en el flujo ya contiene el codigo de autorizacion por partede OAuth:

![[Pasted image 20230210065810.png]]

vamos a copiar esta URL:

![[Pasted image 20230210065836.png]]

vamos a crear el siguiente payload CSRF:

```html
<!--<iframe src="https://YOUR-LAB-ID.web-security-academy.net/oauth-linking?code=STOLEN-CODE"></iframe>-->

<iframe src="https://0ad800e103318c62c0fd87c000f00002.web-security-academy.net/oauth-linking?code=0nileBjVFbBJ2Pu4MvgW89Q7ahJ0wzbhaRX88IKyyly"></iframe>
```

Para enviarlo a la victima, cuando la victima habra el enlace nos dara acceso a su cuenta mediante redes sociales. Entonces primero cerramos sesion comn nuestra propia cuenta y luego intentamos ingresar con redes sociales:

![[Pasted image 20230210070256.png]]

obtendra directamente los valores de la victima:

![[Pasted image 20230210070405.png]]

y estaremos en su cuenta:

![[Pasted image 20230210071210.png]]
ack
### OAuth account hijing via redirect_uri

### Stealing OAuth access tokens via an open redirect

### SSRF via OpenID dynamic client registration
