# Introducción

Uploader es una máquina de dificultad Easy de The Hacker Labs enfocada en técnicas de explotación web, obtención de credenciales y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios HTTP y el abuso de una funcionalidad vulnerable de subida de archivos para obtener acceso inicial al sistema. Posteriormente, se realizará movimiento lateral mediante el análisis y descifrado de archivos comprimidos con credenciales filtradas, hasta finalmente conseguir privilegios de root aprovechando permisos inseguros de sudo sobre el binario `tar`.

---

## **Reconocimiento**

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![1. ping.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/1.png) 

Continuamos con un escaneo **nmap** para enumerar los puertos abiertos y los servicios en ejecución dentro de la máquina:

![2. nmap.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/2.png)

El resultado nos muestra que tenemos que el único puerto abierto es el **80** con un servidor **HTTP** en ejecución. Vamos a acceder a la página:

![3. pagina.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/3.png)

Vemos que es una página de subida de archivos, vamos a explorar:

![4. pagina upload.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/4.png)

---

## Explotación

Encontramos un formulario de subida de archivos, intentemos subir una reverse shell:

![5. subida de rv shell.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/5.png)

![5. 2.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/5.5.png)

Se subió correctamente y la podemos encontrar en el directorio **uploads**.

![6. direc uploads.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/6.png)

 Iniciemos una conexión en escucha con **netcat** para posteriormente ejecutar la reverse shell:

![7. nc.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/7.png)

Es momento de ejecutar la reverse shell:

![8. exec rv shell.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/8.png)

Y una vez la ejecutemos habremos obtenido acceso al sistema:

![9. acceso a la maquina.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/9.png)

Navegando un poco hallamos una pista que nos indica que existe un archivo comprimido con credenciales de otro usuario. Haremos una búsqueda de dicho archivo con **find**:

![11. find zip.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/10.png)

Localizamos el archivo comprimido, ahora intentemos descomprimirlo con **unzip**:

![12. unzip no.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/11.png)

Al parecer **unzip** no existe en el sistema. Vamos a hacer uso de **base64** para transferir el archivo comprimido a nuestra máquina para trabajar allí:

![13. base64.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/12.png)

Copiamos la salida de **base64** y lo pegamos para decodificarlo en nuestra máquina:

![14. base64 -d.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/13.png)

Con esto logramos transferir el archivo comprimido a nuestra máquina.

Ahora vamos a intentar descomprimirlo con **7z**:

![15. contrasena 7z.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/14.png)

El archivo solicita una contraseña para poder descomprimirlo. Vamos a usar **zip2john** para extraer el **hash** de la contraseña del **archivo .zip**:

![16. zip2john.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/15.png)

Una vez extraído el **hash** haremos uso de **john** para hallar la contraseña que corresponde al **hash**:

![17. jhon.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/16.png)

![18. john show.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/17.png)

Ya que tenemos la contraseña procedemos a descomprimir el archivo y a ver su contenido:

![19. descompresion y cat.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/18.png)

Obtuvimos las credenciales del usuario **operatorx,** pero la contraseña esta hasheada, así que usaremos **hashcat** para crackear el **hash**:

![20. hashcat y contrasena.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/19.png)

Ahora sí, volvamos a la máquina objetivo e iniciemos sesión como **operatorx**:

![21. operatorx.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/20.png)

En este punto podremos obtener la **user flag**:

![22. userflag.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/21.png)

---

## Post-Explotación/Escalada de privilegios

Llegó el momento de elevar privilegios, vamos a iniciar listando los comando que el usuario puede ejecutar como sudo con: `sudo -l`

![23. sudo -l.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/22.png)

Encontramos el comando **tar**. Haciendo una breve búsqueda en gtfobins hallamos una forma de elevar privilegios sacando provecho de este comando:

![24. gtfobins tar.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/23.png)

Ejecutando los comando mostrados ganaremos una shell como **root** y tendremos acceso a la **root flag**:

![25. rootflag.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/24.png)
![Pwned.png](https://github.com/vonoHC/Writeups/blob/main/Uploader%20-%20TheHackerLabs/Capturas/25.png)
**¡Y con esto habremos hecho nuestro el sistema de la máquina Uploader de The Hacker Labs!**

