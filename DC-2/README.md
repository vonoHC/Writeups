# Introducción
DC-2 es una máquina de dificultad media de VulnHub enfocada en técnicas de enumeración web, fuerza bruta de credenciales y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios HTTP y SSH, el reconocimiento de un entorno WordPress y la generación de wordlists personalizadas mediante `cewl`. Posteriormente, se realizará un ataque de fuerza bruta con `wpscan` para obtener credenciales válidas, permitiendo el acceso al sistema mediante SSH. Finalmente, se abusará de configuraciones inseguras relacionadas con los binarios `vi` y `git` para escapar de una shell restringida y conseguir privilegios de root mediante técnicas de GTFOBins.


---

### Reconocimiento

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![1. ping.png](attachment:617f71e2-6dc0-447e-8dcd-4d500d80a44a:0dc53a43-310c-4def-8c30-b85367f6d52d.png)

Continuamos con un escaneo **nmap** para enumerar los puertos abiertos y los servicios en ejecución dentro de la máquina:

![2.nmap.png](attachment:12e989a2-420f-4d18-97bc-3a3d8c6813a4:dce5bc9a-f04b-4bab-aca4-cfae381c8374.png)

![2.nmap 2.png](attachment:e91313c6-515d-42a5-94e4-38532e85c38c:2ff70606-6118-451e-8103-59f948d0a370.png)

El resultado de **nmap** nos muestra que además de **ssh** el cual está corriendo en el puerto **7744**, tenemos un servicio web, el cual tiene **virtual host** y nos redirige a la página **dc-2**, por lo que debemos añadir ese dominio a nuestro archivo **/etc/hosts** para que nuestra máquina pueda acceder a la máquina:

![3. nano et hosts.png](attachment:ad254408-ffdb-4536-8a78-30397781e204:d2cc9718-c545-4de7-87d8-513a30b46287.png)

![3. nano etc hosts 2.png](attachment:107b34bf-25a5-4a19-8879-fcc01f5f27d5:31ef4d9b-e28c-4b94-bf9c-1d2cd8c9a46f.png)

Una vez hecho esto podremos acceder a la página web:

![4. pagina.png](attachment:32872990-7045-4ac6-a8e9-ba3731469618:70f43091-0b20-4313-9b3c-1bf12fb7669b.png)

Navegando por la página encontramos el **primer flag**:

![5. flag.png](attachment:f2a7a488-4e11-4422-9b5a-8971b91d82ea:e49e684c-1d98-4418-a024-8eeeca9a4d77.png)

Además de hablarnos sobre el inicio de sesión, lo que indica que existe una página de login, también se menciona la herramienta **cewl** como pista. Por ello, la utilizaremos para generar una wordlist de contraseñas que emplearemos en un próximo ataque de fuerza bruta. Antes de eso, realizaremos una enumeración de directorios ocultos con **ffuf**.

![6. ffuf.png](attachment:1cdca8f3-f00c-4f3b-b4e4-863f0aea2e92:91858955-ab90-42e7-8a98-a0ba0400d90f.png)

En los resultados vemos la ruta **wp-admin**, la cual nos lleva a la página de login:

![7. wp-admin.png](attachment:fa36e887-0d3a-4c0e-93a7-ed7a73acab6d:3c0a9d6a-6abd-4888-b376-4dfb0689c609.png)

---

### Explotacion

Ya que sabemos en qué página debemos iniciar sesión, haremos uso de la herramienta **cewl** para crear un wordlist de contraseñas y guardarlo en el archivo **passwd.txt** para un próximo ataque de fuerza bruta: 

![8. cewl.png](attachment:56e78160-dc83-44b5-9e89-27bf4f44e057:2df34f68-01d5-4a18-ac28-8f3dd1f31882.png)

![8. cewl cat.png](attachment:982f851f-3819-4f20-bd13-8deb9348653e:6017c9c8-7abb-40ad-ad18-7c61cfcdd3b4.png)

Una vez generado la wordlist de contraseñas haremos uso de la herramienta **wpscan** para hacer una enumeración de nombres de usuarios: 

![9. enum de u.png](attachment:1acef269-5fbf-4281-a105-827de0971790:9b6ae520-ead9-481a-9f41-a0443a0909c8.png)

![9. enum de u 2.png](attachment:2dd0293e-fb16-4561-b853-1f96781ac746:2f7f206d-7418-4864-b5cb-b563949e0afb.png)

El resultado nos muestra que existen 3 usuarios: **admin, jerry y tom**. Ahora haremos un archivo con estos nombres de usuarios para usarlo como wordlist para el ataque de fuerza bruta:

![10. creacion de wordlist de usr.png](attachment:8a83af2a-10ed-4d3e-a3ce-9fbf21cb2afe:1478efb9-b65e-427e-a78f-c2c65c37c90a.png)

Ahora iniciaremos con el ataque de fuerza bruta haciendo uso de la herramienta **wpscan**:

![11. fuerza bruta con wspscan.png](attachment:019ee6a8-37ea-499c-8ae8-b091a18ce015:3f301150-44a1-4a04-b18e-d28228b681fb.png)

![11. resultados de fuerza  bruta.png](attachment:dd0e0a86-6eac-40d5-99bc-c076893c2491:10e6626c-99ec-4144-bd29-7d171e66410b.png)

Los resultados nos muestran los usuarios con los que podríamos iniciar sesión y sus respectivas contraseñas. Vamos a iniciar sesión con el usuario **jerry**: 

![12. inicio de sesion como jerry.png](attachment:ce679700-d5bf-4880-ab99-4656556e46a5:89d0a9a4-6fda-4fb6-a754-9fcc748ee7fd.png)

Luego de haber iniciado sesión vemos un panel de administración de la página, explorándolo un poco hallamos el **flag2**:

![13. flag2.png](attachment:de2fc94e-3b5b-4982-85be-4d775ab2017e:ce17b660-f6f7-4e22-b676-44b2d22efba2.png)

El flag nos da una pista para arrancar sesión a través de **SSH** con las credenciales que obtuvimos. Vamos a ello:

![14. ssh.png](attachment:45cda275-3398-447b-8211-15db91625261:4c0ae3ac-ab09-413e-9ed0-f209adebc7fc.png)

Pudimos iniciar sesión como el usuario **tom**, ahora es momento de conseguir el siguiente flag:

![15. rbash.png](attachment:5c8e25ea-5ac4-4257-be31-e05a61eb8281:1d9cd72f-af35-4567-b47f-a1f0c6841e88.png)

Como vemos, tenemos como shell una Bash **restringida**, la cual nos limita el uso de comandos. Tenemos que hallar una forma de conseguir una **bash** normal. Y explorando un poco la ubicación actual descubrimos que podemos ejecutar el comando **vi**:

![16. descubrimiento de vi.png](attachment:bad04221-94fb-44db-aff7-be53bf80ef02:2acd8a65-eb73-426d-9226-75fc2b7cd9ce.png)

Si buscamos este comando en el portal de **gtfobins** vemos que hay una forma de aprovecharnos de este para conseguir una shell:

![17. gtfobins vi.png](attachment:8824ea4d-33c2-42b7-8901-0e83496e1198:ec8eba67-2046-4aa5-9139-ee9934ac27ff.png)

Una vez ejecutados los comandos mostrados obtendremos una nueva shell, en la cual tendremos que editar la variable **PATH** para poder ejecutar los comandos plenamente:

![18. editar ruta PATH.png](attachment:a4833c27-ab52-4ca4-9f10-7f607fb74ce3:9ec2987d-7ab4-4fe1-9426-3d14c30e7c8a.png)

Luego de hacer esto, seremos capaces de ver el contenido del **flag3**:

![19. flag3.png](attachment:33d50855-48af-4495-ac75-6fbcc72ab47f:46f9ac10-0352-4a97-97a2-e76b7a937bd9.png)

---

### Post-Explotación/Escalada de privilegios

Ahora vamos a intentar cambiar al usuario **jerry** con las credenciales que conseguimos anteriormente: 

![20. su jerry.png](attachment:b9020277-2cdd-4554-b9fc-01ebeb1bea91:8a7733d1-d2a0-4b5e-aff0-5c056f01ea3f.png)

Ahora seremos capaces de leer el **flag4**, el cual se encuentra en el directorio personal de **jerry**:

![21. flag4.png](attachment:ffb4885b-2e4a-4ab3-82f7-dd971e128853:3cf861ab-e497-4b17-9fb0-7bd8e3c7407f.png)

El flag nos menciona **git** como pista, así que probablemente a través de este podamos escalar privilegios a **root**, vamos a comprobarlo ejecutando el comando: `sudo -l`, para listar los comando que el usuario actual tiene permiso de ejecutar como **root**:

![22. descubrimiento de git.png](attachment:352db2e1-a46b-4c8e-af78-a2e45a7f9239:22e8003e-5e40-45d8-be79-8be452f8e361.png)

La suposición es correcta. Haciendo una búsqueda del comando **git** en el portal de **gtfobins** hallamos una manera de aprovecharnos de este para poder volvernos **root**:

![23. gtfobins git.png](attachment:e1ec49ce-6216-4258-97bf-8c9745bceee4:fb644218-3f08-4675-b6ff-ea78d3585ebc.png)

Una vez ejecutados los comando anteriores nos habremos vuelto **root**:

![24. escalada de privilegios.png](attachment:c3d32fc2-40aa-4f58-8ed1-9818280a9865:7397c559-1e7a-466a-aed6-8834dc7af9aa.png)

Ahora ya tendríamos acceso a nuestro objetivo, la **root flag**:

![25. root flag.png](attachment:1252d471-d12b-4ff2-b4cc-54b7c103315b:582c6e5d-3636-48e7-9123-912659e81e80.png)

**¡Y con esto ya habremos hecho nuestro el sistema de la máquina DC-2!**
