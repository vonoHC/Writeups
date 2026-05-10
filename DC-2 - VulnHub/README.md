# Introducción
DC-2 es una máquina de dificultad easy de VulnHub enfocada en técnicas de enumeración web, fuerza bruta de credenciales y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios HTTP y SSH, el reconocimiento de un entorno WordPress y la generación de wordlists personalizadas mediante `cewl`. Posteriormente, se realizará un ataque de fuerza bruta con `wpscan` para obtener credenciales válidas, permitiendo el acceso al sistema mediante SSH. Finalmente, se abusará de configuraciones inseguras relacionadas con los binarios `vi` y `git` para escapar de una shell restringida y conseguir privilegios de root mediante técnicas de GTFOBins.


---

## Reconocimiento

Como primer paso, obtenemos la **IP** de la máquina objetivo y le realizamos un **ping**. De esta forma, sabremos si está activa.

![1. ping.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/1.png)

Continuamos con un escaneo **nmap** para enumerar los puertos abiertos y los servicios en ejecución dentro de la máquina:

![2.nmap.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/2.png)

![2.nmap 2.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/2.5.png)

El resultado de **nmap** nos muestra que además de **ssh** el cual está corriendo en el puerto **7744**, tenemos un servicio web, el cual tiene **virtual host** y nos redirige a la página **dc-2**, por lo que debemos añadir ese dominio a nuestro archivo **/etc/hosts** para que nuestra máquina pueda acceder a la máquina:

![3. nano et hosts.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/3.png)

![3. nano etc hosts 2.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/3.5.png)

Una vez hecho esto podremos acceder a la página web:

![4. pagina.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/4.png)

Navegando por la página encontramos el **primer flag**:

![5. flag.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/5.png)

Además de hablarnos sobre el inicio de sesión, lo que indica que existe una página de login, también se menciona la herramienta **cewl** como pista. Por ello, la utilizaremos para generar una wordlist de contraseñas que emplearemos en un próximo ataque de fuerza bruta. Antes de eso, realizaremos una enumeración de directorios ocultos con **ffuf**.

![6. ffuf.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/6.png)

En los resultados vemos la ruta **wp-admin**, la cual nos lleva a la página de login:

![7. wp-admin.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/7.png)

---

## Explotacion

Ya que sabemos en qué página debemos iniciar sesión, haremos uso de la herramienta **cewl** para crear un wordlist de contraseñas y guardarlo en el archivo **passwd.txt** para un próximo ataque de fuerza bruta: 

![8. cewl.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/8.png)

![8. cewl cat.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/8.5.png)

Una vez generada la wordlist de contraseñas haremos uso de la herramienta **wpscan** para hacer una enumeración de nombres de usuarios: 

![9. enum de u.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/9.png)

![9. enum de u 2.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/9.5.png)

El resultado nos muestra que existen 3 usuarios: **admin, jerry y tom**. Ahora haremos un archivo con estos nombres de usuarios para usarlo como wordlist para el ataque de fuerza bruta:

![10. creacion de wordlist de usr.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/10.png)

Ahora iniciaremos con el ataque de fuerza bruta haciendo uso de la herramienta **wpscan**:

![11. fuerza bruta con wspscan.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/11.png)

![11. resultados de fuerza  bruta.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/11.5.png)

Los resultados nos muestran los usuarios con los que podríamos iniciar sesión y sus respectivas contraseñas. Vamos a iniciar sesión con el usuario **jerry**: 

![12. inicio de sesión como jerry.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/12.png)

Luego de haber iniciado sesión vemos un panel de administración de la página, explorándolo un poco hallamos el **flag2**:

![13. flag2.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/13.png)

El flag nos da una pista para arrancar sesión a través de **SSH** con las credenciales que obtuvimos. Vamos a ello:

![14. ssh.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/14.png)

Pudimos iniciar sesión como el usuario **tom**, ahora es momento de conseguir el siguiente flag:

![15. rbash.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/15.png)

Como vemos, tenemos como shell una Bash **restringida**, la cual nos limita el uso de comandos. Tenemos que hallar una forma de conseguir una **bash** normal. Y explorando un poco la ubicación actual descubrimos que podemos ejecutar el comando **vi**:

![16. descubrimiento de vi.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/16.png)

Si buscamos este comando en el portal de **gtfobins** vemos que hay una forma de aprovecharnos de este para conseguir una shell:

![17. gtfobins vi.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/17.png)

Una vez ejecutados los comandos mostrados obtendremos una nueva shell, en la cual tendremos que editar la variable **PATH** para poder ejecutar los comandos plenamente:

![18. editar ruta PATH.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/18.png)

Luego de hacer esto, seremos capaces de ver el contenido del **flag3**:

![19. flag3.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/19.png)

---

## Post-Explotación

Ahora vamos a intentar cambiar al usuario **jerry** con las credenciales que conseguimos anteriormente: 

![20. su jerry.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/20.png)

Ahora seremos capaces de leer el **flag4**, el cual se encuentra en el directorio personal de **jerry**:

![21. flag4.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/21.png)

El flag nos menciona **git** como pista, así que probablemente a través de este podamos escalar privilegios a **root**, vamos a comprobarlo ejecutando el comando: `sudo -l`, para listar los comandos que el usuario actual tiene permiso de ejecutar como **root**:

![22. descubrimiento de git.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/22.png)

La suposición es correcta. Haciendo una búsqueda del comando **git** en el portal de **gtfobins** hallamos una manera de aprovecharnos de este para poder volvernos **root**:

![23. gtfobins git.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/23.png)

Una vez ejecutados los comandos anteriores nos habremos vuelto **root**:

![24. escalada de privilegios.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/24.png)

Ahora ya tendríamos acceso a nuestro objetivo, la **root flag**:

![25. root flag.png](https://github.com/vonoHC/Writeups/blob/main/DC-2%20-%20VulnHub/Capturas/25.png)

**¡Y con esto ya habremos hecho nuestro el sistema de la máquina DC-2!**
