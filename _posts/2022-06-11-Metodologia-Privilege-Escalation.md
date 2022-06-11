---
title: Metodologia Privilege Escalation
published: true
---


# METODOLOGIA PARA ELEVAR PRIVILEGIOS

## WINDOWS

- enumeracion de privilegios (**whoami /priv** y **SeImpersonatePrivilege**)
- verificar kernel exploits (**systeminfo**)
- verificacion de servicios de red (**netstat -ano**)
- verificacion de interfaces de red (**ipconfig**)
- buscar contraseñas en la raiz
- tirar winpeas
- listar programas instaladosy verificar sus permisos (**dir C:\\Program Files** y **accesschk.exe**)
- verificar AlwaysInstallElevated es igual a 1 (**reg query HKLM\\Software\\Policies\\Microsoft\\Windows\\Installer** y **reg query HKCU\\Software\\Policies\\Microsoft\\Windows\\Installer*)
- verificar **Modifiable Services** (ver windows_privilege_escalation file)
- verificar **unquoted service path** (ver windows_privilege_escalation file)
- verificar **service binpath** (ver windows_privilege_escalation file)

## LINUX

- enumeracion del sistema (**whoami, id**)
- verificar permisos docker LXC / LXD (**id**)
- enumerar procesos (**netstat -ano, ps -f aux**)
- enumerar interfaces de red (**ifconfig, ip a**)
- verificar sudo misconfiguration (**sudo -l**)
- buscar kernel exploits (**uname -a**)
- buscar contraseñas en el sistema
- buscar SSH keys, bash history
- enumerar archivos con permisos de escritura
- buscar permisos SUID (Combinar con GTFOBins, PATH HIJACKING, WILDCARD INJECTION, BoF)
- buscar tareas cron (**pspy, cat /var/spool/cron/crontabs/\<user\>**)
- buscar capabilities
- verificar si hay monturas (Docker, **mount -l**, **cat /etc/fstab** )
- verificar NFS Root Squashing