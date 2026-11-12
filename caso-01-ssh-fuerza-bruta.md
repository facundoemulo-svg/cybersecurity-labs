# **Caso N 1**

Nov 11, 2026 · @Facu

## **Datos del caso**

Número de caso:1

Empresa: Estudio Contable del Valle (ficticio), Mendoza. Tiene un servidor Linux (srv-archivos) con la documentación de sus clientes en /srv/clientes, accesible por SSH desde internet. 

Qué se reportó: Marcelo (sistemas) encontró en el servidor un usuario que nadie creó y menos espacio libre en el disco que el viernes. 

**¿Qué pasó?** 

**¿Qué pasó?**

Compromiso confirmado. Un atacante entró por fuerza bruta a SSH, robó la documentación de los clientes y dejó una puerta trasera para volver a entrar.

* 03:02–03:19 (10/11): fuerza bruta por SSH desde 203.0.113.88, con 628 intentos fallidos de un total de 668 en la semana.  
* 03:19:14: login exitoso con la cuenta “soporte” (cuenta de un proveedor externo con permisos sudo), adivinando su contraseña.  
* 03:23–03:31: con sudo, lee /etc/shadow (hashes de todas las contraseñas), crea el usuario “sysbackup” con permisos de administrador, le instala una llave SSH propia (persistencia) y comprime /srv/clientes en /tmp/.c.tgz.  
* 03:31–03:41: envía /tmp/.c.tgz por scp a 198.51.100.40 y borra el archivo para no dejar rastro. Fuga de datos de clientes confirmada.  
* 04:12: vuelve a entrar con la cuenta “sysbackup” y su llave SSH desde 198.51.100.40. La puerta trasera ya está en uso.

Causa raíz: SSH abierto a internet con login por contraseña (PasswordAuthentication yes) y login directo de root (PermitRootLogin yes); una cuenta de proveedor externo con sudo y contraseña débil; ningún bloqueo de intentos ni alertas.

## **¿Cómo se resolvió?**

Investigación (en Google Cloud Shell):

1. Conté los intentos fallidos con `grep -c "Failed password"`: 668\.  
2. Agrupé los intentos por IP con `grep -oE "from [0-9.]+" | sort | uniq -c | sort -rn`: 628 venían de 203.0.113.88.  
3. Busqué logins exitosos de esa IP con `grep "Accepted"`: entró como “soporte” a las 03:19.  
4. Revisé passwd.txt y grupo\_sudo.txt: “soporte” y el nuevo “sysbackup” tienen sudo.  
5. Busqué los comandos de sudo con `grep "COMMAND"`: creación de usuario, llave SSH y compresión de /srv/clientes.  
6. Leí el bash\_history de “soporte”: envío de los datos por scp a 198.51.100.40.  
7. Busqué esa IP en el log: volvió a entrar con “sysbackup” a las 04:12.

Contención:

* Aislar el servidor.  
* Bloquear 203.0.113.88 y 198.51.100.40.  
* Bloquear las cuentas “sysbackup” y “soporte” sin borrarlas (evidencia) y cerrar sus sesiones.  
* Quitar la llave del atacante de authorized\_keys, guardando antes una copia.

Erradicación:

* Cambiar todas las contraseñas del servidor, porque el atacante tiene los hashes de /etc/shadow.  
* Buscar otras puertas traseras: llaves en otros authorized\_keys (incluido root), crontab y usuarios nuevos.

Prevención:

* SSH solo con llaves (PasswordAuthentication no) y sin root (PermitRootLogin no).  
* Quitar sudo a la cuenta del proveedor y darle acceso solo cuando lo necesite.  
* Instalar fail2ban y restringir SSH a una VPN o a IPs conocidas.  
* Alertas ante muchos intentos fallidos.

Comunicación: avisar a los clientes del estudio por la fuga de su documentación y evaluar las obligaciones de la Ley 25.326 de protección de datos personales.

## **Herramientas usadas**

`grep`, `sort`, `uniq`, `cat`, `head`. 

