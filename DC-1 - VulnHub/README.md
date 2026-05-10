# Introducción
DC-1 es una máquina de dificultad media de VulnHub enfocada en técnicas de explotación web y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios HTTP y el reconocimiento de un CMS Drupal vulnerable. Posteriormente, se aprovechará un exploit de inyección SQL para obtener acceso administrativo al panel, permitiendo la ejecución de código PHP y la obtención de una reverse shell en el sistema. Finalmente, se realizará una escalada de privilegios mediante el abuso del binario `find` con permisos SUID para conseguir acceso como root.

---

### **Reconocimiento**

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![1- ping.png](attachment:6dedbc34-76bc-494d-a992-75fcbcd66b18:8498f68c-b8a9-40a7-bff8-3f82ffb0bdc5.png)

Ahora que sabemos que la máquina está activa realizaremos un escaneo con **Nmap**: 

![2- nmap.png](attachment:f075a6df-b096-4d75-82e4-ecc1a9a23e54:c993c8f0-495b-4e0e-be1a-10de6793d4f9.png)

Vemos que hay un servidor web corriendo en la máquina objetivo.

Accedemos a la página web a través del navegador: 

![3- pagina web - drupal.png](attachment:48bdc38d-cc61-4078-97fb-2806e2696b39:a47ea42b-61b1-4eff-b36a-ecc2428dd162.png)

Podemos ver que la página web es un panel de inicio de sesión, el cual está motorizado por el **CMS** **Drupal**, este podría ser el vector de ataque.

Ahora haremos uso de **whatweb** para ver cuál es la versión de **Drupal** utilizada en la página:

![4- whatweb.png](attachment:2ac16563-fc08-4a0f-8566-1e3e19f37fcb:f01f4d2c-0fa1-43c7-a14e-f41f26d15b3f.png)

Gracias a **whatweb** ahora sabemos que el **CMS** utilizado en la página es **Drupal 7**, por lo que ahora haremos una búsqueda de exploits conocidos con **searchsploit**:

![5- searchsploit.png](attachment:b966ba1a-58d5-4381-9969-8c26b750ad10:851dd26e-f480-4a4f-9845-1432b2ed0ee9.png)

Hemos encontrado un exploit muy interesante, ya que permite añadir una cuenta admin a la base de datos de **Drupal** a través de **SQI**. 

---

### **Explotación**

Ahora descargamos el exploit con el comando `searchsploit -m php/webapps/34992.py`  y lo ejecutamos, especificando los parámetros requeridos por este: 

![6- exploit 1.png](attachment:4695ba9b-d251-4c1f-9791-02ea9e191033:3052807c-0122-490a-b1ab-ab283300fe9e.png)

![6- exploit 2.png](attachment:b7b63158-1f0e-45b0-915c-3dcf16e7b5f0:cfc0a056-56a0-44c9-845d-ea2d47171878.png)

La creación de la cuenta admin fue exitosa. 

Vamos a iniciar sesión en la página: 

![7- inicio de sesion como admin.png](attachment:3d8efa60-ddfa-4940-982e-47f982f410cd:757f03a2-e18b-44f5-9824-3f4343f764aa.png)

Navegando un poco por la página logramos encontrar un apartado que permite añadir contenido, el cual tal vez podríamos usar para inyectar código **PHP** y obtener una **[Reverse Shell](https://www.notion.so/Reverse-shell-24dd30e24492807e99c1ce25f9d96d03?pvs=21)** si se nos permite:

![8- add content.png](attachment:d52ff968-1737-43ba-a8e0-dc5fbe80273c:ba7187e8-1c6f-496d-a3aa-6308720e4bdf.png)

Vemos que en los formatos de texto no está incluido **PHP**, pero ya que somos administradores de la página tal vez podamos cambiar las configuraciones para que se permita la subida de **PHP**: 

![9- php filter - modulo.png](attachment:77704fa1-4915-4bdd-b571-2481afafd1b5:ec884728-7a25-4803-b57c-80c13e2e0ecc.png)

![10- usar php como formato - config.png](attachment:d74cb9d6-ab90-4edb-9e42-549d25166a47:017f085e-773a-4ee2-928f-0bc6bab0acf0.png)

Encontramos un par de apartados y configuraciones con los cuales podemos añadir el formato de texto **PHP**.

Ahora vamos a volver a la pestaña de **add content** para ver si el cambio surtió efecto:

![11- habilitacion de php exitosa.png](attachment:af2871a3-9e46-4219-9324-e91555bc3c49:fcf8282e-c599-4c4c-b38b-0aa8052aa6c6.png)

Como vemos, se ha habilitado la opción de **PHP**.

Vamos a pegar una [**Reverse Shell**](https://www.notion.so/Reverse-shell-24dd30e24492807e99c1ce25f9d96d03?pvs=21) de **PHP** en este campo y de paso también iniciaremos una conexión en escucha con **netcat** en nuestra terminal: 

![12- inyeccion de reverse shell.png](attachment:b5759ee1-c21b-4b43-a43a-b54469f516f7:66d9cc00-bd17-4084-b8fe-a8df6b790d48.png)

![13- netcat.png](attachment:364248ff-d269-40f6-a51d-b691d8f1f6e6:755a66c6-e4a7-46bf-8300-b4d0d4427739.png)

Subimos la página que contiene la **[Reverse Shell](https://www.notion.so/Reverse-shell-24dd30e24492807e99c1ce25f9d96d03?pvs=21)** y ya obtendríamos acceso a la terminal de la máquina: 

![14- acceso al sistema gracias a la rv shell.png](attachment:b8e7920e-c866-47bf-a531-c9a47aedf842:9adc7eea-2614-4500-8e20-5dbf372a1898.png)

---

### Post-Explotación/Escalada de Privilegios

Ya que conseguimos acceso a la máquina sería momento de escalar privilegios de alguna forma, así que vamos a hacer una búsqueda de binarios con permisos especiales: 

![15- busqueda de archivos con permisos especiales para elevar privilegios.png](attachment:dd063c2e-a277-4aa5-8ab7-28e863961692:7584f219-44e5-43b9-944f-f1a3b3752928.png)

![16- comprobacion de que find tiene permiso suid.png](attachment:7c88fee1-9705-4103-8bde-ae74746d49ca:c5700a57-8725-4fb4-8d61-64b6b5207492.png)

Encontramos el binario **/usr/bin/find** el cual tiene permisos **suid** y pertenece al usuario **root,** por lo que esta es la brecha por la que podremos escalar privilegios, y lo haremos con el siguiente comando: 

```bash
find . -exec /bin/sh \; -quite 
```

![17- escalada de privilegios exitosa.png](attachment:ac601878-053d-487c-a4c0-00dd1c32657b:930fa9ac-6c3a-4f0d-afb9-33c93e5d9f43.png)

Una vez ejecutado el comando seriamos **root**, y ****ya tendremos acceso a la última flag: 

![18- root flag.png](attachment:88ce3910-d34f-4694-adbb-566438c1974c:d7cb1ccc-df93-4bda-985f-dd690759e731.png)

**¡Y con esto ya habremos hecho nuestro el sistema de la máquina DC-1!**
