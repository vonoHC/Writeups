# Introducción
Thoth Tech es una máquina de dificultad fácil de VulnHub enfocada en técnicas básicas de enumeración, fuerza bruta de credenciales y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios FTP y SSH, el análisis de información filtrada en archivos accesibles mediante FTP anónimo y la obtención de credenciales válidas mediante un ataque de fuerza bruta con **Hydra**. Finalmente, se aprovecharán permisos inseguros de sudo sobre el binario **find** para obtener privilegios de root mediante técnicas de GTFOBins.

---

## **Reconocimiento**

Como primer paso, obtenemos la **IP** de la máquina objetivo y le realizamos un **ping**. De esta forma, sabremos si está activa.

![1. ping.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/1.png)

Continuamos con un escaneo **nmap** para enumerar los puertos abiertos y los servicios en ejecución dentro de la máquina:

![2. nmap.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/2.png)

![2. nmap2.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/2.5.png)

El escaneo nos muestra que el acceso anónimo a FTP está permitido. Ingresando vemos el archivo **note.txt** y lo descargamos en nuestra máquina:

![3. ftp.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/3.png)

Veamos el contenido del archivo:

![4. cat note.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/4.png)

Vemos que en el mensaje mostrado se menciona el nombre de usuario **pwnlab**, el cual tiene una contraseña débil, esto nos permite deducir que podríamos llevar a cabo un ataque de fuerza bruta para obtenerla. 

---

## Explotación

Vamos a intentar adivinar las credenciales para loguearnos en **SSH** con la herramienta **Hydra**:

![5. hydra.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/5.png)

Como se muestra, **Hydra** halló la contraseña para iniciar sesión en **SSH** como el usuario **pwnlab**. Llegó el momento de acceder a la máquina:

![6. ssh.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/6.png)

![6. ssh2.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/6.5.png)

Pudimos acceder a la máquina, desde aquí podremos conseguir el **user flag**:

![7. userflag.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/7.png)

---

## Post-Explotación

Llegó el momento de hallar una manera de elevar privilegios, y para ello vamos a ver si el usuario tiene permiso de ejecutar algún comando como sudo, con la instrucción: `sudo -l`

![8. sudo -l.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/8.png)

Podemos ejecutar el comando **find** como **sudo**, y sacando provecho de esto podremos obtener una shell como **root**. La instrucción que estaremos usando para volvernos **root** será:

```bash
sudo find . -exec /bin/sh \; -quit 
```

Una vez ejecutado el comando anterior habremos obtenido una shell como **root** y tendríamos acceso a la **root flag**:

![9. privesc y root.png](https://github.com/vonoHC/Writeups/blob/main/Toth%20Tech%20-%20VulnHub/Capturas/9.png)

**¡Y con esto habremos hecho nuestro el sistema de la máquina Thoth Tech de VulnHub!**
