# Introducción
EasyCTF es una máquina de dificultad fácil de TryHackMe enfocada en técnicas de enumeración web, explotación de CMS vulnerables y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios FTP, HTTP y SSH, el análisis de información filtrada mediante FTP anónimo y el reconocimiento de un CMS Made Simple vulnerable a SQL Injection. Posteriormente, se obtendrán credenciales válidas para acceder al sistema a través de SSH y, finalmente, se aprovecharán permisos inseguros de sudo sobre el binario `vim` para conseguir privilegios de root mediante técnicas de GTFOBins.

---

### Reconocimiento

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![1. ping.png](attachment:c0c4495a-deeb-400a-9fc3-271be6df2fc2:b6bceeec-70ba-4f20-a7de-b4cfb57d2586.png)

Ahora realizaremos un scan usando **nmap** para enumerar los puertos y servicios activos en la máquina: 

![2. nmap 1.png](attachment:ee92d557-0bff-439f-b93b-5c4ef055fbd8:a817acbc-ea6e-4e4c-8df8-9f0ce0149135.png)

![2. nmap 2.png](attachment:a0827d9a-bcae-4d6d-9209-6c02c7fee9c2:797049e8-ec32-448f-808e-4984cfe59edf.png)

El escaneo nos muestra que **SSH** está corriendo en el puerto **2222** y **FTP** tiene el **acceso anónimo** habilitado, así que vamos a acceder para ver que encontramos: 

![3. ftp.png](attachment:4936f4dd-d102-401f-96a2-4eb4aa33f15d:e167b314-ce89-40cd-8206-f291b41e2bf7.png)

![4. get ftp.png](attachment:91e865bb-bc3e-4842-9e02-0f90f275abd7:a5baa7c7-9aa6-42ce-9717-5e854ed41841.png)

Luego de explorar un poco el servidor **FTP** encontramos el archivo **ForMitch.txt** el cual descargamos en nuestra máquina con el comando **get**. Vamos a ver su contenido:

![5. ForMitch.png](attachment:73361820-d060-4dc2-a936-dff225bd74b9:23418c9e-7991-44e9-b94e-aa33d4cf0fdc.png)

Vemos que menciona unas credenciales débiles de inicio de sesión, vamos a ver la página en nuestro navegador para ver si hallamos algo referente:

![6. pagina apache.png](attachment:27cb7715-17b1-4bb1-96d2-3d47719b23a9:efbaa41b-2e10-42d1-b394-7132efdb860d.png)

Vemos que es una página por defecto de apache, vamos a hacer una enumeración de directorios ocultos con **ffuf** para ver si logramos hallar algo más: 

![7. ffuf.png](attachment:893f57ee-43fc-4355-8a2a-bd7228fb187e:3b1795d8-a09f-4ac0-87bc-d06ed566247c.png)

Encontramos la ruta **/simple**, vamos a acceder a ella: 

![8. simple cms.png](attachment:6ad6282d-deef-4962-9e87-d52778806567:b0c58ee8-e64b-47a7-aa01-68435e637cb3.png)

---

### Explotación

Explorando un poco la página vemos que está motorizada por el **CMS Made Simple** version **2.2.8**, y haciendo una búsqueda rápida en Google hallamos un exploit de esta versión de **Made Simple**  la cual es vulnerable a **sqi**:

![9. exploit de Made Simple.png](attachment:c382debb-9d3c-4557-8cd7-beb77eb204dc:a7d5dead-b508-4f5c-b91a-bcc9d05fda0f.png)

Descargamos el exploit y lo ejecutamos junto a los parámetros que este solicita:

![10. ejecutamos el exploit.png](attachment:d5b1b31c-0912-473a-9d93-d45665921601:f3d35c5f-62b8-48b0-86cb-db6f2f0cf4d1.png)

Luego de un tiempo el exploit nos entrega varias credenciales de inicio de sesión: 

![11. resultado del exploit.png](attachment:1b85d09b-b60e-4f1a-b71c-fc2c6dcf5651:71cf552b-c09d-4778-9b5b-bf23f6d46812.png)

Ahora vamos a hacer una segunda enumeración de directorios ocultos con **ffuf** pero ahora añadiendo la ruta **/simple**: 

![12. ffuf simple.png](attachment:a167f7a4-b536-4d7e-be09-927fb0396933:c40564b6-7b30-46bb-8240-be1b7609f514.png)

![12. ffuf simple 2.png](attachment:d2eb23a3-6a36-4e1d-82ea-dff762213307:c6337896-b762-4745-83cb-e6909658f9c1.png)

Hallamos la ruta **/admin** la cual parece ser un login page, accedamos a ella e intentemos iniciar sesión con las credenciales que obtuvimos anteriormente:

![14. simple login incorrecto.png](attachment:50fb3e0a-4aeb-4453-a109-1dd1eac7c09d:5f08b16e-b027-4026-ab1d-d6bbc693f5e4.png)

Tal parece que las credenciales no funcionan en este panel de inicio de sesión. Pero revisando el archivo **ForMitch.txt** que obtuvimos de **FTP**, vemos que menciona que las credenciales de inicio de sesión son las mismas para el sistema, así que vamos a intentar ingresar a través de **SSH**: 

![15. ssh.png](attachment:579d8bfa-6699-4d44-b204-6af7966b49c7:15a38b4a-d75e-4b8b-86f5-28732204170f.png)

La deducción fue correcta, una vez ingresamos al sistema podremos encontrar el **user flag**:

![16. userflag.png](attachment:d4989a18-e230-4377-86b4-57c5bb42ec51:3fdd412b-b678-4d40-b219-a100e2a16480.png)

---

### Post-Explotación/Escalada de privilegios

Ahora tocaría hallar una forma de elevar privilegios, para ello vamos a listar los comando a los que el usuario actual tiene permiso de ejecutar como **root**, con el comando **`sudo -l`**:

![18. sudo -l.png](attachment:a5f93895-5a28-4e4a-a365-07afb084fa6b:2fb8f150-4a53-484e-833d-becad591eb8e.png)

Vemos que podemos ejecutar **vim** como **root**. Haciendo una búsqueda de este comando en el portal de **gtfobins** hallamos una manera de elevar privilegios: 

![19. gtfobins vim.png](attachment:ad67a7d3-79d9-45eb-901d-e0efa094a990:49d0dda5-16c5-4aeb-95b5-551501ee2724.png)

![sudo vim.png](attachment:584339da-05aa-400f-b6b5-a1acc1b24a89:1205fd92-643f-4d40-89ba-e7c615653c1d.png)

![20. vim.png](attachment:d7be670b-f6ec-41b8-8b32-4d6b199e675c:5accd6d5-87cc-4145-a0fa-4075fcdf67fd.png)

Una vez ejecutado los comandos anteriores habremos conseguido una shell como **root** y tendríamos acceso a la **root** **flag**:

![21. escalada y root flag.png](attachment:fb58faff-b657-4156-851a-ba0283cf043b:6b59d1af-1db2-4e05-8da7-9e13a1a758a6.png)

**¡Y con esto habremos hecho nuestro el sistema de la máquina EasyCTF de THM!**

![completeee.png](attachment:ad0b83a5-469e-4f1c-a936-ffe42520e299:completeee.png)
