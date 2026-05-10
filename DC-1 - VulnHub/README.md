# Introducción
DC-1 es una máquina de dificultad easy de VulnHub enfocada en técnicas de explotación web y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios HTTP y el reconocimiento de un CMS Drupal vulnerable. Posteriormente, se aprovechará un exploit de inyección SQL para obtener acceso administrativo al panel, permitiendo la ejecución de código PHP y la obtención de una reverse shell en el sistema. Finalmente, se realizará una escalada de privilegios mediante el abuso del binario `find` con permisos SUID para conseguir acceso como root.

---

## **Reconocimiento**

Como primer paso, obtenemos la **IP** de la máquina objetivo y le realizamos un **ping**. De esta forma, sabremos si está activa.

![1- ping.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/1.png)

Ahora que sabemos que la máquina está activa, realizaremos un escaneo con **Nmap**: 

![2- nmap.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/2.png)

Vemos que hay un servidor web corriendo en la máquina objetivo.

Accedemos a la página web a través del navegador: 

![3- pagina web - drupal.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/3.png)

Podemos ver que la página web es un panel de inicio de sesión, el cual está motorizado por el **CMS** **Drupal**, este podría ser el vector de ataque.

Ahora haremos uso de **whatweb** para ver cuál es la versión de **Drupal** utilizada en la página:

![4- whatweb.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/4.png)

Gracias a **whatweb** ahora sabemos que el **CMS** utilizado en la página es **Drupal 7**, por lo que ahora haremos una búsqueda de exploits conocidos con **searchsploit**:

![5- searchsploit.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/5.png)

Hemos encontrado un exploit muy interesante, ya que permite añadir una cuenta admin a la base de datos de **Drupal** a través de **SQI**. 

---

## **Explotación**

Ahora descargamos el exploit con el comando `searchsploit -m php/webapps/34992.py`  y lo ejecutamos, especificando los parámetros requeridos por este: 

![6- exploit 1.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/6.png)

![6- exploit 2.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/6.5.png)

La creación de la cuenta admin fue exitosa. 

Vamos a iniciar sesión en la página: 

![7- inicio de sesion como admin.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/7.png)

Navegando un poco por la página logramos encontrar un apartado que permite añadir contenido, el cual tal vez podríamos usar para inyectar código **PHP** y obtener una **[Reverse Shell](https://www.notion.so/Reverse-shell-24dd30e24492807e99c1ce25f9d96d03?pvs=21)** si se nos permite:

![8- add content.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/8.png)

Vemos que en los formatos de texto no está incluido **PHP**, pero ya que somos administradores de la página tal vez podamos cambiar las configuraciones para que se permita la subida de **PHP**: 

![9- php filter - modulo.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/9.png)

![10- usar php como formato - config.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/10.png)

Encontramos un par de apartados y configuraciones con los cuales podemos añadir el formato de texto **PHP**.

Ahora vamos a volver a la pestaña de **add content** para ver si el cambio surtió efecto:

![11- habilitacion de php exitosa.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/11.png)

Como vemos, se ha habilitado la opción de **PHP**.

Vamos a pegar una [**Reverse Shell**](https://www.notion.so/Reverse-shell-24dd30e24492807e99c1ce25f9d96d03?pvs=21) de **PHP** en este campo y de paso también iniciaremos una conexión en escucha con **netcat** en nuestra terminal: 

![12- inyeccion de reverse shell.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/12.png)

![13- netcat.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/13.png)

Subimos la página que contiene la **[Reverse Shell](https://www.notion.so/Reverse-shell-24dd30e24492807e99c1ce25f9d96d03?pvs=21)** y ya obtendríamos acceso a la terminal de la máquina: 

![14- acceso al sistema gracias a la rv shell.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/14.png)

---

## Post-Explotación/Escalada de Privilegios

Ya que conseguimos acceso a la máquina sería momento de escalar privilegios de alguna forma, así que vamos a hacer una búsqueda de binarios con permisos especiales: 

![15- busqueda de archivos con permisos especiales para elevar privilegios.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/15.png)

![16- comprobacion de que find tiene permiso suid.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/16.png)

Encontramos el binario **/usr/bin/find** el cual tiene permisos **suid** y pertenece al usuario **root,** por lo que esta es la brecha por la que podremos escalar privilegios, y lo haremos con el siguiente comando: 

```bash
find . -exec /bin/sh \; -quite 
```

![17- escalada de privilegios exitosa.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/17.png)

Una vez ejecutado el comando seriamos **root**, y ****ya tendremos acceso a la última flag: 

![18- root flag.png](https://github.com/vonoHC/Writeups/blob/main/DC-1%20-%20VulnHub/Capturas/18.png)

**¡Y con esto ya habremos hecho nuestro el sistema de la máquina DC-1!**
