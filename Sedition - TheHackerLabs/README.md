Sedition es una máquina de dificultad media de The Hacker Labs enfocada en técnicas de enumeración, obtención de credenciales y escalada de privilegios en entornos Linux. 

En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios SMB y SSH, el análisis de archivos y credenciales filtradas, el acceso a bases de datos MariaDB y el movimiento lateral entre usuarios, hasta finalmente conseguir privilegios de root aprovechando una configuración insegura de sudo sobre el binario `sed`.


---

### **Reconocimiento**

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![1. ping.png](attachment:5afb79cf-abf2-4f6c-bc42-f665e1457f1c:a1c40397-092f-4e80-bf0d-29ddb24414a9.png)

Continuamos con un escaneo **nmap** para enumerar los puertos abiertos y los servicios en ejecución dentro de la máquina:

![2. nmap.png](attachment:62f16a91-4974-4bcf-84c2-4fede2ce253f:5a6a7cc5-fd3a-4c97-92a2-be7b88aa9180.png)

El escaneo nos muestra que el servicio **SMB** está corriendo en la máquina, vamos a ver que encontramos accediendo a él con **smbclient**:

![3. smbclient.png](attachment:bd3822ab-21e5-4334-b50e-2bcd50ddd5ef:c6a024e3-912c-4a6d-90d0-84a9277d345e.png)

 Vemos un recurso compartido agregado personalizado llamado **backup**, vamos a acceder a él y a descargar el archivo comprimido que está dentro:

![4. secretito.png](attachment:6ab76aef-284b-4e2b-a8f9-b9737c07327c:4._secretito.png)

---

### Explotación

Una vez descargamos el archivo, cuando intentamos descomprimirlo nos pide una contraseña, la cual es: `sebastian` (la obtuve haciendo uso de **zip2john** para obtener el hash del .zip y descifrarlo).

Teniendo la contraseña podemos extraer el contenido del archivo comprimido:

![5. unzip.png](attachment:a187e06f-9f6f-44bb-a2b9-ecbcd55fe1b3:c6bac418-afcf-48cf-a67b-613384346038.png)

Vemos que contenía un archivo llamado **password**, vamos a ver que contiene:

![6. password.png](attachment:68850c46-976a-4bf2-b2a5-e935f44906be:8752d2a5-df9a-4c44-9d76-6066bff4a0dc.png)

El archivo **password** contiene la contraseña de algún usuario. Ahora haremos uso de **Hydra** para adivinar el usuario al que pertenece esa contraseña, para iniciar sesión a través de **SSH**:

![7. hydra ssh.png](attachment:986e0e00-e299-4d17-a553-05d87cc5def3:56fa3e83-9254-4087-ac12-30029782594e.png)

Hallamos el usuario **cowboy**. Es momento de acceder a la máquina con  las credenciales que obtuvimos:

![8. ssh.png](attachment:84603b50-473c-4ffc-b996-d79871bb6775:9b86023d-80b9-452f-bf54-fb09a591bb5a.png)

Ahora vamos a listar los comandos anteriormente ejecutados por **cowboy** con el comando `history`, para ver si hallamos algo interesante:

![9. history.png](attachment:0218cffd-8308-43e1-8a3c-7b72bd2e653f:2ea5ac09-0f13-4895-bdb6-cf775af9a7d8.png)

Vemos que el usuario había accedido anteriormente a un servidor local de **MariadDB** con las credenciales mostradas.

Ahora que entendemos esto, accedamos a **MariaDB** y listemos las bases de datos almacenadas con el comando: `SHOW DATABASES;`.

![10. show db.png](attachment:8bf4d5f6-7beb-4f1a-9fda-1bf5efc1ae9f:4ba4eb20-b0a5-41d1-8f2b-a5230744a4ff.png)

Vamos a acceder a la base de datos **bunker** con el comando `USE bunker;`  y listamos las tablas que contiene con: `SHOW TABLES;`

![11. show tables.png](attachment:17797d9e-72c3-410a-a2e0-a2f9e99388f7:f4dfc530-4332-4451-8fad-9fde0ddd9e65.png)

Ahora vemos el contenido de la tabla con el comando `SELECT * FROM users;` 

![12. users.png](attachment:9d9a8002-5d90-4b5d-a684-f2386c1fcdf9:fbca1ecb-1c79-43a7-a024-6de1b80bf9c5.png)

Hallamos el nombre de usuario **debian** y el hash de su contraseña. Ahora haremos uso de **hashcat** para descifrar la contraseña: 

![13. hashcat.png](attachment:663cd25b-f8ba-4600-aa4c-0313032f0829:bdc88845-68f7-4b90-b713-ac0dba91e194.png)

Luego de descifrar el hash obtuvimos la contraseña: **password1**. Ahora vamos a cambiar al usuario **debian** en la máquina objetivo:

![14. debian.png](attachment:d0479e29-6b7a-40ba-a6e7-f59641e1ee56:43cb127e-2811-4fc4-ae97-889f0636d9eb.png)

Una vez siendo **debian** seremos capaces de conseguir el **user flag**:

![15. userflag.png](attachment:325ade23-dd4b-4c54-b1ac-01d51ebf2e66:928d7ff1-8f11-46b9-b978-55944a6a705c.png)

### Post-Explotación/Escalada de privilegios

Llegó el momento de elevar privilegios y para ello empezaremos listando los comandos que el usuario puede ejecutar como sudo con el comando: `sudo -l`

![16. sudo -l.png](attachment:8adce48c-a51a-4184-b21b-c2aa4484e123:a2468db4-1848-4a75-8924-c8924c51763d.png)

Vemos que podemos ejecutar el comando **sed** como sudo. Haciendo una breve búsqueda en **gtfobins** hallamos una manera de aprovecharnos de **sed** para elevar privilegios:

![17. sed gtfobins.png](attachment:1301ca40-da42-498f-a16d-995a35239c29:e9fa5972-2f67-42f0-bbd2-86b9d6b0d0d5.png)

Ejecutando los comandos mostrados obtendríamos una shell como **root**:

![18. privesc.png](attachment:69899df3-fc47-4b81-9754-5aef2ec34a0a:a4324cf7-abd6-4069-93c5-e4a7dd6c3480.png)

Ahora obtendremos el **root flag**:

![19. rootflag.png](attachment:c8a172a9-cb2b-45d1-8347-91db8cc043f4:e7b00ea9-bbf0-4040-809c-a94adf0f8bc3.png)

**¡Y con esto habremos hecho nuestro el sistema de la máquina Sedition de The Hacker Labs!**
