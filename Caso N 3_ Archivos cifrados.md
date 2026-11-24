# **Caso N 3: Archivos cifrados (Inmobiliaria del Cerro)** 

Nov 11, 2026 · @Facu

## **Datos del caso**

Empresa: Inmobiliaria del Cerro (ficticia), Mendoza, 6 empleados. Servidor Windows SRV-ARCHIVOS con carpeta compartida D:\\Compartido. 

Qué se reportó: El lunes 23/11 a las 08:30 nadie puede abrir los archivos: todos terminan en **`.cerro`** y en cada carpeta apareció un **`LEEME_RECUPERAR.txt`** 

**¿Qué pasó?** 

22/11 dia domingo entre la 01:40 y las 02:12: hubo un ataque de fuerza bruta por RDP desde 203.0.113.61 con 772 intentos fallidos. A las 02:13 logro ingresar el atacante con la cuenta “recepcion” por escritorio remoto (logontype 10\)

02:14–02:23: Reconocimiento (whoami, administradores, carpetas compartidas); descarga del ransomware svch0st.exe desde la misma ip 203.0.113.61; Borro copias de sombra, catalogos de backups y recuperacion de windows; 02:22:58: el ransomware se ejecuto en D:\\Compartido (extencion .cerro)

03:41: La misma cuenta borro el registro de auditoria (1102). La evidencia se conservo gracias al colector de logs centralizado

El ataque fue posible porque el RDP estaba expuesto a internet, sin bloqueo de cuenta ni MFA, la cuenta recepcion era administradora y los backups estaban en el mismo servidor. 

## **¿Cómo se resolvió?**

Contención: aislar SRV-ARCHIVOS sin apagarlo, cerrar el RDP publicado en el router, bloquear 203.0.113.61, deshabilitar recepcion y cambiar todas las credenciales.

Recuperación: los backups estaban en el mismo servidor y fueron borrados. Revisar si quedan copias en el disco E: y buscar un descifrador en nomoreransom.org. No pagar el rescate. 

Cerrar el RDP y usar VPN, activar el bloqueo de cuenta y MFA, quitar permisos de administrador a recepcion y aplicar backups 3-2-1 

Comunicación: avisar a los clientes cuyos contratos estaban en D:\\Compartido y evaluar con asesoría legal la notificación según la Ley 25.326 (AAIP). 

## **Herramientas usadas**

* Google Cloud Shell  
* `jq` (con `-r`, `-c`, `select` y `@tsv`)   
* `sort` y `uniq -c`   
* `cat`, `ls`, `cd` y `pwd` 

cd \~/caso-4

jq \-r '.\[\].EventID' evidencia/eventos\_security.json \| sort \| uniq \-c

jq \-r '.\[\] \| select(.EventID \== 4625\) \| .IpAddress' evidencia/eventos\_security.json \| sort \| uniq \-c \| sort \-rn

jq \-c '.\[\] \| select(.EventID \== 4624 and .IpAddress \== "203.0.113.61")' evidencia/eventos\_security.json

jq \-r '.\[\] \| select(.EventID \== 4688 and .SubjectUserName \== "recepcion") \| \[.TimeCreated, .CommandLine\] \| @tsv' evidencia/eventos\_security.json

jq \-c '.\[\] \| select(.EventID \== 1102)' evidencia/eventos\_security.json

