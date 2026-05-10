# Introducción
Empire: Breakout es una máquina de dificultad media de VulnHub enfocada en técnicas de enumeración de servicios, análisis de aplicaciones web y escalada de privilegios en entornos Linux. En este Writeup se documentará paso a paso el proceso de explotación de la máquina, comenzando con la enumeración de servicios HTTP y SMB, el análisis de credenciales ocultas mediante Brainfuck y la obtención de acceso a través de Usermin. Posteriormente, se realizará reconocimiento interno del sistema y se aprovecharán capacidades especiales asignadas al binario `tar` para acceder a archivos restringidos y obtener privilegios de root.

---

### Reconocimiento

Como primer paso obtenemos la **IP** de la máquina objetivo y le realizamos un **ping,** de esta forma sabremos si está activa.

![1. ping.png](attachment:6f279751-c0a3-40ac-ae32-3b9008bbfc09:c843d616-6003-4f78-bdfd-be45c0f44579.png)

Ahora es momento de hacer un escaneo con **nmap** para enumerar los puertos y servicios que están activos en la máquina: 

![2. nmap.png](attachment:021fd64a-3c90-42c4-b6a3-7e123f0b1b92:eb9b3bb9-fb33-4880-ba7d-2dfe2759512a.png)

Vemos que en la máquina están abiertos los puertos: **80**, **139**, **445**, **10000**, **20000**, los cuales están corriendo los servicios: **HTTP** (en los puertos **80**, **10000** y **20000**) y **SMB** (en los puertos **139** y **445**).

Vamos a ingresar a la página web por defecto del puerto 80 para ver que hallamos:

![3. pagina 80.png](attachment:8f80d473-dc26-47f6-9d9d-7239dffe3ff0:51349fba-1ab5-4ad0-8c8b-f01949cc6547.png)

Es una página por defecto de **Apache**, revisemos el código fuente para ver si vemos algo interesante:

![4. codigo fuente.png](attachment:f6570e15-6174-4ee1-b2a5-7d7a0eda56b6:44e51cef-9aed-4396-8a64-68df982fa8d2.png)

Vemos un comentario que indica que el texto de abajo es una credencial de acceso encriptada. 

Investigando un poco descubrimos que la credencial está escrita en un lenguaje de programación llamado **Brainfuck**, y lo decodificamos en el siguiente decoder: https://www.dcode.fr/brainfuck-language

![5. brainfuck decoder.png](attachment:2ab7a233-abfc-47bc-a7a2-b3e09e354cab:1737b8df-1c1f-4a69-ac39-c88196f1985a.png)

Al parecer es una contraseña para iniciar sesión en algún sitio, pero nos falta el usuario…

Como en la máquina está corriendo el servicio **SMB** podríamos usar la herramienta **enum4linux** para enumerar usuarios existentes, vamos a ello: 

![6. enum4.png](attachment:c5c607fa-2040-468d-b201-0466e504cab4:71a6aeb2-c187-4b5b-a734-81fe8b3e906d.png)

![6. cyber.png](attachment:b01f81ae-b8cb-4716-9391-47a4151b2f4d:20770364-0a1e-46fb-ab85-90db93627346.png)

La salida de **enum4linux** nos muestra que existe el usuario **cyber**. Ya que tenemos las credenciales de inicio de sesión de algún sitio es momento de hallarlo, para esto vamos a visitar las otras páginas que tiene la máquina:

![7. webmin.png](attachment:4c2f02d6-3834-4482-b56f-c349285d5239:0281c95f-2ee5-4797-94fb-30c7756fa3c0.png)

En la página del puerto **10000** llamada **webmin**, no funcionan las credenciales, intentemos con la página en el puerto **20000**:

![8. usermin.png](attachment:e6492a1f-26b4-41db-8d2c-6b244478889a:bfffa02b-8d0e-48a5-8d06-eca549b9bb5e.png)

![8. inicio de sesion exitoso.png](attachment:2b9af672-36a6-4360-ac80-58642e1efd18:dce3be5a-87e0-4b42-b2d3-9338fdaae4e7.png)

El inicio de sesión fue exitoso. Si navegamos un poco por la página seremos capaces de hallar una terminal web interactiva donde tendremos acceso a la **user flag**:

![9. userflag.png](attachment:e6ba5840-9f10-4d39-8360-263620e9b0d9:877e613b-4491-46c4-b274-ac439d9c1365.png)

---

### Post-Explotación/Escalada de privilegios

Ahora llegó el momento de encontrar una forma de elevar privilegios, para ello vamos a listar los archivos ejecutables con el comando: **`getcap -r / 2>/dev/null`**

![El comando tar permite comprimir o descomprimir archivos y directorios.](attachment:3af34e68-e69b-4d8e-9be9-6e5e080f2527:cd56be13-6b6e-449c-a9b5-4ee56e19b23b.png)

El comando tar permite comprimir o descomprimir archivos y directorios.

El resultado nos muestra que el comando tar tiene una capacidad especial que nos permite leer cualquier archivo sin importar los permisos, luego de comprimirlo. 

Luego de explorar por el sistema hallamos el archivo **`/var/backups/.old_pass.bak`** el cual tiene la contraseña para acceder al sistema como **root**:

![11. old_pass_bak.png](attachment:7805c090-458b-4ef7-a2c5-e3fb7898b22e:860c81c4-38ad-463c-82a0-c17724f13f72.png)

Ahora sería momento de usar tar para comprimir y descomprimir el archivo para poder leerlo:

![12. com y descompresion de old.pass.bak.png](attachment:2c1d645b-0c65-4601-90f9-509059e8cb28:feb039ea-36b0-4601-bf12-73423b08d06b.png)

Luego de haber hecho el proceso con los comandos mostrados en la captura anterior, podremos ver la contraseña del usuario **root**, la cual es la que se encuentra resaltada en la captura.

Ahora, si cerramos la sesión actual y podremos ingresar como **root** con las credenciales obtenidas:

![13. usermin root.png](attachment:0cf376ec-256e-432b-8c6a-8872dfae56c1:1afb9552-9926-455b-985a-f69e65dfc2ff.png)

![13. login root.png](attachment:6461655f-350d-4e1c-8730-f98491528b6c:c27c841a-d86d-4beb-8fde-3def9e34eeb4.png)

En este punto solo faltaría arrancar la terminal y obtener el **flag root**:

![14. flag root.png](attachment:de1e8e24-a12b-4fc9-beb4-56adfbe6ed8d:dce8d1ce-6907-492c-bf28-f80545db3088.png)

**¡Y con esto ya habremos hecho nuestro el sistema de la máquina Breakout!**
