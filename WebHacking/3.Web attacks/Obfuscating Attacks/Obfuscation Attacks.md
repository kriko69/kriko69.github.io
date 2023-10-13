# Obfuscating attacks using encodings

## Table of contents

- [Obfuscating attacks using encodings](#obfuscating-attacks-using-encodings)
  - [Contexto](#contexto)
  - [Ofuscación a través de la codificación de URL](#ofuscacin-a-travs-de-la-codificacin-de-url)
  - [Ofuscación a través de codificación de URL doble](#ofuscacin-a-travs-de-codificacin-de-url-doble)
  - [Ofuscación mediante codificación HTML](#ofuscacin-mediante-codificacin-html)
    - [Ceros a la izquierda](#ceros-a-la-izquierda)
  - [Ofuscación mediante codificación XML](#ofuscacin-mediante-codificacin-xml)
  - [Ofuscación mediante escape Unicode](#ofuscacin-mediante-escape-unicode)
  - [Ofuscación a través de escape hexadecimal](#ofuscacin-a-travs-de-escape-hexadecimal)
  - [Ofuscación mediante escape octal](#ofuscacin-mediante-escape-octal)
  - [Ofuscación a través de múltiples codificaciones](#ofuscacin-a-travs-de-mltiples-codificaciones)
  - [Ofuscación a través de la función SQL CHAR()](#ofuscacin-a-travs-de-la-funcin-sql-char)

## Contexto

Tanto los clientes como los servidores usan una variedad de codificaciones diferentes para pasar datos entre sistemas. Cuando realmente quieren usar los datos, esto a menudo significa que primero tienen que decodificarlos. La secuencia exacta de pasos de decodificación que se realizan depende del contexto en el que aparecen los datos. **Por ejemplo, un parámetro de consulta normalmente se descodifica como URL en el lado del servidor, mientras que el contenido de texto de un elemento HTML puede estar decodificado en HTML en el lado del cliente.**

**_Al construir un ataque, debe pensar en dónde se inyecta exactamente su carga útil. Si puede inferir cómo se decodifica su entrada en función de este contexto, puede identificar formas alternativas de representar la misma carga útil._**

## Ofuscación a través de la codificación de URL

En las URL, una serie de caracteres reservados tienen un significado especial. Por ejemplo, un ampersand ( `&`) se usa como delimitador para separar los parámetros en la cadena de consulta. El problema es que las entradas basadas en URL pueden contener estos caracteres por una razón diferente. Considere un parámetro que contiene la consulta de búsqueda de un usuario. ¿Qué sucede si el usuario busca algo como "Fish & Chips"?

Cualquier entrada basada en URL se decodifica automáticamente como URL en el lado del servidor antes de que se asigne a las variables relevantes. Esto significa que, en lo que respecta a la mayoría de los servidores, secuencias como `%22`, `%3D `y `%3E` en un parámetro de consulta son sinónimos de caracteres `"`, `<` y `>`  respectivamente. En otras palabras, puede inyectar datos codificados en URL a través de la URL y, por lo general, la aplicación de back-end los interpretará correctamente.

## Ofuscación a través de codificación de URL doble

Por una razón u otra, algunos servidores realizan dos rondas de descodificación de URL en cualquier URL que reciben. Esto no es necesariamente un problema en sí mismo, siempre que cualquier mecanismo de seguridad también decodifique dos veces la entrada al verificarla. De lo contrario, esta discrepancia permite que un atacante pase de contrabando información maliciosa al back-end simplemente codificándola dos veces.

```http
#normal
<img src=x onerror=alert(1)>

#URL encoded
%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E

# double URL Encoded
%253Cimg%2520src%253Dx%2520onerror%253Dalert(1)%253E
```

Como WAF solo decodifica esto una vez, es posible que no pueda identificar que la solicitud es peligrosa. Si el servidor back-end posteriormente decodifica dos veces esta entrada, la carga útil se inyectará con éxito.

## Ofuscación mediante codificación HTML

En los documentos HTML, ciertos caracteres deben escaparse o codificarse para evitar que el navegador los interprete incorrectamente como parte del marcado. Esto se logra sustituyendo los caracteres ofensivos con una referencia, con el prefijo de un ampersand y terminado con un punto y coma.

En muchos casos, se puede utilizar un nombre para la referencia. Por ejemplo, la secuencia `&colon;`representa un carácter de dos puntos.

![[Pasted image 20230227141918.png]]

Alternativamente, la referencia se puede proporcionar utilizando el punto de código decimal o hexadecimal del carácter, en este caso, `&#58;`y `&#x3a;` respectivamente.

![[Pasted image 20230227141958.png]]

![[Pasted image 20230227142013.png]]

**_En ubicaciones específicas dentro del HTML, como el contenido de texto de un elemento o el valor de un atributo, los navegadores descodificarán automáticamente estas referencias cuando analicen el documento. Al inyectar dentro de una ubicación de este tipo, ocasionalmente puede aprovechar esto para ofuscar cargas útiles para ataques del lado del cliente, ocultándolas de cualquier defensa del lado del servidor que esté en su lugar._**

Ejemplo:

Si observa detenidamente la carga útil XSS de nuestro ejemplo anterior, observe que la carga útil se inyecta dentro de un atributo HTML, es decir, el controlador de eventos `onerror`. Si las comprobaciones del lado del servidor buscan explícitamente la carga útil`alert()`, es posible que no detecten esto si codificas en HTML uno o más de los caracteres:

```html
<img src=x onerror="&#x61;lert(1)">
```

Cuando el navegador muestra la página, decodificará y ejecutará la carga útil inyectada.

### Ceros a la izquierda

Curiosamente, al usar codificación HTML de estilo decimal o hexadecimal, puede incluir opcionalmente un número arbitrario de ceros a la izquierda en los puntos de código. Algunos WAF y otros filtros de entrada no tienen esto en cuenta adecuadamente.

Si su carga útil aún se bloquea después de codificarla en HTML, es posible que pueda evadir el filtro simplemente anteponiendo los puntos de código con algunos ceros:

```html
<a href="javascript&#00000000000058;alert(1)">Click me</a>
```

## Ofuscación mediante codificación XML

XML está estrechamente relacionado con HTML y también admite la codificación de caracteres utilizando las mismas secuencias de escape numéricas. Esto le permite incluir caracteres especiales en el contenido de texto de los elementos sin romper la sintaxis, lo que puede resultar útil al probar XSS a través de una entrada basada en XML, por ejemplo.

```xml
<stockCheck>
    <productId>
        123
    </productId>
    <storeId>
        999 &#x53;ELECT * FROM information_schema.tables
    </storeId>
</stockCheck>
```

![[Pasted image 20230227142538.png]]

**_Incluso si no necesita codificar ningún carácter especial para evitar errores de sintaxis, puede aprovechar este comportamiento para ofuscar las cargas útiles de la misma manera que lo hace con la codificación HTML._**

## Ofuscación mediante escape Unicode

Las secuencias de escape Unicode constan del prefijo `\u` seguido del código hexadecimal de cuatro dígitos del carácter. Por ejemplo, `\u003a` representa dos puntos. ES6 también es compatible con una nueva forma de escape Unicode mediante llaves: `\u{3a}`.

![[Pasted image 20230227145154.png]]

Al analizar cadenas, la mayoría de los lenguajes de programación decodifican estos escapes Unicode. Esto incluye el motor de JavaScript utilizado por los navegadores. Al inyectar en un contexto de cadena, puede ofuscar las cargas útiles del lado del cliente usando unicode

![[Pasted image 20230227145417.png]]

Por ejemplo, supongamos que está tratando de explotar [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based) donde su entrada se pasa al sink `eval()` como una cadena. Si sus intentos iniciales están bloqueados, intente escapar de uno de los personajes de la siguiente manera:

```javascript
eval("\u0061lert(1)")
```

**_Como esto permanecerá codificado en el lado del servidor, puede pasar desapercibido hasta que el navegador lo decodifique nuevamente._**

También vale la pena señalar que los escapes Unicode de estilo ES6 también permiten ceros iniciales opcionales, por lo que algunos WAF pueden engañarse fácilmente usando la misma técnica que usamos para las codificaciones HTML. 

Por ejemplo:

```html
<a href="javascript\u{0000000003a}alert(1)">Click me</a>
```

## Ofuscación a través de escape hexadecimal

Otra opción cuando se inyecta en un contexto de cadena es usar escapes hexadecimales, que representan caracteres usando su punto de código hexadecimal, con el prefijo `\x`. Por ejemplo, la letra minúscula `a` está representada por `\x61`.

```javascript
eval("\x61lert")
```

![[Pasted image 20230227145724.png]]

>[!success]
>Tenga en cuenta que a veces también puede ofuscar sentencias SQL de manera similar utilizando el prefijo `0x`. Por ejemplo, `0x53454c454354` puede decodificarse para formar la `SELECT` palabra clave.

## Ofuscación mediante escape octal

El escape octal funciona prácticamente de la misma manera que el escape hexadecimal, excepto que las referencias de caracteres usan un sistema de numeración de base 8 en lugar de base 16. Estos tienen el prefijo de una barra invertida independiente, lo que significa que la letra minúscula `a` está representada por `\141`.

![[Pasted image 20230227145957.png]]

```javascript
eval("\141lert(1)")
```

## Ofuscación a través de múltiples codificaciones

>[!success]
>**Es importante tener en cuenta que puede combinar codificaciones para ocultar sus cargas útiles detrás de múltiples capas de ofuscación.**

ejemplo:

```html
<a href="javascript:&bsol;u0061lert(1)">Click me</a>
```

Los navegadores primero descodificarán HTML, `&bsol;` lo que dará como resultado una barra invertida. Esto tiene el efecto de convertir los caracteres arbitrarios en el escape Unicode `u0061`  ->  `\u0061`:

```html
<a href="javascript:\u0061lert(1)">Click me</a>
```

Luego, esto se decodifica aún más para formar una carga útil XSS funcional:

```html
<a href="javascript:alert(1)">Click me</a>
```

**_Claramente, para inyectar con éxito una carga útil de esta manera, necesita una comprensión sólida de qué decodificación se realiza en su entrada y en qué orden._**

## Ofuscación a través de la función SQL CHAR()

Aunque no es estrictamente una forma de codificación, en algunos casos, puede ofuscar sus ataques de inyección SQL usando la función `CHAR()`. Esto acepta un solo punto de código decimal o hexadecimal y devuelve el carácter coincidente. Los códigos hexadecimales deben tener el prefijo `0x`. 

Por ejemplo, ambos `CHAR(83)`y `CHAR(0x53)`devuelven la letra mayúscula `S`.

Al concatenar los valores devueltos, puede usar este enfoque para ofuscar las palabras clave bloqueadas. Por ejemplo, incluso si `SELECT` está en la lista negra, la siguiente inyección inicialmente parece inofensiva:

```sql
CHAR(83)+CHAR(69)+CHAR(76)+CHAR(69)+CHAR(67)+CHAR(84)
```

Sin embargo, cuando la aplicación lo procesa como SQL, construirá dinámicamente la palabra clave `SELECT` y ejecutará la consulta inyectada.

