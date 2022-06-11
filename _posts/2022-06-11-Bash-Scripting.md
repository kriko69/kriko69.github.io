---
title: Bash Scripting
published: true
---

# BASH SCRIPTING (COMANDOS UTILES PARA SCRIPTS)

## Table of contents

- [BASH SCRIPTING (COMANDOS UTILES PARA SCRIPTS)](#bash-scripting-comandos-utiles-para-scripts)
  - [seq (para la creacion de secuencias)](#seq-para-la-creacion-de-secuencias)
  - [tr para transformar una cadena](#tr-para-transformar-una-cadena)
    - [archivo: data = hola chris](#archivo-data--hola-chris)
  - [bc (basic calculator)](#bc-basic-calculator)
  - [VARIABLES](#variables)
  - [EXPRESIONES NUMERICASEXPRESIONES NUMERICAS](#expresiones-numericasexpresiones-numericas)
    - [siempre enre (())](#siempre-enre-)
  - [VARIABLES ESPECIALES](#variables-especiales)
  - [ENTRADA DE DATOS](#entrada-de-datos)
  - [COMENTARIOS](#comentarios)
  - [QUOTES](#quotes)
  - [DEPURACION](#depuracion)
  - [CODIGOS DE EJECUCION](#codigos-de-ejecucion)
  - [grep (filtrado)](#grep-filtrado)
  - [FIND (buscar)](#find-buscar)
  - [SORT (ordenar archivos o su contenido)](#sort-ordenar-archivos-o-su-contenido)
  - [HEAD](#head)
  - [TAIL (lo contrario de head)](#tail-lo-contrario-de-head)
  - [WC](#wc)
- [BASH SCRIPTING 2 (CONDICIONALES)](#bash-scripting-2-condicionales)
  - [COMPARACION DE NUMEROS](#comparacion-de-numeros)
  - [COMPARACION DE CADENAS](#comparacion-de-cadenas)
  - [CONDIICIONALES IF - ELSE](#condiicionales-if---else)
- [BASH SCRIPTING (BUCLES)](#bash-scripting-bucles)
  - [CICLOS](#ciclos)
  - [WHILE](#while)
- [BASH SCRIPTING (ARRAYS Y SHIFT)](#bash-scripting-arrays-y-shift)
  - [ARRAYS](#arrays)
  - [USO DE SHIFT](#uso-de-shift)
- [BASH SCRIPTING (SWITCH CASE Y FUNCIONES)](#bash-scripting-switch-case-y-funciones)
  - [switch case](#switch-case)
    - [Multiples string como opcion](#multiples-string-como-opcion)
    - [usar patrones en los casos de un switch](#usar-patrones-en-los-casos-de-un-switch)
  - [FUNCIONES](#funciones)
- [BASH SCRIPTING (AVANZADO)](#bash-scripting-avanzado)
  - [controlar el evento ctrl + c](#controlar-el-evento-ctrl--c)
  - [SLEEP](#sleep)
  - [verificar si es un usuario root](#verificar-si-es-un-usuario-root)
  - [colores de texto de la terminal](#colores-de-texto-de-la-terminal)
  - [*while optimo](#while-optimo)
  - [DECLARE](#declare)
  - [SUBCADENAS](#subcadenas)

## seq (para la creacion de secuencias)

```
seq 10 -> 1 2 3 4 5 6 7 8 9 10
el inicio siempre sera 1

seq 3 10 -> 3 4 5 6 7 8 9 10
definiendo inicio y fin

seq 3 2 10 -> 3 5 7 9
definiendo inicio, salto y fin

seq -f "usuario %g2:" 2 5  -> usuario 2: usuario 3: usuario 4: usuario 5: 
-f de formato y con %g2 ponemos una de las anteriores secuencias en un texto

seq -f "hola %g2" 2 2 10

seq -s ":" 2 5 -> 2:3:4:5
-s de separador

seq -w 2 5 -> 02 03 04 05
agrega un 0 al inicio
```

## tr para transformar una cadena

### archivo: data = hola chris

```
cat data | tr "[a-z]" "[A-Z]" -> HOLA CHRIS
convierte a mayusculas

 o tambien esto  hace lo mismo

cat data | tr "[:lower:]" "[:upper:]"

reemplazar espacio por tab

cat data | tr [:space:] "\t"

reemplazar {} por ()

cat data | tr "{}" "()"

para eliminar un caracter repetido en este caso espacios ej: "hola     mundo" => " hola mundo"

cat data | tr -s [:space:] ""

eliminar un terminado carater

cat data | tr -d 'h'  --> elimina la o las h

se puede eliminar los digitos

cat data | tr -d [:digits:]

para eliminar todo menos los digitos (lo contrario)

cat data | tr -cd [:digits:]
```

## bc (basic calculator)

```
echo "9 / 2" | bc => 4

echo "9/ 2" | bc -l => 4.5

```

## VARIABLES
 
[http://www.compciv.org/topics/bash/variables-and-substitution/](http://www.compciv.org/topics/bash/variables-and-substitution/)

```
numero=10
echo $numero
echo "mi numero es $numero"

soy=$(whoami)
echo "hola ${soy}" => puedes poner $soy o ${soy} en si es mejor el con llaves para concatenar con otras cadenas

para hacer funciones aritmeticas

$numero1=30
$numero2=13
$suma=$(($numero1+$numero2))  => se usa $(())

echo $suma
echo $(($numero1+$numero2))

```

## EXPRESIONES NUMERICASEXPRESIONES NUMERICAS

### siempre enre (())

```
## Post-increment example 
$ var=10
$ echo $((var++))  ## First print 10 then increase value by 1
 
## Pre-increment example
$ var=10
$ echo $((++var))  ## First increase value by 1 then print 11 

## Post-decrement example 
$ var=10
$ echo $((var--))  ## First print 10 then decrease value by 1
 
## Pre-decrement example
$ var=10
$ echo $((--var))  ## First decrease value by 1 then print 9
```

## VARIABLES ESPECIALES

```
$0 -> es el nombre del script
$1 - $9 -> argumentos del script, a partir del decimo argumento se usa llaves, ${10},${11}, etc.
$? -> verificamos el codigo de ejecucion de un comando
$! -> devuelve el ultimo atributo de un comando
$# -> devuelve el numero de argumentos del script
$@ -> devuelve todos los argumentos del script

args=("$@")
echo "First->"  ${args[0]}

$$ -> devuelve el PID del proceso del script
$* -> retorna todos los argumentos del script pero con comillas dobles

!! -> devuelve todo el ultimo comando escrito
```

## ENTRADA DE DATOS

```
read variable -> lee una variable con salto de linea
read -p "mensaje" variable -> lee una variable en una sola linea
read -sp "mensaje" variable -> lee una variable en una sola linea pero no se muestra que se escribe (ideal para contrraseñas)

con echo -e "" -> puede poner las variables dentro de las comillas con el simbolo $ ademas de usar \n o \t y el echo lo interpretara

```

## COMENTARIOS

```
#        -> para una sola linea

<<COMMENTS
muchas lineas para comentar
COMMENTS

: '
muchas lineas para comantar
'
```

## QUOTES

```
"" -> para cadenas simples  y para imprimir una variable
'' -> para cadenas simples

nombre="chris"
echo "$nombre" -> devuelve chris
echo '$nombre' -> devuelve $nombre"" -> para cadenas simples  y para imprimir una variable
'' -> para cadenas simples

nombre="chris"
echo "$nombre" -> devuelve chris
echo '$nombre' -> devuelve $nombre
```

## DEPURACION

```
bash -xv mi_script.sh
```

## CODIGOS DE EJECUCION

```
0 -> denota que el comando tiene una salida valida
1 -> la salida tiene un error (va desde el numero 1 - 255)
```

## grep (filtrado)

```
grep -E "hola|adios" => filtra por hola y por adios

o sino

grep -e "hola" -e "adios" data.txt

existe muchas variantes de grep, una de ellas es:

egrep => es igual a grep -E

buscar letras consecutivas

grep -E o\{2}  => (sin comillas) busca que haya 2 letras 'o' consecutivas

busqueda recursiva

1.- nos situamos en un directorio

...$ grep -r "hola"

```

## FIND (buscar)

```
find [ruta] -opcion parametro

find . -name data.txt => busqueda por nombre

find /home -name data.txt => en la ruta /home 

find . -iname data.txt => busqueda por nombre sin case sensitive (data, Data)

find . -type d/f -name data => busqueda por tipo 'd' directorio y 'f' file

find . -type f -name "*.php" => buscar todos los archivos de una extension

find . -type f -perm 0775 -print => encontrar archivos con un cierto permiso

find . -type f ! -perm 777  => encontrar archivos sin un determinado permiso

find . -perm /u=r => permiso de usuario para lectura

find . -name data.txt -exec rm -rf {} \; => buscar un archivo y eliminarlo (o ejecutar algun comando)

find . -empty => buscar archivos vacios

```

## SORT (ordenar archivos o su contenido)

```
sort data.txt => ordena el archivo data

sort -ordenado.txt data.txt => se define la salida

sort data.txt > ordenado.txt => se define una salida

sort -r data.txt =>ordena reversamente

sort -n numeros.txt => ordena un archivo a nivel numerico

sort -nr numeros.txt => orena reversamente numeros

sort -k 2 data.txt => ordena segun la segunda columna
EJ:

asd 100			asd 100
ad 400		=>	de 300
de 300			ad 400

sort -u data.txt => ordena y elimina los duplicos (-uniq)

*Si lo ponemos con | ordena la salida del comando anterior

cat data.txt | sort

```

## HEAD 

```
head -n 5 data.txt => muestra solo las 5 primeras lineas

head -c 5 data.txt => muestra los 5 primeros caracteres
```

## TAIL (lo contrario de head)

```
tail -n 5 data.txt => muestra solo las 5 ultimas lineas

tail -c 5 data.txt => muestra los 5 ultimos caracteres
```

## WC

contar lineas, ficheros y directorios

```
ls | wc -l  ->contamos cuantos directorios y ficheros hay

con ls -a podemos contar los ocultos mas

fichero.txt | wc -l  -> contamos las lineas
fichero.txt | wc -m  -> contamos los caracteres

```

# BASH SCRIPTING 2 (CONDICIONALES)

## COMPARACION DE NUMEROS

**siempre entre doble parentesis ((expresion))**

```
if ((10 > 5)); then (retorna True)

((n1 == n2))    ## n1 es igual que n2
((n1 != n2))    ## n1 es diferente que n2
((n1 > n2))     ## es mayor que n2
((n1 >= n2))    ## n1 es mayor o igual que n2
((n1 < n2))     ## n1 es menor que n2
((n1 <= n2))    ## n1 es menor o igual que n2

```

## COMPARACION DE CADENAS

**siempre entre corchetes [expresion]**

```
if [ "$str1" == "$str2" ]     # True si es igual
if [ "$str1" != "$str2" ]     # True si no es igual
```

## CONDIICIONALES IF - ELSE

**if**

```
if [ $myvar -gt 10 ]
then
    echo "Value is greater than 10"
fi
```

**if - else**

```
if [ $myvar -gt 10 ]
then
    echo "OK"
else
    echo "Not OK"
fi
```

**if - elif - else**

```
if [ $marks -ge 80 ]
then
    echo "Very Good"
 
elif [ $marks -ge 50 ]
then
    echo "Good"
 
elif [ $marks -ge 33 ]
then
    echo "Just Satisfactory"
else
    echo "Not OK"
fi
```

**if anidado**

```
if [ $i -gt $j ]
then
    if [ $i -gt $k ]
    then
        echo "i is greatest"
    else
        echo "k is greatest"
    fi
else
    if [ $j -gt $k ]
    then
        echo "j is greatest"
    else
 echo "k is greatest"
    fi
fi
```

**AND - OR**

```
&& condicional y
|| condicional o
```

**OPERADORES DE UN CONDICIONAL**

```
-n VAR --> True si la longitud de VAR es maor a 0.
-z VAR --> True si VAR es vacio.
STRING1 = STRING2 --> True si STRING1 y STRING2 son iguales.
STRING1 != STRING2 --> True si STRING1 y STRING2 son no iguales.
INTEGER1 -eq INTEGER2 --> True si INTEGER1 y INTEGER2 son iguales.
INTEGER1 -gt INTEGER2 --> True si INTEGER1 es mayor que INTEGER2.
INTEGER1 -lt INTEGER2 --> True si INTEGER1 es menor que INTEGER2.
INTEGER1 -ge INTEGER2 --> True si INTEGER1 es igual o mayor que INTEGER2.
INTEGER1 -le INTEGER2 --> True si INTEGER1 es igual o menor que INTEGER2.
-h o -L FILE --> True si el FILE existe y es u enlace simbolico.
-r FILE --> True si el FILE existe y es readable.
-w FILE --> True si el FILE existe y es writable.
-x FILE --> True si el FILE existe y es executable.
-d FILE --> True si el FILE existe y es un directorio.
-e FILE --> True si el FILE existe
-f FILE --> True si el FILE existe y es un archivo (tipo archivo).
-N FILE --> True si el FILE fue modificado despues de su ultima vez leida.
-O FILE --> True si el usuario actual es dueño del archivo FILE.
-G FILE --> si el id de grupo del archio FILE coincide con el del usuario actual.
-s FILE --> True siel artchivo FILE tiene un tamaño mayor a 0.

podemos usar negacion (!) al comienzo: !-e --> no existe

También tenemos algunas pruebas de comparación de archivos:

-nt prueba si el archivo de la izquierda es mas nuevo que el archivo de la derecha
-ot prueba si el archivo de la izquierda es mas antiguo que el archivo de la derecha
-ef   prueba si ambos archivos son enlaces duros al mismo archivo

if [ “$file” -nt “/home/myuser/myfile”]

if [ “$file” -ef “$otherfile” ]


```

# BASH SCRIPTING (BUCLES)


## CICLOS

**sintaxis**

```
for VARIABLE in PARAM1 PARAM2 PARAM3
do
  // scope of for loop
done
```

**Ejemplos**

```
for i in 1 2 3 4 5
do
   echo "$i"
done

for i in {1..5}  -> se puede usar el comando $(seq) entre parentesis tambien
do
   echo "$i"
done

for OUTPUT in $(Linux-Or-Unix-Command-Here)
do
	command1 on $OUTPUT
	command2 on $OUTPUT
	commandN
done

for day in SUN MON TUE WED THU FRI SUN
do
   echo "$day"
done

```


**al estilo del lenguaje C**

```
for ((i=1; i<=10; i++))
do
  echo "$i"
done
```

**trabajando con archivos**

```
for fname in *
do
  ls -l $fname
done
```

**for infinito**

```
#!/bin/bash
for (( ; ; ))
do
   echo "infinite loops [ hit CTRL+C to stop]"
done

sentencia break

for file in /etc/*
do
	if [ "${file}" == "/etc/resolv.conf" ]
	then
		countNameservers=$(grep -c nameserver /etc/resolv.conf)
		echo "Total  ${countNameservers} nameservers defined in ${file}"
		break
	fi
done

sentencia continue

FILES="$@"
for f in $FILES
do
        # if .bak backup file exists, read next file
	if [ -f ${f}.bak ]
	then
		echo "Skiping $f file..."
		continue  # read next file and skip the cp command
	fi
        # we are here means no backup file exists, just use cp command to copy file
	/bin/cp $f $f.bak
done

lleyendo parametros

for i in $@
do
    echo "Script arg is $i"
done

```

**trabajando con arrays**

```
DB_AWS_ZONE=('us-east-2a' 'us-west-1a' 'eu-central-1a')
 
for zone in "${DB_AWS_ZONE[@]}"
do
  echo "Creating rds (DB) server in $zone, please wait ..."
  aws rds create-db-instance \
  --availability-zone "$zone"
  --allocated-storage 20 --db-instance-class db.m1.small \
  --db-instance-identifier test-instance \
  --engine mariadb \
  --master-username my_user_name \
  --master-user-password my_password_here
done
```

## WHILE

**sintaxis**

```
while [condition]
do
  //programme to execute
done
```

**ejemplos**

```
num=1
while [ $num -le 5 ]
do
   echo "$num"
   let num++
done

ciclo infinito

while true
do
  echo "Press CTRL+C to Exit"
done
```

**al estilo del lenguaje C**

```
num=1
while((num <= 5))
do
   echo $num
   let num++
done

```
**leyendo un archivo**

```
while read myvar
do
   echo $myvar
done < /tmp/filename.txt
```

# BASH SCRIPTING (ARRAYS Y SHIFT)

## ARRAYS

**declaracion**

```
#!/bin/bash
numeros=(1 2 3 4 5)

declare -a numeros=(1 2 3 4 5)
```

**mostrar todos los elementos del array**

```
echo ${numeros[@]}

echo ${numeros[*]}
```

**accedder a elementos del array**

```
echo ${numeros[0]}

echo ${numeros[10]}

numeros=(1 2 3 4 5)
numeros[1]="Cambiado"
echo ${numeros[*]}

```

**agregar nueo elementos**

```
en base a un iindice inexistente agregamos un elemento

numeros=(1 2 3 4 5)
numeros[5]=6
echo ${numeros[*]}

```
**ver los indices de los elementos**

```
echo ${!numeros[*]}

numeros=(1 2 3 4 5)
numeros[10]=6
echo "Contenido del array:"
echo ${numeros[*]}
echo "Índices del array:"
echo ${!numeros[*]}

OUTPUT:

Contenido del array:
1 2 3 4 5 6
Índices del array:
0 1 2 3 4 10
```

**TAMAÑO DEL ARRAY**

```
numeros=(1 2 3 4 5)
echo ${#numeros[*]}

tamaño de un solo elemento

numeros=(Primero Segundo Tercero)
echo ${#numeros[1]}

```

**secuencias dentro de un array**

```
cifrasLetras=( {A..Z} {a..z} {0..9} )
echo ${cifrasLetras[*]}

OUTPUT:

A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z 0 1 2 3 4 5 6 7 8 9
```

**metiendoo una lista de archivos en un array**

```
#!/bin/bash
ficheros=(`ls`)
echo ${ficheros[*]}
echo ${#ficheros[*]}
```

## USO DE SHIFT

shift vuelve a asignar los parámetros de posición ( $1, $2, etc.) de manera que $1 asume el antiguo valor de $2, $2 asume el valor de $3, etc 
El antiguo valor de $1 se descarta. ( $0 no ha cambiado).

# BASH SCRIPTING (SWITCH CASE Y FUNCIONES)

## switch case

sintaxis

```
#!/bin/bash
 
read -p "Enter your choice [yes/no]:" choice
 
case $choice in
     yes)
          echo "Thank you"
          echo "Your type: Yes"
          ;;
     no)
          echo "Ooops"
          echo "You type: No"
          ;;
     *)
          echo "Sorry, invalid input"
          ;;
esac

*) es el por defecto
```

Se debe dar permisos de ejecucion

### Multiples string como opcion

```
#!/bin/bash
 
read -p "Enter your choice [yes/no]:" choice
 
case $choice in
     Y/y/Yes/YES/yes)
          echo "Thank you"
          echo "Your type: Yes"
          ;;
     N/n/No/NO/no)
          echo "Ooops"
          echo "You type: No"
          ;;
     *)
          echo "Sorry, invalid input"
          ;;
esac
```

### usar patrones en los casos de un switch

```
read -p "Enter a string:" choice
shopt -s extglob  --> sin esto no funciona los patrones
case $choice in
     a*)                    ### matches anything starting with "a"
          #script here
          ;;
 
     b?)                    ### matches any two-character string starting with "b"
          #script here
          ;;
 
     s[td])                 ### matches "st" or "sd"
          #script here
          ;;
 
     r[ao]m)                ### matches "ram" or "rom"
          #script here
          ;;
 
     me?(e)t)               ### matches "met" or "meet"
          #script here
          ;;
 
     @(a|e|i|o|u))          ### matches one vowel
          #script here
          ;;
 
     *)                     ### Catchall matches anything not matched above
          #script here
          ;; 
esac
```

## FUNCIONES

**sintaxis**

```
funcationName(){    
  // scope of function
}

functionName  //calling of function
```

**funciones con argumentos**

```
#!/bin/bash

funArguments(){
   echo "First Argument: " $1
   echo "Second Argument: " $2
   echo "Third Argument: " $3
   echo "Fourth Argument: " $4
}

# Call funArguments from anywhere in the script using parameters like below

funArguments 1 Welcome to TecAdmin --> PASO DE PARAMETROS

OUTPUT:

First Argument : 1
Second Argument : Welcome
Third Argument : to
Fourth Argument : TecAdmin
```

**funciones de retorno**

```
funReturnValues(){
echo "5" --> AQUI RETORNO
}

# Call funReturnValues from any where in script and get return values

values=$(funReturnValues) --> ASIGNO VALORES
echo "Return value is : $values"
```

**recursividad**

```
#!/bin/bash

funRecursive(){
val=$1
if [ $val -gt 5 ]
then
	exit 0
else
	echo $val
fi
val=$((val+1))
funRecursive $val     # Function calling itself here
}

# Call funRecursive from any where in script

funRecursive 1
```

# BASH SCRIPTING (AVANZADO)

## controlar el evento ctrl + c

```
trap ctrl_c INT

function ctrl_c(){
    echo "Esto se ejecuta al pulsar control + c"
}
sleep 5 -> se deja en espera un momento para aplicar cambios
```

buen para matar procesos al salir

## SLEEP

dejar en pausa la consola

1.  s for seconds (the default)
2.  m for minutes.
3.  h for hours.
4.  d for days

```
sleep 5 -> 5 segundos
sleep 1.5 -> 1.5 segundos
sleep .5 -> 0.5 segundos
sleep 2m -> 2 minutosw
sleep 3h -> 3 horas
sleep 5d -> 5 dias

ejemplo:

command1 && sleep 1m && command2
```

## verificar si es un usuario root

```
if ["$(id -u)"=="0"];
then
    //es root
else
    // no es
fi
```

## colores de texto de la terminal

```
#Colours
greenColour="\e[0;32m\033[1m"
endColour="\033[0m\e[0m"
redColour="\e[0;31m\033[1m"
blueColour="\e[0;34m\033[1m"
yellowColour="\e[0;33m\033[1m"
purpleColour="\e[0;35m\033[1m"
turquoiseColour="\e[0;36m\033[1m"
grayColour="\e[0;37m\033[1m"

uso:

echo -e "${redColour}hola${endColourendColour}"
-e para que interprete caracteres especiales (\n \t etc)
siempre terminar con el endColour sinoo se queda toda la terminal con el color definido de inicio

```

## *while optimo

```
while getops ":a:b:c:" arg; do   (getopt es como un case dode los parametros seran -a,-b, -c pero se lo coloca con ':' en el while, siempre debe terminar en ':')

case $arg in (hacemos el case)
    a) variable = $OPTARG;; (en $OPTARG se tiene el parametro pasado con -a paraetro...)
    b) ;;
    c) function ;;
esac
done

ya debajo del done podems manipular la 'variable'


```

## DECLARE

declaracion de variables con tipo
asi se ooptimiza memoria

```
constante (variable que solo se puede leer y no editar)

declare -r NOMBRE_CONSTANTE="valor fijo de la constante"

enteros

declare -i variable=5
si se le asigna un string deuelve 0

arrays

declare -a usuarios=([0]='juan' [1]='pepe' [2]='ana' [3]='eugenia')

```

## SUBCADENAS

```
${string:position:length}

EJEMPLO

long="USCAGol.blah.blah.blah"
short="${long:0:2}" ; echo "${short}"
```
