# INFORMATION DISCLOUSURE

## Table of contents

- [INFORMATION DISCLOUSURE](#information-disclousure)
  - [INTRODUCCION](#introduccion)
  - [EJEMPLOS](#ejemplos)
  - [EXPLOTACION](#explotacion)
  - [USO DE LOGGER++](#uso-de-logger)
  - [CASOS DE EXPLOTACION](#casos-de-explotacion)
    - [Information disclosure in error messages](#information-disclosure-in-error-messages)
    - [Information disclosure on debug page (Source Code)](#information-disclosure-on-debug-page-source-code)
    - [Source code disclosure via backup files (robots.txt)](#source-code-disclosure-via-backup-files-robotstxt)
    - [Authentication bypass via information disclosure (HTTP TRACE VERB)](#authentication-bypass-via-information-disclosure-http-trace-verb)
    - [Divulgación de información en el historial de control de versiones (.git folder)](#divulgacin-de-informacin-en-el-historial-de-control-de-versiones-git-folder)

## INTRODUCCION

La divulgación de información, también conocida como fuga de información, es cuando un sitio web revela involuntariamente información confidencial a sus usuarios. Dependiendo del contexto, los sitios web pueden filtrar todo tipo de información a un atacante potencial, que incluye:

-   Datos sobre otros usuarios, como nombres de usuario o información financiera
-   Datos comerciales o empresariales sensibles
-   Detalles técnicos sobre el sitio web y su infraestructura

Los peligros de filtrar datos confidenciales de usuarios o empresas son bastante obvios, pero la divulgación de información técnica a veces puede ser igual de grave. Aunque parte de esta información tendrá un uso limitado, puede ser potencialmente un punto de partida para exponer una superficie de ataque adicional, que puede contener otras vulnerabilidades interesantes. El conocimiento que puede recopilar podría incluso proporcionar la pieza faltante del rompecabezas cuando intenta construir ataques complejos de alta gravedad.

## EJEMPLOS

Algunos ejemplos básicos de divulgación de información son los siguientes:

-   Revelar los nombres de los directorios ocultos, su estructura y su contenido a través de una lista `robots.txt` de archivos o directorios
-   Proporcionar acceso a archivos de código fuente a través de copias de seguridad temporales
-   Mención explícita de nombres de columnas o tablas de bases de datos en mensajes de error
-   Exponer innecesariamente información altamente confidencial, como detalles de tarjetas de crédito
-   Claves API codificadas, direcciones IP, credenciales de la base de datos, etc. en el código fuente
-   Insinuar la existencia o ausencia de recursos, nombres de usuario, etc. a través de sutiles diferencias en el comportamiento de la aplicación

## EXPLOTACION

- Consultar direccorios que revelan rutas, como `robots.txt` o `sitemap.xml`.
- Fuzzing de directorios ocultos y archivos. Podemos usar gobuster, wfuzz, ffuf o BurpSuite intruder, podemos realizar analisis de tiempos de respuesta, longitud de contenido, codigo de estado, etc.
- evaluacion de con el complemento de burpsuite **Logger++**
- Provocar errores en el servidor, podemos mandar datos string en campos numericos o cosas similares para provocar un error (500) y ver si divulga informacion.
- Buscar comentarios dentro de la respuesta HTML.
- buscar versiones de las libreria que utiliza la pagina en scripts y .css
- Buscar directorios ocultos de backup `/backup`, `/bk`.
- En otros casos, los desarrolladores pueden olvidarse de deshabilitar varias opciones de depuración en el entorno de producción. Por ejemplo, el método HTTP `TRACE` está diseñado para fines de diagnóstico. Si está habilitado, el servidor web responderá a las solicitudes que utilicen el método `TRACE` haciendo eco en la respuesta de la solicitud exacta que se recibió. Este comportamiento suele ser inofensivo, pero en ocasiones conduce a la divulgación de información, como el nombre de los encabezados de autenticación interna que se pueden agregar a las solicitudes de los proxies inversos.
- Verificar la ruta de control de version `/.git`. Recomiendo ver los commits y buscar diferencias:

```bash
#ver commits
git log

#ver diferencias entre commit1 y commit2
git diff commit1 commit2
```


## USO DE LOGGER++

Descargamos el complemento:

![[Pasted image 20220928163251.png]]

ahora interactuamos con la pagina que estamos testeando, nos aparecera una nueva pestaña donde aparecera todas las solicitudes que realiza la pagina:

![[Pasted image 20220928163510.png]]

podemos reenviar cualquier peticion al repeater o intruder.

Nosotros con el burpsuite vemos solo un GET, pero en realidad hace mas consultas para cargar recursos y demas.

Lo bueno de esta extension es que podemos hacer filtrados:

- damos click derecho en filter:

![[Pasted image 20220928163703.png]]

- podemos elegir filtrar por el **Response** o el **Request**, podemos seleccionar del body, header, etc:

![[Pasted image 20220928163834.png]]

- por ejemplo, podemos realizar el siguiente filtro:

```bash
#CSRF VALIDATION
Request.Method == "POST" AND !(Request.Body CONTAINS "csrf")

#CORS VALIDATION
Response.Headers CONTAINS "Access-Control-Allow-Credentials: true" AND Response.Headers CONTAINS "Access-Control-Allow-Origin: *"
```

- Podemos crear un alias para no tener que escribir el filtro otras vez, en su lugar llamaos al alias con un **#** por delante:

![[Pasted image 20220928164555.png]]

![[Pasted image 20220928164646.png]]

- Podemos colocar colores para que apenas detecte una URL con ese filtro lo pin de otro color:

![[Pasted image 20220928164813.png]]

- Podemos crear cuantos alias querramos.

**FUENTE:** [https://www.youtube.com/watch?v=Cqlgcjt6vgI](https://www.youtube.com/watch?v=Cqlgcjt6vgI)

## CASOS DE EXPLOTACION

### Information disclosure in error messages

si vemos un parametro que controlas de un tipo en especifico, por ejemplo entero, podemos enviarle un valor de otro tipo de dato:

parametro de tipo entero:

![[Pasted image 20230208183413.png]]

podemops enviarle un string u otro tipo de dato:

![[Pasted image 20230208183505.png]]

y causamos un error que divulga informacion.

### Information disclosure on debug page (Source Code) 

Es posible que exista comentarios en el codigo fuente de una pagina:

![[Pasted image 20230208202705.png]]

puede revelar alguna ruta que contenga informacion de mas:

![[Pasted image 20230208202820.png]]

### Source code disclosure via backup files (robots.txt)

Siempre podemos consultar el archivo `robots.txt` en un web server por si acaso, puede revelar informacion de rutas que contengan informacion:

![[Pasted image 20230208203412.png]]

Esa ruta muede mostrarnos informacion confidencial:

![[Pasted image 20230208203453.png]]

![[Pasted image 20230208203530.png]]


### Authentication bypass via information disclosure (HTTP TRACE VERB)

El método HTTP `TRACE` está diseñado para fines de diagnóstico. Si está habilitado, el servidor web responderá a las solicitudes que utilicen el método `TRACE` haciendo eco en la respuesta de la solicitud exacta que se recibió. Este comportamiento suele ser inofensivo, pero en ocasiones conduce a la divulgación de información, como el nombre de los encabezados de autenticación interna que se pueden agregar a las solicitudes de los proxies inversos.

Por ejemplo sabemos de una ruta que lleva al dashboard de administrador `/admin` pero esta restringido para usuarios sin privilegios:

![[Pasted image 20230208204331.png]]

si interceptamos la peticion y cambiamos al vebo `TRACE`:

![[Pasted image 20230208204445.png]]

nos revela un header que contiene una IP y valida la autorizacion, si copiamos ese header, lo enviamos en la peticion GET a `/admin` e indicamos localhost (127.0.0.1) para hacer parecer que la peticion viene del servidor local:

![[Pasted image 20230208204917.png]]

nos da un estado 200 y podemos evadir la autenticacion para entrar al dashboard de administrador:

![[Pasted image 20230208205157.png]]

### Divulgación de información en el historial de control de versiones (.git folder)

Prácticamente todos los sitios web se desarrollan utilizando algún tipo de sistema de control de versiones, como Git. De forma predeterminada, un proyecto de Git almacena todos sus datos de control de versiones en una carpeta llamada `.git`. Ocasionalmente, los sitios web exponen este directorio en el entorno de producción. En este caso, es posible que pueda acceder a él simplemente navegando hasta `/.git`.

![[Pasted image 20230208205733.png]]

podemos usar **GitTools** y ver si en algun commit hay alguna informacion:

- [[git]]

