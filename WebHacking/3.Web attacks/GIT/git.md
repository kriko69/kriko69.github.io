# GIT

los archivos **.git** pueden tener informacion valiosa ya que por cada commit puede ser que se hayan olvidado alguna contraseña, token o informacion confidencial.

## Table of contents

- [GIT](#git)
  - [RECOPILACION DE INFORMACION A TRAVES DE DESCARGA DE CODIGO FUENTE](#recopilacion-de-informacion-a-traves-de-descarga-de-codigo-fuente)
  - [RECOPILACION DE INFORMACION A TRAVES DE WEB PATH](#recopilacion-de-informacion-a-traves-de-web-path)

## RECOPILACION DE INFORMACION A TRAVES DE DESCARGA DE CODIGO FUENTE

Si es que en algun lugar puede descargar el codigo fuente de una aplicacion y este tiene la carpeta **.git**, puede usar la herramienta [GitTools](https://github.com/internetwache/GitTools):

```bash
extract.sh repositorio /tmp/result
```

con esto podra extraer todos los commits realizados en el repositorio (ruta donde esta el .git) y puede ver el historial de comandos:

```bash
git log

tree -fas

fzf-lovely
```

- HTB secret
- Wreath THM

Podemos revisar cada commit tiene un archivo `commit-meta.txt`, el que no tenga el valor `parent` es el ultimo commit. Podemos ir viendo quien es el padre de quien para ver la cronologia

## RECOPILACION DE INFORMACION A TRAVES DE WEB PATH

puede que una de las rutas de la pagina web sea **/.git**, entonces usando la misma herramienta puede primeramente descargar el repositorio y seguir los pasos del aso anterior:

```bash
dumper.sh http://10.10.10.10/.git /tmp/result
```

esto descargar el contenido de **.git** para que pueda extraer los commits y verlos.

- [https://tryhackme.com/room/githappens](https://tryhackme.com/room/githappens)