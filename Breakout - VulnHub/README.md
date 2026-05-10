# Introducción
Empire: Breakout es una máquina de dificultad media de VulnHub enfocada en técnicas de enumeración de servicios, análisis de aplicaciones web y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios HTTP y SMB, el análisis de credenciales ocultas mediante Brainfuck y la obtención de acceso a través de Usermin. Posteriormente, se realizará reconocimiento interno del sistema y se aprovecharán capacidades especiales asignadas al binario **tar** para acceder a archivos restringidos y obtener privilegios de root.

---

## Reconocimiento

Como primer paso, obtenemos la **IP** de la máquina objetivo y le realizamos un **ping**. De esta forma, sabremos si está activa.

![1. ping.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/1.png)

Ahora es momento de hacer un escaneo con **nmap** para enumerar los puertos y servicios que están activos en la máquina: 

![2. nmap.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/2.png)

Vemos que en la máquina están abiertos los puertos: **80**, **139**, **445**, **10000**, **20000**, los cuales están corriendo los servicios: **HTTP** (en los puertos **80**, **10000** y **20000**) y **SMB** (en los puertos **139** y **445**).

Vamos a ingresar a la página web por defecto del puerto 80 para ver que hallamos:

![3. pagina 80.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/3.png)

Es una página por defecto de **Apache**, revisemos el código fuente para ver si vemos algo interesante:

![4. codigo fuente.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/4.png)

Vemos un comentario que indica que el texto de abajo es una credencial de acceso encriptada. 

Investigando un poco, descubrimos que la credencial está escrita en un lenguaje de programación llamado **Brainfuck**, y lo decodificamos en el siguiente decoder: https://www.dcode.fr/brainfuck-language

![5. brainfuck decoder.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/5.png)

Al parecer es una contraseña para iniciar sesión en algún sitio, pero nos falta el usuario…
## Explotación
Como en la máquina está corriendo el servicio **SMB** podríamos usar la herramienta **enum4linux** para enumerar usuarios existentes, vamos a ello: 

![6. enum4.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/6.png)

![6. cyber.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/6.5.png)

La salida de **enum4linux** nos muestra que existe el usuario **cyber**. Ya que tenemos las credenciales de inicio de sesión de algún sitio es momento de hallarlo, para esto vamos a visitar las otras páginas que tiene la máquina:

![7. webmin.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/7.png)

En la página del puerto **10000** llamada **webmin**, no funcionan las credenciales, intentemos con la página en el puerto **20000**:

![8. usermin.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/8.png)

![8. inicio de sesion exitoso.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/8.5.png)

El inicio de sesión fue exitoso. Si navegamos un poco por la página seremos capaces de hallar una terminal web interactiva donde tendremos acceso a la **user flag**:

![9. userflag.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/9.png)

---

## Post-Explotación

Ahora llegó el momento de encontrar una forma de elevar privilegios, para ello vamos a listar los archivos ejecutables con el comando: **`getcap -r / 2>/dev/null`**

![El comando tar permite comprimir o descomprimir archivos y directorios.](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/10.png)

El comando tar permite comprimir o descomprimir archivos y directorios.

El resultado nos muestra que el comando tar tiene una capacidad especial que nos permite leer cualquier archivo sin importar los permisos, luego de comprimirlo. 

Luego de explorar por el sistema hallamos el archivo **`/var/backups/.old_pass.bak`** el cual tiene la contraseña para acceder al sistema como **root**:

![11. old_pass_bak.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/11.png)

Ahora sería momento de usar tar para comprimir y descomprimir el archivo para poder leerlo:

![12. com y descompresión de old.pass.bak.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/12.png)

Luego de haber hecho el proceso con los comandos mostrados en la captura anterior, podremos ver la contraseña del usuario **root**, la cual es la que se encuentra resaltada en la captura.

Ahora, si cerramos la sesión actual y podremos ingresar como **root** con las credenciales obtenidas:

![13. usermin root.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/13.png)

![13. login root.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/13.5.png)

En este punto solo faltaría arrancar la terminal y obtener el **flag root**:

![14. flag root.png](https://github.com/vonoHC/Writeups/blob/main/Breakout%20-%20VulnHub/Capturas/14.png)

**¡Y con esto ya habremos hecho nuestro el sistema de la máquina Breakout!**
