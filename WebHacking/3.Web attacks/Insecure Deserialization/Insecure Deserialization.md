# INSECURE DESERIALIZATION ATTACK

## Table of contents

- [INSECURE DESERIALIZATION ATTACK](#insecure-deserialization-attack)
  - [QUE ES LA SERIALIZACION](#que-es-la-serializacion)
  - [COMO NOS APROVECHAMOS DE ESTO](#como-nos-aprovechamos-de-esto)
  - [ENUMERACION](#enumeracion)
  - [JexBoss Tool](#jexboss-tool)
  - [WAR MALICIOSO](#war-malicioso)
  - [YSOSERIAL](#ysoserial)

## QUE ES LA SERIALIZACION

La serialización en un resumen es el proceso de convertir datos, específicamente "Objetos" en lenguajes de Programación Orientada a Objetos (POO) como Java en un formato de nivel inferior conocido como "flujos de bytes", donde se pueden almacenar para su uso posterior, como dentro archivos, bases de datos y/o a través de una red. Luego se convierte más tarde de este "flujo de bytes" nuevamente en el "Objeto" de nivel superior. Esta conversión final se conoce como "Desserialización".

![goto](./img/1.png)

## COMO NOS APROVECHAMOS DE ESTO

Un ataque de "serialización" es la inyección y/o modificación de datos a lo largo de la etapa de "flujo de bytes". Cuando la aplicación accede más tarde a estos datos, el código malicioso puede tener implicaciones graves... que van desde DoS, fuga de datos o ataques mucho más nefastos como ser "rooteado".

## ENUMERACION

Los siguientes puertos y servicios pueden ser vulnerables:

- Puertos: 1090, 1098, 1099, 8009, 8080, 8083
- Servicios: Jboss, Struts, Jenkins, Apache tomcat

```bash
# Nmap 7.80 scan initiated Fri Apr 17 19:53:10 2020 as: nmap -sV -sC -o nmap_scan 10.10.91.15
Nmap scan report for 10.10.91.15
Host is up (0.20s latency).
Not shown: 989 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d6:97:8c:b9:74:d0:f3:9e:fe:f3:a5:ea:f8:a9:b5:7a (DSA)
|   2048 33:a4:7b:91:38:58:50:30:89:2d:e4:57:bb:07:bb:2f (RSA)
|   256 21:01:8b:37:f5:1e:2b:c5:57:f1:b0:42:b7:32:ab:ea (ECDSA)
|_  256 f6:36:07:3c:3b:3d:71:30:c4:cd:2a:13:00:b5:25:ae (ED25519)
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Hugo 0.66.0
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Tony&#39;s Blog
1090/tcp open  java-rmi    Java RMI
|_rmi-dumpregistry: ERROR: Script execution failed (use -d to debug)
1091/tcp open  java-rmi    Java RMI
1098/tcp open  java-rmi    Java RMI
1099/tcp open  java-object Java Object Serialization
| fingerprint-strings: 
|   NULL: 
|     java.rmi.MarshalledObject|
|     hash[
|     locBytest
|     objBytesq
|     #http://thm-java-deserial.home:8083/q
|     org.jnp.server.NamingServer_Stub
|     java.rmi.server.RemoteStub
|     java.rmi.server.RemoteObject
|     xpwA
|     UnicastRef2
|_    thm-java-deserial.home
4446/tcp open  java-object Java Object Serialization
5500/tcp open  hotline?
| fingerprint-strings: 
|   DNSStatusRequestTCP: 
|     DIGEST-MD5
|     NTLM
|     GSSAPI
|     CRAM-MD5
|     thm-java-deserial
|   DNSVersionBindReqTCP: 
|     DIGEST-MD5
|     NTLM
|     CRAM-MD5
|     GSSAPI
|     thm-java-deserial
|   GenericLines, NULL: 
|     CRAM-MD5
|     DIGEST-MD5
|     GSSAPI
|     NTLM
|     thm-java-deserial
|   GetRequest: 
|     NTLM
|     GSSAPI
|     DIGEST-MD5
|     CRAM-MD5
|     thm-java-deserial
|   HTTPOptions, TLSSessionReq: 
|     DIGEST-MD5
|     CRAM-MD5
|     NTLM
|     GSSAPI
|     thm-java-deserial
|   Help: 
|     NTLM
|     GSSAPI
|     CRAM-MD5
|     DIGEST-MD5
|     thm-java-deserial
|   Kerberos: 
|     CRAM-MD5
|     DIGEST-MD5
|     NTLM
|     GSSAPI
|     thm-java-deserial
|   RPCCheck: 
|     DIGEST-MD5
|     CRAM-MD5
|     GSSAPI
|     NTLM
|     thm-java-deserial
|   RTSPRequest: 
|     CRAM-MD5
|     GSSAPI
|     DIGEST-MD5
|     NTLM
|     thm-java-deserial
|   SSLSessionReq: 
|     GSSAPI
|     CRAM-MD5
|     NTLM
|     DIGEST-MD5
|     thm-java-deserial
|   TerminalServerCookie: 
|     CRAM-MD5
|     GSSAPI
|     NTLM
|     DIGEST-MD5
|_    thm-java-deserial
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
| ajp-methods: 
|   Supported methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|   Potentially risky methods: PUT DELETE TRACE
|_  See https://nmap.org/nsedoc/scripts/ajp-methods.html
8080/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
| http-methods: 
|_  Potentially risky methods: PUT DELETE TRACE
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Welcome to JBoss AS
8083/tcp open  http        JBoss service httpd
|_http-title: Site doesn't have a title (text/html).
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port1099-TCP:V=7.80%I=7%D=4/17%Time=5E9A4F92%P=x86_64-pc-linux-gnu%r(NU
... [ snip ] ...
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Apr 17 19:54:24 2020 -- 1 IP address (1 host up) scanned in 73.17 seconds
```

## JexBoss Tool

Podemos verificar si la version de jboss es vulnerable a este ataque:

verificacion de la version, por ejemplo si el jboss esta en http://10.10.10.10/8080 podemos comprobar en la siguiente ruta:

```bash
http://10.10.10.10/8080/jbossws/
```

podemos utilizar la herramienta [https://github.com/joaomatosf/jexboss](https://github.com/joaomatosf/jexboss) para validar vulnerabilidades:

```bash
python jexboss.py -u http://10.10.154.94:8080
```

## WAR MALICIOSO

**CREDENCIALES POR DEFECTO JBOSS: ADMIN - ADMIN**

podemos cargar un WAR al estimo de la explotacion de tomcat.

La ruta para cargar el war es **JBossAS Servers > JBoss AS 6 (default) > Applications > Web Applications (WAR)s**

## YSOSERIAL

