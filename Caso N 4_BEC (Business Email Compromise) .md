# **Caso N 4:BEC (*Business Email Compromise*)** 

Nov 11, 2026 · @Facu

## **Datos del caso**

Empresa: Ferretería Los Álamos es una ferretería ficticia de Mendoza que le compra acero desde hace años a un proveedor: **Aceros Andinos SA**. 

Qué se reportó: Ayer el proveedor llamó para reclamar que **no les pagaron** una factura. En administración dicen que **sí la pagaron**, por transferencia, el 20/11. El proveedor jura que esa plata **nunca les llegó**. 

**¿Qué pasó?** 

El 03/11 llega un mail legitimo de un proveedor habitual con su perspectivismo factura. Se paga el 04/11 al CBU habitual (…12345678)

El 19/11, a las 17:52: El proveedor se contacta con nosotros via mail:

* Enviado desde un VPS de un hosting (`203.0.113.144`) con **PHPMailer**, no con Outlook.  
* `Reply-To` a un **Gmail**: las respuestas iban al atacante.  
* SPF pasa porque el dominio falso es del atacante; DKIM y DMARC sin configurar.  
* El texto mete **urgencia** ("pagar HOY") y evita la verificación ("el teléfono está en mantenimiento").  
* El PDF adjunto pide transferir a un **nuevo CBU** (…55512340, titular "Andinos Gestión SRL")

El 20/11 la administracion transfiere el dinero correspondiente al pedido realizado on el proveedor con un monto de **\$6.950.000** pero lo envian sin darse cuenta al CBU del atacante.

* **Tipo de ataque:** BEC (*Business Email Compromise*), fraude por cambio de cuenta bancaria.  
* **Hipótesis:** el buzón del proveedor pudo estar comprometido, porque el atacante conocía la factura pendiente, el nombre de la responsable y el formato de las facturas.  
* **Causa:** se cambiaron datos bancarios solo por mail, sin verificar por otro canal.

## **¿Cómo se resolvió?**

* **Investigación:**  
  * Comparación de los dos mails con `cat`.  
  * Encabezados clave con `grep -E "^(From|Reply-To|Return-Path|X-Mailer):"`: dominio parecido, Reply-To a Gmail y PHPMailer.  
  * Controles con `grep -A3 "^Authentication-Results"`: SPF pass del dominio falso, sin DKIM ni DMARC.  
  * Decodificación del adjunto con `sed`, `grep -v` y `base64 -d`: pedido de nuevo CBU.  
  * Cruce con `registro_pagos.txt`: el pago del 20/11 fue al CBU del atacante.  
* **Acciones inmediatas:**  
  * Llamar al banco para intentar **congelar** la transferencia.  
  * Hacer la **denuncia policial**.  
  * **Bloquear** el dominio falso, el Gmail del Reply-To y la IP.  
  * **Buscar** en todos los buzones otros mails de ese dominio.  
  * **Avisar al proveedor**, para que revise su correo y alerte a sus clientes.  
* **Prevención:**  
  * Todo cambio de datos bancarios se verifica **por teléfono a un número agendado o en persona**, nunca por el mail.  
  * **Doble aprobación** de cambios de CBU y pagos grandes.  
  * **Capacitar** a administración en las señales: dominio parecido, Reply-To distinto, urgencia y "no llamen".  
  * **Etiquetar** los dominios reales de los proveedores y marcar los mails externos.  
  * Configurar **SPF, DKIM y DMARC** en el dominio propio.

## **Herramientas usadas**

Google Cloud Shell, `cat`, `grep` (con `-E`, `-A` y `-v`), `sed` y `base64` 