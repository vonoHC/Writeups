## Introduccion

Metasploitable es una máquina virtual vulnerable diseñada específicamente para practicar pruebas de penetración y análisis de seguridad en entornos controlados. Fue creada por Rapid7 como una plataforma educativa que permite a estudiantes y profesionales de ciberseguridad identificar, explotar y comprender distintas vulnerabilidades reales presentes en sistemas y servicios mal configurados.

Esta máquina incluye aplicaciones desactualizadas, configuraciones inseguras y múltiples fallos conocidos, lo que la convierte en un entorno ideal para aprender técnicas de enumeración, explotación y post-explotación sin poner en riesgo sistemas reales. Debido a su naturaleza deliberadamente vulnerable, se recomienda utilizarla únicamente en laboratorios aislados y con fines educativos.

En este Writeup, aprenderemos cómo realizar todos los procedimientos necesarios para comprometer Metasploitable, desde instalar la máquina virtual hasta explotar la mayoría de los servicios activos de la forma más manual posible.

## Instalacion
Para la instalación de Metasploitable, accedemos al link de descarga para obtener la máquina:
[Metasploitable - SourceForge ](https://sourceforge.net/projects/metasploitable/)

imagen

Una vez que hayamos descargado la maquina virtual, es momento de ingresarla en nuestro hipervirtualizador preferido e iniciarla (para este ejemplo usare VMware):

Como primer paso para instalar la maquina virtual, presionamos el boton "Abrir una maquina virtual" y seleccionamos el archivo descargado:
imagen
imagen

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
imagen
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








