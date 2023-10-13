# Metodologia de enumeracion

## INTRODUCCION

Los procesos complejos deben tener una metodología estandarizada que nos ayude a orientarnos y evitar omitir algún aspecto por error. Especialmente con la variedad de casos que nos pueden ofrecer los sistemas de destino, es casi impredecible cómo debe diseñarse nuestro enfoque. Por lo tanto, la mayoría de los probadores de penetración siguen sus hábitos y los pasos con los que se sienten más cómodos y familiarizados. Sin embargo, esta no es una metodología estandarizada, sino más bien un enfoque basado en la experiencia.

## METODOLOGIA

Esta metodología está anidada en 6 capas y representa, metafóricamente hablando, límites que tratamos de pasar con el proceso de enumeración. Todo el proceso de enumeración se divide en tres niveles diferentes:

- `Infrastructure-based enumeration`
- `Host-based enumeration`
- `OS-based enumeration`

![[Pasted image 20230612221727.png]]

|**Capa**|**Descripción**|**Categorías de información**|
|:------:|:------:|:------:|
|`1. Internet Presence`|Identificación de presencia en Internet e infraestructura accesible externamente.|Dominios, subdominios, vHosts, ASN, Netblocks, direcciones IP, instancias en la nube, medidas de seguridad|
|`2. Gateway`|Identificar las posibles medidas de seguridad para proteger la infraestructura externa e interna de la empresa.|Cortafuegos, DMZ, IPS/IDS, EDR, Proxies, NAC, Segmentación de red, VPN, Cloudflare|
|`3. Accessible Services`|Identifique las interfaces y los servicios accesibles que se alojan externa o internamente.|Tipo de servicio, funcionalidad, configuración, puerto, versión, interfaz|
|`4. Processes`|Identifique los procesos internos, las fuentes y los destinos asociados con los servicios.|PID, datos procesados, tareas, origen, destino|
|`5. Privileges`|Identificación de los permisos y privilegios internos a los servicios accesibles.|Grupos, Usuarios, Permisos, Restricciones, Entorno|
|`6. OS Setup`|Identificación de los componentes internos y configuración de los sistemas.|Tipo de sistema operativo, nivel de parche, configuración de red, entorno del sistema operativo, archivos de configuración, archivos privados confidenciales|

