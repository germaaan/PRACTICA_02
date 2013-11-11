# Práctica 2: Aislamiento de una aplicación web usando
# German Martinez Maldonado
# Publicado bajo licencia GNU GENERAL PUBLIC LICENSE Version 3

Esta práctica consiste en realizar la instalación y configuración de una jaula chroot para habituarse al uso de estas infraestructuras virtuales. Empezamos creando el directorio en el que vamos a crear nuestra jaula e instalando un sistema básico dentro de dicho directorio con la utilidad de **debostrap**:
 ```
sudo mkdir -p /home/jaulas/practica2
sudo debootstrap --arch=amd64 saucy /home/jaulas/practica2/ http://archive.ubuntu.com/ubuntu
 ```
![practica02_01](https://dl.dropboxusercontent.com/s/v829cfrg5f8scwj/01.png)

Ahora vamos a configurar la jaula usando **schroot**, para lo que primero tenemos que crear un usuario con el que acceder a la jaula:
 ```
sudo useradd -s /bin/bash -m -d /home/jaulas/practica2/home/usupra2 -c "Usuario jaula" -g users usupra2
sudo passwd usupra2
 ```
![practica02_23](https://dl.dropboxusercontent.com/s/6tmgp1goihp0dvy/23.png)

A continuación, creamos el archivo de configuración **"practica2.conf"** en el directorio **/etc/schroot/chroot.d** con el siguiente contenido:
 ```
[practica2]
type=directory
description=PRACTICA_02
directory=/home/jaulas/practica2
users=usupra2
root-groups=root
root-users=root
aliases=practicaDOS
 ```

Comprobamos que podemos acceder mediante la configuración que acabamos de añadir:
 ```
sudo schroot -c practicaDOS --user=usupra2 --directory=/
 ```
![practica02_02](https://dl.dropboxusercontent.com/s/0lv2k0nlulpr8hb/schroot.png)

Ahora para que solo root y el usuario que hemos creado puedan acceder a la jaula, cambiamos el propietario de la estructura de carpetas de la jaula a root: 
 ```
sudo chown -R root:root /home/jaulas/practica2/
 ```

Nos situamos en el directorio de la jaula con **chroot**, montamos el filesystem virtual **/proc** para poder ejecutar las aplicaciones e instalamos el paquete **language-pack-es** para evitar problemas con algunos comandos porque no se tengan los **locales**:
 ```
sudo chroot /home/jaulas/practica2
mount -t proc proc /proc/
apt-get install language-pack-es
 ```
![practica02_02](https://dl.dropboxusercontent.com/s/jadg5zpsx1xg8u3/02.png)

Vamos a usar la misma aplicación que desarrollamos para la Práctica 1, por lo que necesitaremos instalar una solución **LAMP** consistente en un servidor web **Apache**, un gestor de base de datos **MySQL** y el lenguaje de programación **PHP**, además de los paquetes necesarios para que todas estas aplicaciones puedan funcionar de forma conjunta:
 ```
apt-get install apache2 php5 libapache2-mod-php5 php5-cli php5-mysql mysql-server mysql-client libmysqlclient-dev
 ```

La ventaja de instalar grandes cantidades de paquetes usando gestores de paquetes como **apt**, es que la configuración de los mismos es prácticamente automática, en este caso solo nos pedirá que introduzcamos la contraseña para acceder como usuario **root** a nuestra base de datos:
![practica02_05](https://dl.dropboxusercontent.com/s/j0d9abgpk78ykxy/05.png)

Es muy posible que la primera vez que arranquemos nuestro servidor **Apache** obtengamos un error como el que aparece en la imagen:
![practica02_06](https://dl.dropboxusercontent.com/s/4zx0qd903kw54xz/06.png)

Esto es porque nos hemos indicado un nombre para nuestro servidor en la configuración de **Apache**, por lo que debemos añadir la línea `ServerName localhost` al archivo de configuración (en nuestro caso, **/etc/apache2/apache2.conf**):
![practica02_07](https://dl.dropboxusercontent.com/s/txegs3e82112a5r/07.png)


Una vez que reiniciamos **Apache** para que cargue la nueva configuración (`services apache2 restart`), podremos acceder cualquier navegador que tengamos instalado en nuestro sistema de fuera de la jaula si introducimos la dirección **http://localhost/**:
![practica02_09](https://dl.dropboxusercontent.com/s/kifcd6yciusf5fk/09.png)

Pasamos ahora a comprobar que nuestro servidor puede procesar aplicaciones PHP, para lo que creamos una pequeña aplicación PHP que cuando es procesada en un servidor devuelve la información del PHP instalado en el sistema. Esta aplicación la guardamos en el directorio **/var/www** que corresponde al directorio raíz desde el que nuestro servidor sirve el contenido vía web.
 ```
German Martinez Maldonado

<?php
          phpinfo();
?>
 ```

Podremos acceder a dicha aplicación si introducimos la dirección **http://localhost/info.php**:
![practica02_11](https://dl.dropboxusercontent.com/s/46i2xpqj3pag0j9/11.png)

Para trabajar con MySQL podemos optar por usar la aplicación **phpMyAdmin**, que nos proporcionará una interfaz web que nos simplificará el funcionamiento del gestor. El problema está en que como **debootstrap** instala un sistema mínimo, solo añade el repositorio principal de la distribución, así que no podremos instalar directamente **phpMyAdmin** como hemos hecho con el resto de paquetes:

Para saber en que repositorio está un determinado paquete podemos usar `apt-cache` con la opción `policy`, como vemos **phpMyAdmin** se encuentra en el repositorio **universe**, que contiene los programas mantenidos por la comunidad:
 ```
sudo apt-cache policy phpmyadmin
 ```
![practica02_13](https://dl.dropboxusercontent.com/s/tih2w80ny3ltzfw/13.png)

Así que editamos el fichero **/etc/apt/sources.list**, si hay una entrada para el repositorio **main** simplemente tenemos que añadir **universe**, en caso contrario añadimos la siguiente línea completa:
 ```
deb http://archive.ubuntu.com/ubuntu saucy main universe
 ```

Una vez que actualicemos la lista de paquetes, podremos instalar **phpMyAdmin** como cualquier otro paquete:
 ```
apt-get update
apt-get install phpmyadmin
 ```

Indicando cuando lo pida que queremos realizar una configuración de **phpMyAdmin** para un servidor **Apache**:
![practica02_16](https://dl.dropboxusercontent.com/s/6rdhzmjuob01x4c/16.png)

Seleccionamos que sí queremos configurar la base de datos con la aplicación **dbconfig-common**:
![practica02_17](https://dl.dropboxusercontent.com/s/d0nhmdmn8d9cs99/17.png)

Introducimos una contraseña para acceder a la administración de la base de datos:
![practica02_18](https://dl.dropboxusercontent.com/s/h65rs83uc7vanop/18.png)

Ahora introducimos la contraseña con la que accederemos a la base de datos:
![practica02_19](https://dl.dropboxusercontent.com/s/agv5bj64hmehaiq/19.png)

Para que **Apache** pueda procesar **phpMyAdmin** y podamos acceder desde el navegador, deberemos incluir el archivo de configuración de **phpMyAdmin** en el propio archivo de configuración de **Apache**:
 ```
Include /etc/phpmyadmin/apache.conf
 ```
![practica02_20](https://dl.dropboxusercontent.com/s/mdcbv4jjsnm594q/20.png)

Ya podemos acceder a **phpMyAdmin** si introducimos la dirección **http://localhost/phpmyadmin**:
![practica02_21](https://dl.dropboxusercontent.com/s/880ajf7k14h2brg/21.png)

Y una vez hayamos accedido como un usuario existente, veremos su interfaz:
![practica02_22](https://dl.dropboxusercontent.com/s/3pkoo9uuknuz6wx/22.png)

Sobretodo en aplicaciones web que pueden estar corriendo en una máquina a la que no estamos directamente conectados, es posible que nos interese tener la opción de acceder a dicha máquina de forma remota para poder cambiar configuraciones, por esto vamos a limitar el ámbito de funcionamiento remoto al sistema de archivos de la jaula del usuario. Para que remotamente el usuario no pueda salirse de la jaula deberemos especificarlo en la configuración de SSH (**/etc/ssh/sshd_config**) con las siguientes líneas:
 ```
Match User usupra2
          ChrootDirectory /home/jaulas/practica2
          X11Forwarding no
          AllowTcpForwarding no
 ```
![practica02_24](https://dl.dropboxusercontent.com/s/tsj5cu4juqn7inv/24.png)

Después de reiniciar SSH, si nos conectamos mediante el usuario que hemos creado, podremos comprobar que el directorio en el que nos situamos como raíz es el directorio de nuestra jaula, no pudiendo "subir" más arriba:
 ```
sudo service ssh
ssh usupra2@localhost
 ```
![practica02_25](https://dl.dropboxusercontent.com/s/5u9tbpy114pec7h/25.png)

Como la aplicación que vamos a meter en la jaula es una versión adaptada para este sistema de la aplicación desarrollada para la Práctica 1, instalaremos **git** para poder trabajar directamente desde el interior de nuestra jaula:
 ```
apt-get install git
 ```

Siendo lo primero que hacemos clonar el repositorio de la práctica:
 ```
git clone git://github.com/germaaan/PRACTICA_01.git
 ```
![practica02_27](https://dl.dropboxusercontent.com/s/fzi2nnpxb325bwt/27.png)

Antes de empezar a adaptar la aplicación para que funcione en este sistema, vamos a restaurar la base de datos usada por la aplicación, este se puede realizar de dos formas, en modo terminal o a través de una interfaz web. En modo terminal usaríamos **mysql** para trabajar directamente con el gesto de base de datos, accediendo, creando la base de datos y finalmente importando los datos:
```
mysql -u root -p

mysql> create database acceso;                 # "acceso" es la base de datos con la información
mysql> exit

mysql -u root -p acceso < /acceso.sql          # Siendo "acceso.sql" el archivo con la información
```
![practica02_28](https://dl.dropboxusercontent.com/s/h03o3ql3njwl84f/28.png)

Si nos volvemos a conectar a la base de datos, podemos comprobar que la información ha sido restaurada correctamente:
 ```
mysql -u root -p

mysql> use acceso;                                                # Seleccionamos la base de datos
mysql> describe Alumnos;                                          # Comprobamos la estructura de la tabla
mysql> select * from Alumnos where id_alumno = 'germaaan'         # Hacemos una consulta de prueba
 ```
![practica02_29](https://dl.dropboxusercontent.com/s/kv0mxwz21zkdys9/29.png)

Otra opción sería realizar esta misma operación mediante la interfaz web de **phpMyAdmin**, para lo cual deberíamos acceder desde la ventana principal a la pestaña **"Importar"**, seleccionando el archivo SQL que contiene la información de la base de datos y pulsando el botón "Continuar":
![practica02_30](https://dl.dropboxusercontent.com/s/5plezxivbftvidl/30.png)

Como en este caso la aplicación está en un servidor en una jaula en nuestro sistema, que no puede ser accedido desde el exterior, en principio nos dará igual que el archivo de configuración se encuentre en el mismo directorio que la aplicación, por lo que en el mismo **/var/www** crearemos el archivo **"configuracion.inc"** con la configuración para poder realizar la conexión a la base de datos:
```
<?php
          define("DB_DSN","mysql:host=localhost;port=22;dbname=acceso");
          define("DB_USUARIO","root");
          define("DB_PASS","rootroot");
          define("TABLA_ALUMNOS","Alumnos");
?>
```

Teniendo ahora que modificar la ruta del archivo de configuración importado en la clase PHP:
```
require_once "configuracion.inc;"
```
![practica02_34](https://dl.dropboxusercontent.com/s/d9owc0svyl9hivv/34.png)

Finalmente, podemos comprobar que la aplicación es la misma que montamos en **OpenShift**, pero ahora funcionando localmente bajo un sistema LAMP montado en una jaula chroot:

![practica02_35](https://dl.dropboxusercontent.com/s/niu657lpgf8yx3l/35.png)
![practica02_36](https://dl.dropboxusercontent.com/s/dtk3o0wxvti0ywn/36.png)
![practica02_37](https://dl.dropboxusercontent.com/s/au3zwms3aslbhj4/37.png)
