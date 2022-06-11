---
title: AWK Command (Output Manipulation)
published: true
---

# AWK

## Table of contents

- [AWK](#awk)
  - [TRATAMIENTO DE CONTENIDO DE ARCHIVOS](#tratamiento-de-contenido-de-archivos)
  - [VARIABLES DE AWK](#variables-de-awk)
  - [VARIABLES DEFINIDAS POR EL USUARIO](#variables-definidas-por-el-usuario)
  - [COMANDOS ESTRUCTURADOS](#comandos-estructurados)
  - [DANDO FORMATO A LA IMPRESION](#dando-formato-a-la-impresion)
  - [FUNCIONES MATEMATICAS Y DE CADENAS](#funciones-matematicas-y-de-cadenas)
  - [FUNCIONES](#funciones)

## TRATAMIENTO DE CONTENIDO DE ARCHIVOS

al hacer un 'cat' de un archivo es posible filtrar su contenido de diersas formas
haciendo un 'grep' podemos filtrar segun una coincidencia o patron
pero haciendo uso de 'awk' podemos especificar un delimitador de manera mas sencilla
y hacer otras operaciones de utilidad, ademas podemos usar tanto 'grep' como 'awk' al mismo tiempo.

## VARIABLES DE AWK

* $0 = toda la linea
* $1 - $n = campos especificos segun un delimitador
* -F[separador] = delimitador. -F: este ejemplo delimita ':'
* FS="separador" = delimitador FS=":" este ejemplo delimita ':'
* OFS="SEPARADOR" = delimitador del OUTPUT

```
cat passwd | grep root | awk '{print $1}' FS=":"

//output: root
```

Filtra el archivo 'passwd' hace un grep a root y usa el delimitador ':' para imprimir el primer campo
Si se va a usar FS se tiene que poner al final y fuera de las comillas simples

**uso de -F**

Mismo ejemplo:

```
cat passwd | grep root | awk -F: '{print $1}'

//output: root
```

Para usar -F se debe colocar al comienzo de la sentencia awk y fuera delas comillas simples

**PRE Y POST PROCESAMIENTO**

Se refiere a algo que quieres hacer antess de una sentencia awk. normalmente aqui puedes definir delimitadores.

```
cat passwd | awk 'BEGIN {FS=":"; OFS="-"} {print $1,$7}'
```

esto imprime el campo 1 y 7 de larchivo 'passwd' usando como delimitador ':' y delimitador del resultado '-'
los delimitadores van dentro de la sentencia BEGIN{} usando ';' para separa opciones.
esta sentencia se ejecura antes que el {print...}
se puede usar para poner un encabezado a la salida o algo similar

Para POST procesamiento se use END{} despues de la sentencia {print...}
este se ejecuta despues de la sentencia awk {print...}

para escribir algo dentro de las sentencias BEGIN{} o END{} se usa print ""


```
BEGIN{print "encabezado"} o END{print "footer"}
```


* FIELDWIDTHS = especifica lacantidad de caracteres de cada campo, es como un delimitador pero con longitud de cadena.
* RS = especifica el separador por registros, es decir por cada fila.
* ORS = especifica el separador por registros del OUTPUT.

Supongamos este contenido:

```
1235.96521

927-8.3652

36257.8157
```

```
cat data.txt | awk 'BEGIN{FIELDWIDTHS="3 4 3"} {print $1,$2,$3}' 

OUTPUT:

123 5.96 521
927 -8.3 652
362 57.8 157
```

con FIELDWITHDS definimos de toda la linea la longitud de las subcadenas.

```
cat data.txt | awk 'BEGIN{RS="";ORS="\n---\n"} {print $0}' 

OUTPUT:

1235.96521

---

927-8.3652

---

36257.8157
```

* FILENAME = devuele el nombre del archivo que estamos tratando.
* NF = Cuenta los campos de la línea que está siendo procesada. Si se usa como variable ($NF) devuelve el ultimo campo
* NR = Retorna el numero de registro procesado. 
* IGNORECASE = Le dice al programa que ignore las diferencias entre mayúsculas y minúsculas.

```
...{print $1,$NF,"FILENAME="NOMBRE,"NR="NR,IGNORECASE}...
```

## VARIABLES DEFINIDAS POR EL USUARIO

```
awk 'BEGIN{saludo="hola chris" print test}'
```

## COMANDOS ESTRUCTURADOS

```
...'if ($1 >10) print $1}'...

ó

...'{if ($1 >10){
        x=$1*5
        print x
    }
   }'...

se le puiede agreegarun else{...}
tambien while(...>...){...}
usar sentencias como break o continue
for (var=1;var<5;var++){...}

```

## DANDO FORMATO A LA IMPRESION

Junto a awk podemos usar print o printf, este ultimo se usa para dar formato
a la impresion:

Sintaxis:

```
'...%[modificador]', variable
```

Modificadores:

* c = Imprime salidas numéricas como una cadena de caracteres (string).
* d = Imprime un valor entero.
* e = Imprime números con notación científica.
* f = Imprime valores numéricos con decimales (float).
* o = Imprime valores en notación octal.
* s = Imprime una cadena de texto.

Ejemplo:

```
... 'BEGIN{
    x =100 * 100
    printf "El resultado es %e", x
}'

```

## FUNCIONES MATEMATICAS Y DE CADENAS

* sin(x)
* cos(x)
* sqrt(x)
* exp(x)
* log(x)
* rand()
* toupper(x)
* tolower(x)

Existen mas, consultar con el manual de awk en linea.

## FUNCIONES

```

awk '

function myfunc()

{

printf "The user %s has home path at %s\n", $1,$6

}

BEGIN{FS=":"}

{

myfunc()

}' /etc/passwd

```


