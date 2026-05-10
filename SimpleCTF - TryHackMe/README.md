# Introducción
EasyCTF es una máquina de dificultad fácil de TryHackMe enfocada en técnicas de enumeración web, explotación de CMS vulnerables y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios FTP, HTTP y SSH, el análisis de información filtrada mediante FTP anónimo y el reconocimiento de un CMS Made Simple vulnerable a SQL Injection. Posteriormente, se obtendrán credenciales válidas para acceder al sistema a través de SSH y, finalmente, se aprovecharán permisos inseguros de sudo sobre el binario `vim` para conseguir privilegios de root mediante técnicas de GTFOBins.

---

## Reconocimiento

Como primer paso, obtenemos la **IP** de la máquina objetivo y le realizamos un **ping**. De esta forma sabremos si está activa.

![1. ping.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/1.png)

Ahora realizaremos un scan usando **nmap** para enumerar los puertos y servicios activos en la máquina: 

![2. nmap 1.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/2.png)

![2. nmap 2.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/2.5.png)

El escaneo nos muestra que **SSH** está corriendo en el puerto **2222** y **FTP** tiene el **acceso anónimo** habilitado, así que vamos a acceder para ver que encontramos: 

![3. ftp.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/3.png)

![4. get ftp.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/4.png)

Luego de explorar un poco el servidor **FTP** encontramos el archivo **ForMitch.txt** el cual descargamos en nuestra máquina con el comando **get**. Vamos a ver su contenido:

![5. ForMitch.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/5.png)

Vemos que menciona unas credenciales débiles de inicio de sesión, vamos a ver la página en nuestro navegador para ver si hallamos algo referente:

![6. pagina apache.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/6.png)

Vemos que es una página por defecto de apache, vamos a hacer una enumeración de directorios ocultos con **ffuf** para ver si logramos hallar algo más: 

![7. ffuf.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/7.png)

Encontramos la ruta **/simple**, vamos a acceder a ella: 

![8. simple cms.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/8.png)

---

## Explotación

Explorando un poco la página vemos que está motorizada por el **CMS Made Simple** version **2.2.8**, y haciendo una búsqueda rápida en Google hallamos un exploit de esta versión de **Made Simple**  la cual es vulnerable a **sqi**:

![9. exploit de Made Simple.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/9.png)

Descargamos el exploit y lo ejecutamos junto a los parámetros que este solicita:

![10. ejecutamos el exploit.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/10.png)

Luego de un tiempo el exploit nos entrega varias credenciales de inicio de sesión: 

![11. resultado del exploit.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/11.png)

Ahora vamos a hacer una segunda enumeración de directorios ocultos con **ffuf** pero ahora añadiendo la ruta **/simple**: 

![12. ffuf simple.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/12.png)

![12. ffuf simple 2.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/13.png)

Hallamos la ruta **/admin** la cual parece ser un login page, accedamos a ella e intentemos iniciar sesión con las credenciales que obtuvimos anteriormente:

![14. simple login incorrecto.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/14.png)

Tal parece que las credenciales no funcionan en este panel de inicio de sesión. Pero revisando el archivo **ForMitch.txt** que obtuvimos de **FTP**, vemos que menciona que las credenciales de inicio de sesión son las mismas para el sistema, así que vamos a intentar ingresar a través de **SSH**: 

![15. ssh.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/15.png)

La deducción fue correcta, una vez ingresamos al sistema podremos encontrar el **user flag**:

![16. userflag.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/16.png)

---

## Post-Explotación/Escalada de privilegios

Ahora tocaría hallar una forma de elevar privilegios, para ello vamos a listar los comando a los que el usuario actual tiene permiso de ejecutar como **root**, con el comando **`sudo -l`**:

![18. sudo -l.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/17.png)

Vemos que podemos ejecutar **vim** como **root**. Haciendo una búsqueda de este comando en el portal de **gtfobins** hallamos una manera de elevar privilegios: 

![19. gtfobins vim.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/18.png)

![sudo vim.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/19.png)

![20. vim.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/20.png)

Una vez ejecutado los comandos anteriores habremos conseguido una shell como **root** y tendríamos acceso a la **root** **flag**:

![21. escalada y root flag.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/21.png)

**¡Y con esto habremos hecho nuestro el sistema de la máquina EasyCTF!**

![completeee.png](https://github.com/vonoHC/Writeups/blob/main/SimpleCTF%20-%20TryHackMe/Capturas/Pwned.png)
