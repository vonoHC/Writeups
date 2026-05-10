# Introducción
**Cap** es una máquina de dificultad Easy en Linux que se centra en la explotación de vulnerabilidades web comunes y el abuso de configuraciones incorrectas en el sistema operativo. El vector de ataque inicial consiste en identificar una vulnerabilidad de IDOR (Insecure Direct Object Reference) en un panel de gestión de red. A través de esta debilidad, es posible acceder a capturas de tráfico (PCAP) generadas por otros usuarios, las cuales contienen credenciales en texto claro de servicios inseguros.

Una vez obtenida la intrusión mediante SSH, la escalada de privilegios se basa en la enumeración de Linux Capabilities. En este escenario, abusaremos de una capacidad inusual asignada al binario de Python, lo que nos permitirá manipular el UID del proceso para obtener una terminal como el usuario administrador de forma inmediata.

---

## Reconocimiento

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![**ping** nos devolvió una respuesta positiva, excelente!](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/1.png)

**ping** nos devolvió una respuesta positiva, excelente!

Ahora empezaremos con la etapa de reconocimiento, llevando a cabo un escaneo inicial con **nmap**. Estos son los parámetros que estaremos usando para ello:

```bash
nmap (ip) -sS -Pn -vvv -p- --open --min-rate 5000 
```

![**nmap** nos devolvió información interesante. Gracias al resultado del escaneo ahora sabemos que en la máquina objetivo hay un total de 3 puertos abiertos: **21**, **22** y **80**, con los servicios activos: **FTP**, **SSH** y **HTTP** respectivamente. ](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/2.png)

**nmap** nos devolvió información interesante. Gracias al resultado del escaneo ahora sabemos que en la máquina objetivo hay un total de 3 puertos abiertos: **21**, **22** y **80**, con los servicios activos: **FTP**, **SSH** y **HTTP** respectivamente. 

Ahora que sabemos qué puertos y servicios están corriendo en la máquina vulnerable, es hora de hacer un escaneo  exhaustivo con **nmap,** dirigido a los puertos descubiertos para recopilar más información sobre la máquina. Los parámetros que usaremos para este tipo de escaneo, serán:

```bash
nmap (ip) -sS -sCV -Pn -p 21,22,80 --mi-rate 5000 -vvv
```

![Screenshot 2025-08-06 154422.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/3.png)

![Como vemos, el resultado nos muestra información adicional de los distintos servicios activos en la máquina. Podemos descartar la posibilidad de acceder a la máquina a través de **FTP** por ahora, ya que el acceso anónimo está deshabilitado. ](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/4.png)

Como vemos, el resultado nos muestra información adicional de los distintos servicios activos en la máquina. Podemos descartar la posibilidad de acceder a la máquina a través de **FTP** por ahora, ya que el acceso anónimo está deshabilitado. 

Puesto que verificamos que en la máquina objetivo está corriendo **HTTP**, lo cual hace referencia a un servidor web, vamos a intentar acceder a la página a través del navegador:

![¡Bien! Pudimos acceder a la página web. Y tal parece que esta página es un panel de seguridad de alguna empresa.](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/5.png)

¡Bien! Pudimos acceder a la página web. Y tal parece que esta página es un panel de seguridad de alguna empresa.

Navegando un poco por la página, vemos que está la pestaña llamada **Security Snapshot** y dentro de esta, al parecer, se guardan archivos ****con terminación **.pcap** (archivos [**wireshark**](https://www.innovaciondigital360.com/iot/que-es-wireshark-y-casos-de-uso/)).

![Screenshot 2025-08-06 160331.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/6.png)

Ahora descargamos el archivo **.pcap** para ver si podemos hallar algo interesante dentro de los paquetes del tráfico de red almacenado en él:

![Mmm, qué raro, dentro del archivo **.pcap** no hay nada.] ](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/7.png)

Mmm, qué raro, dentro del archivo **.pcap** no hay nada. 

Volvamos a la página web y sigamos investigando.

![Hallamos algo interesante. Al ingresar a la pestaña **Security Snapshot**, en la URL aparecen nuevos parámetros: /data/9. Y tal parece que el **9** es un identificador de contenido para los archivos **.pcap**. ](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/8.png)

Hallamos algo interesante. Al ingresar a la pestaña **Security Snapshot**, en la URL aparecen nuevos parámetros: /data/9. Y tal parece que el **9** es un identificador de contenido para los archivos **.pcap**. 

---

## Explotación

Ahora intentemos manipular el identificador en la URL colocando distintos números en el campo para ver si podemos ver otros archivos **.pcap** (**Insecure Direct Object Reference** (**IDOR**)).

![Screenshot 2025-08-06 161504.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/9.png)

Cambiamos el **id** **9** por **0** y ahora se nos presenta un archivo **.pcap** con mayor número de paquetes, vamos a descargarlo y a revisarlo: 

![Wow! En este sí se presenta mucha información sobre los paquetes. ](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/10.png)

Wow! En este sí se presenta mucha información sobre los paquetes. 

Veamos si en alguno de estos podemos hallar información importante:

![Screenshot 2025-08-06 162359.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/11.png)

¡Bingo! Hallamos dos paquetes que parecen tener credenciales de inicio de sesión a **FTP**: el **No. 36** nos proporciona el nombre de usuario y el **No. 40** la contraseña.  

Con estas credenciales podemos iniciar sesión a través de ssh: 

![1- ssh.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/12.png)

![2- ssh 2.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/13.png)

Ya que conseguimos acceder a la máquina a través **SSH** podremos encontrar la **user flag**:

![3- userflag.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/14.png)

---

### Post-Explotación/Escalada de privilegios

Ahora es momento de escalar privilegios, por lo que vamos a hacer una búsqueda de binarios con capacidades especiales con el siguiente comando: 

```bash
getcap -r / 2>/dev/null
```

![4- getcap.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/15.png)

Encontramos el binario de **python**, el cual tiene la capacidad especial **setuid**, la cual permite ejecutarlo con los permisos de **root**, por lo que podremos conseguir una shell como este usuario de forma automática, con unos pocos comandos.

Ahora vamos a ejecutar **python** y cambiaremos el **suid** a **0**, lo cual es equivalente a **root**:

![7- Verificación de escalada exitosa.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/16.png)

Solo nos falta una **Shell** como **root**, la cual podemos conseguir fácilmente con el siguiente comando:

```bash
os.system(”/bin/bash”)
```

![8- terminal como root.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/17.png)

Ya con esto podríamos conseguir el **root** flag: 

![9- flag root.png](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/18.png)

**¡Y con esto habremos hecho nuestro el sistema de la máquina Cap!**

![Pwned!](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Cap/Capturas/19.png)
