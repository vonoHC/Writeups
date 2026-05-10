# Introducción
Sedition es una máquina de dificultad Easy de The Hacker Labs enfocada en técnicas de enumeración, obtención de credenciales y escalada de privilegios en entornos Linux. 

En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios SMB y SSH, el análisis de archivos y credenciales filtradas, el acceso a bases de datos MariaDB y el movimiento lateral entre usuarios, hasta finalmente conseguir privilegios de root aprovechando una configuración insegura de sudo sobre el binario `sed`.


---

## **Reconocimiento**

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![1. ping.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/1.png)

Continuamos con un escaneo **nmap** para enumerar los puertos abiertos y los servicios en ejecución dentro de la máquina:

![2. nmap.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/2.png)

El escaneo nos muestra que el servicio **SMB** está corriendo en la máquina, vamos a ver que encontramos accediendo a él con **smbclient**:

![3. smbclient.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/3.png)

 Vemos un recurso compartido agregado personalizado llamado **backup**, vamos a acceder a él y a descargar el archivo comprimido que está dentro:

![4. secretito.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/4.png)

---

## Explotación

Una vez descargamos el archivo, cuando intentamos descomprimirlo nos pide una contraseña, la cual es: `sebastian` (la obtuve haciendo uso de **zip2john** para obtener el hash del .zip y descifrarlo).

Teniendo la contraseña podemos extraer el contenido del archivo comprimido:

![5. unzip.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/5.png)

Vemos que contenía un archivo llamado **password**, vamos a ver que contiene:

![6. password.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/6.png)

El archivo **password** contiene la contraseña de algún usuario. Ahora haremos uso de **Hydra** para adivinar el usuario al que pertenece esa contraseña, para iniciar sesión a través de **SSH**:

![7. hydra ssh.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/7.png)

Hallamos el usuario **cowboy**. Es momento de acceder a la máquina con  las credenciales que obtuvimos:

![8. ssh.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/8.png)

Ahora vamos a listar los comandos anteriormente ejecutados por **cowboy** con el comando `history`, para ver si hallamos algo interesante:

![9. history.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/9.png)

Vemos que el usuario había accedido anteriormente a un servidor local de **MariadDB** con las credenciales mostradas.

Ahora que entendemos esto, accedamos a **MariaDB** y listemos las bases de datos almacenadas con el comando: `SHOW DATABASES;`.

![10. show db.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/10.png)

Vamos a acceder a la base de datos **bunker** con el comando `USE bunker;`  y listamos las tablas que contiene con: `SHOW TABLES;`

![11. show tables.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/11.png)

Ahora vemos el contenido de la tabla con el comando `SELECT * FROM users;` 

![12. users.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/12.png)

Hallamos el nombre de usuario **debian** y el hash de su contraseña. Ahora haremos uso de **hashcat** para descifrar la contraseña: 

![13. hashcat.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/13.png)

Luego de descifrar el hash obtuvimos la contraseña: **password1**. Ahora vamos a cambiar al usuario **debian** en la máquina objetivo:

![14. debian.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/14.png)

Una vez siendo **debian** seremos capaces de conseguir el **user flag**:

![15. userflag.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/15.png)

## Post-Explotación/Escalada de privilegios

Llegó el momento de elevar privilegios y para ello empezaremos listando los comandos que el usuario puede ejecutar como sudo con el comando: `sudo -l`

![16. sudo -l.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/16.png)

Vemos que podemos ejecutar el comando **sed** como sudo. Haciendo una breve búsqueda en **gtfobins** hallamos una manera de aprovecharnos de **sed** para elevar privilegios:

![17. sed gtfobins.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/17.png)

Ejecutando los comandos mostrados obtendríamos una shell como **root**:

![18. privesc.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/18.png)

Ahora obtendremos el **root flag**:

![19. rootflag.png](https://github.com/vonoHC/Writeups/blob/main/Sedition%20-%20TheHackerLabs/Capturas/19.png)

**¡Y con esto habremos hecho nuestro el sistema de Sedition!**
