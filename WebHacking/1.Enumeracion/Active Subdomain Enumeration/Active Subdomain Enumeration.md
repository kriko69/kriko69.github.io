# Active Subdomain Enumeration

## ZoneTransfers

La transferencia de zona es cómo un servidor DNS secundario recibe información del servidor DNS primario y la actualiza. El enfoque maestro-esclavo se usa para organizar servidores DNS dentro de un dominio, y los esclavos reciben información DNS actualizada del DNS maestro. El servidor DNS maestro debe configurarse para permitir transferencias de zona desde servidores DNS secundarios (esclavos), aunque esto podría estar mal configurado.

- [https://hackertarget.com/zone-transfer/](https://hackertarget.com/zone-transfer/)

![[Pasted image 20221216222608.png]]

### FORMA MANUAL CON NSLOOKUP

1. Identificacion de Nameservers

```bash
nslookup -type=NS zonetransfer.me

#output
Server:		10.100.0.1
Address:	10.100.0.1#53

Non-authoritative answer:
zonetransfer.me	nameserver = nsztm2.digi.ninja. #nameserver 1
zonetransfer.me	nameserver = nsztm1.digi.ninja. #nameserver 1
```

2. Prueba de transferencia de zona ANY y AXFR

Esto por cada nameserver

```bash
nslookup -type=any -query=AXFR zonetransfer.me nsztm2.digi.ninja
nslookup -type=any -query=AXFR zonetransfer.me nsztm1.digi.ninja
```

>[!note]
>Si logramos realizar una transferencia de zona exitosa para un dominio, no es necesario continuar enumerando este dominio en particular, ya que esto extraerá toda la información disponible.

## Gobuster

Gobuster es una herramienta que podemos utilizar para realizar la enumeración de subdominios.

podemos hacer uso de patrones personalizados o diccionarios de seclist.

uso de patrones:

```bash
#pattern.txt

lert-api-shv-{GOBUSTER}-sin6
atlas-pp-shv-{GOBUSTER}-sin6
```

estos son subdominios que parecen tener un patron numerico que puede cambiar, es como los ns2., ns3., ns5., etc.

hacemos usodel patron:

```bash
export TARGET="facebook.com"
export NS="d.ns.facebook.com"
export WORDLIST="numbers.txt" #conjunto de numeros (del 1 al 10 por ejemplo)
gobuster dns -q -r "${NS}" -d "${TARGET}" -w "${WORDLIST}" -p ./patterns.txt -o "gobuster_${TARGET}.txt"

#output
Found: lert-api-shv-01-sin6.facebook.com
Found: atlas-pp-shv-01-sin6.facebook.com
Found: atlas-pp-shv-02-sin6.facebook.com
Found: atlas-pp-shv-03-sin6.facebook.com
Found: lert-api-shv-03-sin6.facebook.com
Found: lert-api-shv-02-sin6.facebook.com
Found: lert-api-shv-04-sin6.facebook.com
Found: atlas-pp-shv-04-sin6.facebook.com
```

## DIG

obtener nameserver:

```bash
dig ns dominio @IP

#si es una pagina publica en internet
#puedes probar el DNS publico de google u otro que se sepa que resuelva la pagina
dig ns inlanefright.htb @8.8.8.8

#se coloca la IP de la maquina en caso de estar dentro de una red privada
dig ns inlanefright.htb @10.129.33.176
```

![[Pasted image 20221216230456.png]]

realizamos transferencia de zona:

```bash
dig axfr dominio @nameserver

#en caso de red privada es necesario configurar 
#el dominio y el namserver en el /etc/hosts
dig axfr inlanefright.htb @10.129.33.176

#caso contrario y si se tiene salida a internet
dig axfr google.com @ns1.google.com.
```

![[Pasted image 20221216230922.png]]