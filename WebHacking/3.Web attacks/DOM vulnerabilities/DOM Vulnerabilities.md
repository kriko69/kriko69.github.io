# DOM Vulnerabilities

## Table of contents

- [DOM Vulnerabilities](#dom-vulnerabilities)
  - [¿Qué es el DOM?](#qu-es-el-dom)
  - [XSS BASADOS EN DOM](#xss-basados-en-dom)
    - [SOURCE](#source)
    - [SINK](#sink)
    - [EJEMPLO](#ejemplo)

## ¿Qué es el DOM?

El modelo de objeto de documento (DOM) es la representación jerárquica de un navegador web de los elementos de la página. Los sitios web pueden usar JavaScript para manipular los nodos y objetos del DOM, así como sus propiedades.

```html
<!--ESQUELETO DEL DOM-->
<html>
	<head>
		...
	</head>
	<body>
		...
	</body>
</html>
```

En otras palabras, cuando un navegador recibe una página para cargar, analizará o diseccionará la estructura de la página y separará los diferentes elementos de la página en una estructura similar a un árbol con cada elemento y atributo anidado en su ubicación respectiva.

![[Pasted image 20230202144843.png]]

![[Pasted image 20230202144856.png]]

el elemento raíz "\<html\>" está anidado bajo el nodo "ocument" que es la página misma, y ​​todos los demás elementos y atributos siguen su ejemplo. Dado que las páginas HTML son estáticas, lo que significa que la estructura de la página se presenta exactamente como está almacenada en el servidor sin modificaciones, generalmente se incluye JavaScript para hacer que las páginas web sean más interactivas. A medida que el navegador analiza la página para desarrollar el árbol DOM, cuando llega a la parte del código que contiene JavaScript, el navegador representará el código y producirá una salida basada en la lógica.

## XSS BASADOS EN DOM

Recuerde que el navegador del usuario maneja y procesa el código JavaScript, ya sea mientras se analiza la página o incluso cuando el usuario interactúa con la página.

Las secuencias de comandos entre sitios basadas en DOM se producen cuando el código JavaScript acepta la entrada de un usuario (**SOURCE**) y pasa esa entrada a otra función que muestra los resultados en la página (**SINK**) de forma no segura.

La clave a tener en cuenta aquí es que las solicitudes/ataques basados ​​en DOM no son persistentes ni manejados por el servidor, sino por el navegador del usuario.

### SOURCE

Una función de origen es cualquier propiedad o función de JS que acepta la entrada del usuario desde algún lugar de la página. Un ejemplo de una fuente es la propiedad **location.search** porque lee la entrada de la cadena de consulta.

Estas propiedades son controladas por el usuario a pesar de que no haya ningun campo para ingresar un dato en la pagina web, por ejemplo:

**location.search**: toma el argumento GET de la URL que consultamos:

![[Pasted image 20230202145817.png]]

otras propiedades parecidas:

- **document.URL**: retorna la URL que consultamos.
- **document.documentURI**: retorna la URL que consultamos.
- **document.URLUnencoded**: retorna la URL que consultamos en formato URLEncoded si existe.
- **document.baseURI**: retorna la URL que consultamos.
- **location.search**: toma el argumento GET de la URL que consultamos
- **document.cookie**: devuelve las cookies.
- **document.referrer**: devuelve el URI de la página que se vinculó a esta página. (referer)
- **location**: son atributos de la URL que consultamos.

![[Pasted image 20230202150237.png]]

- **window.name**: retorna el nombre de la ventana.
- **window.origin**: retorna la URL
- **history.pushState**
- **history.replaceState**
- **localStorage**: retorna los valores del local storage.
- **sessionStorage**: retorna los valores del session storage.

### SINK

Un **sink** es una función de JavaScript potencialmente peligrosa que puede causar efectos no deseados si se le pasan datos controlados por el atacante. **Básicamente, si la función devuelve la entrada a la pantalla como salida sin controles de seguridad, se considera un SINK.** 

Un ejemplo de esto sería la propiedad "**innerHTML**" utilizada anteriormente, ya que cambia el contenido de la página HTML a lo que sea que se le dé.

- **document.write()**
- **document.writeln()**
- **document.domain**
- **element.innerHTML**
- **element.outerHTML**
- **element.insertAdjacentHTML**
- **element.onevent**
- **eval()**

Las siguientes funciones de jQuery también son sink que pueden conducir a vulnerabilidades DOM-XSS:

- add()
- after() 
- append() 
- animate()
- insertAfter() 
- insertBefore()
- before()
- html() 
- prepend() 
- replaceAll()
- replaceWith()
- wrap() 
- wrapInner()
- wrapAll() 
- has() 
- constructor() 
- init()
- index()
- jQuery.parseHTML() 
- $.parseHTML()

###  EJEMPLO

Fundamentalmente, las vulnerabilidades basadas en DOM surgen cuando un sitio web pasa datos de un **SOURCE** a un **SINK**, que luego maneja los datos de manera insegura en el contexto de la sesión del cliente.

**Open redirect basado en DOM**

La fuente más común es la URL, a la que normalmente se accede con el objeto `location`. Un atacante puede construir un enlace para enviar a una víctima a una página vulnerable con una carga útil en la cadena de consulta y fragmentar partes de la URL. Considere el siguiente código:

```javascript
goto = location.hash.slice(1); 
if (goto.startsWith('https:')) 
{   
	location = goto; 
}
```

Esto es vulnerable a la redirección abierta basada en DOM porque el source **location.hash** se maneja de manera insegura. Si la URL contiene un fragmento hash que comienza con https:, este código extrae el valor de la propiedad **location.hash** y lo establece como propiedad **location** de **window**. Un atacante podría aprovechar esta vulnerabilidad construyendo la siguiente URL:

```bash
https://www.innocent-website.com/example#https://www.evil-user.net
```

Cuando una víctima visita esta URL, JavaScript establece el valor de la propiedad **location** en https://www.evil-user.net, lo que automáticamente redirige a la víctima al sitio malicioso. Este comportamiento podría explotarse fácilmente para construir un ataque de phishing, por ejemplo.




