# Objeto \_\_proto\_\_

## INTRODUCCION

JavaScript usa un modelo de herencia prototípico, que es bastante diferente del modelo basado en clases que usan muchos otros lenguajes.

## ¿Qué es un objeto en JavaScript?

Un objeto de JavaScript es esencialmente solo una colección de pares `key:value` conocidos como "propiedades". Por ejemplo, el siguiente objeto podría representar a un usuario:

```javascript
const user =  {
    username: "wiener",
    userId: 01234,
    isAdmin: false
}
```

Puede acceder a las propiedades de un objeto utilizando la notación de puntos o la notación de corchetes para hacer referencia a sus respectivas claves:

```javascript
user.username // "wiener" 
user['userId'] // 01234
```

Además de datos, las propiedades también pueden contener funciones ejecutables:

```javascript
const user =  {
    username: "wiener",
    userId: 01234,
    exampleMethod: function(){
        // do something
    }
}
```

## ¿Qué es un prototipo en JavaScript?

**Cada objeto en JavaScript está vinculado a otro objeto de algún tipo, conocido como su prototipo (__proto__).** De forma predeterminada, JavaScript asigna automáticamente a los nuevos objetos uno de sus prototipos integrados. Por ejemplo, a las cadenas se les asigna automáticamente el `String.prototype`. Puede ver algunos ejemplos más de estos prototipos globales a continuación:

```javascript
let myObject = {};
Object.getPrototypeOf(myObject);    // Object.prototype

let myString = "";
Object.getPrototypeOf(myString);    // String.prototype

let myArray = [];
Object.getPrototypeOf(myArray);	    // Array.prototype

let myNumber = 1;
Object.getPrototypeOf(myNumber);    // Number.prototype
```

Esto permite a los desarrolladores crear nuevos objetos que pueden reutilizar las propiedades y métodos de los objetos existentes.Los prototipos integrados proporcionan propiedades y métodos útiles para trabajar con tipos de datos básicos. Por ejemplo, el objeto `String.prototype` tiene un método `toLowerCase()` (en el __proto__). Como resultado, todas las cadenas tienen automáticamente un método listo para usar para convertirlas a minúsculas.

## ¿Cómo funciona la herencia de objetos en JavaScript?

Cada vez que hace referencia a una propiedad de un objeto, el motor de JavaScript primero intenta acceder a esto directamente en el objeto mismo. Si el objeto no tiene una propiedad coincidente, el motor de JavaScript la busca en el prototipo del objeto.

![[Pasted image 20230424223107.png]]

Tenga en cuenta que el prototipo de un objeto es solo otro objeto, que también debe tener su propio prototipo, y así sucesivamente. Como prácticamente todo en JavaScript es un objeto bajo el capó, esta cadena finalmente conduce de regreso al nivel superior `Object.prototype`, cuyo prototipo es simplemente `null`.

![[Pasted image 20230424223158.png]]

Fundamentalmente, los objetos heredan propiedades no solo de su prototipo inmediato, sino de todos los objetos que se encuentran por encima de ellos en la cadena de prototipos. En el ejemplo anterior, esto significa que el objeto `username` tiene acceso a las propiedades y métodos de ambos `String.prototype` y `Object.prototype`.

## Accediendo al prototipo de un objeto usando __proto__

Cada objeto tiene una propiedad especial que puede usar para acceder a su prototipo. Aunque esto no tiene un nombre estandarizado formalmente, `__proto__` es el estándar de facto utilizado por la mayoría de los navegadores. Si está familiarizado con los lenguajes orientados a objetos, esta propiedad sirve como getter y setter para el prototipo del objeto:

```javascript
username.__proto__
username['__proto__']

username.__proto__                        // String.prototype
username.__proto__.__proto__              // Object.prototype
username.__proto__.__proto__.__proto__    // null
```

## Modificación de prototipos

Aunque generalmente se considera una mala práctica, es posible modificar los prototipos integrados de JavaScript como cualquier otro objeto. Esto significa que los desarrolladores pueden personalizar o anular el comportamiento de los métodos integrados e incluso agregar nuevos métodos para realizar operaciones útiles.

```javascript
String.prototype.removeWhitespace = function(){
    // remove leading and trailing whitespace
}
```

**_Gracias a la herencia prototípica, todas las cadenas tendrían acceso a este método:_**

```javascript
let searchTerm = " example "; 
searchTerm.removeWhitespace();
```