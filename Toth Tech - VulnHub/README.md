# Introducción
Thoth Tech es una máquina de dificultad fácil de VulnHub enfocada en técnicas básicas de enumeración, fuerza bruta de credenciales y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios FTP y SSH, el análisis de información filtrada en archivos accesibles mediante FTP anónimo y la obtención de credenciales válidas mediante un ataque de fuerza bruta con `Hydra`. Finalmente, se aprovecharán permisos inseguros de sudo sobre el binario `find` para obtener privilegios de root mediante técnicas de GTFOBins.

---

### **Reconocimiento**

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![1. ping.png](attachment:f1f7b864-08a5-4146-938b-0788b0f95181:311f5b83-aab0-431e-8eb1-10323136c0f1.png)

Continuamos con un escaneo **nmap** para enumerar los puertos abiertos y los servicios en ejecución dentro de la máquina:

![2. nmap.png](attachment:886bf5f3-61a5-4e6b-b509-05271a88dacf:b04ef1b6-deaf-4d1a-8460-6a61fbbb6c9e.png)

![2. nmap2.png](attachment:1136c6e3-22a5-4f79-b116-01a2e642c395:dadaa8b9-ac57-472e-a0dd-5885172a8505.png)

El escaneo nos muestra que el acceso anónimo a FTP está permitido. Ingresando vemos el archivo **note.txt** y lo descargamos en nuestra máquina:

![3. ftp.png](attachment:3a8726e2-b818-457a-a923-d44b03c4416c:957fc12e-af01-491e-829e-a5c7125e7684.png)

Veamos el contenido del archivo:

![4. cat note.png](attachment:5ddd6ec0-260a-4abb-96c1-94a776efc504:c3137612-5676-4fcf-b481-a03af4fa2ede.png)

Vemos que en el mensaje mostrado se menciona el nombre de usuario **pwnlab**, el cual tiene una contraseña débil, esto nos permite deducir que podríamos llevar a cabo un ataque de fuerza bruta para obtenerla. 

---

### Explotación

Vamos a intentar adivinar las credenciales para loguearnos en **SSH** con la herramienta **Hydra**:

![5. hydra.png](attachment:b70c53ca-0a11-40f2-8167-81fe838deaf8:65204efa-2f22-434d-a713-f70028a51179.png)

Como se muestra, **Hydra** halló la contraseña para iniciar sesión en **SSH** como el usuario **pwnlab**. Llegó el momento de acceder a la máquina:

![6. ssh.png](attachment:41162e9b-bd2d-4a94-9d1a-9d5eb3a12437:114a3e45-ebd2-4442-9885-5a0f7c172071.png)

![6. ssh2.png](attachment:f4f8b96e-dbe8-4b38-ac0d-69c8a7af4365:f51b042e-60fb-40c2-a79b-ff44a27743c9.png)

Pudimos acceder a la máquina, desde aquí podremos conseguir el **user flag**:

![7. userflag.png](attachment:a4c39d78-1dbf-44f9-b99d-f4d50727669b:b5e37ee5-cde2-44df-b6f2-6d8864c99e18.png)

---

### Post-Explotación/Escalada de privilegios

Llegó el momento de hallar una manera de elevar privilegios, y para ello vamos a ver si el usuario tiene permiso de ejecutar algún comando como sudo, con la instrucción: `sudo -l`

![8. sudo -l.png](attachment:b68348eb-c2b6-451c-801c-523ff9a79dc7:ded8c894-f6f0-4f2c-93c9-167dde23602c.png)

Podemos ejecutar el comando **find** como **sudo**, y sacando provecho de esto podremos obtener una shell como **root**. La instrucción que estaremos usando para volvernos **root** será:

```bash
sudo find . -exec /bin/sh \; -quit 
```

Una vez ejecutado el comando anterior habremos obtenido una shell como **root** y tendríamos acceso a la **root flag**:

![9. privesc y root.png](attachment:d4d232c6-ac92-4a12-8fef-643d295727e9:dab4e1ab-afc4-4894-bb49-af849f5329c6.png)

**¡Y con esto habremos hecho nuestro el sistema de la máquina Thoth Tech de VulnHub!**
