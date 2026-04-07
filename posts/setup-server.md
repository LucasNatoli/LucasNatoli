# Setup servidor 
## Actualizar el sistema  operativo

```
sudo apt update
sudo apt upgrade
```
## SSH
## Instalar Nginx - Web server
Para servir paginas html mediante http vamos a instalar Nginx. Si bien existen otras opciones como Apache, este es el que elijo por rendimiento y bajo consumo de memoria. Para instalarlo corremos en la consola el siguiente comando

```
sudo apt install nginx
```
Resultado:
```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1 libfreetype6 libgd3
  libjbig0 libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip
  libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter
  libnginx-mod-mail libnginx-mod-stream libtiff5 libwebp6 libxpm4 libxslt1.1
  nginx-common nginx-core
Suggested packages:
  libgd-tools fcgiwrap nginx-doc ssl-cert
The following NEW packages will be installed:
  fontconfig-config fonts-dejavu-core libfontconfig1 libfreetype6 libgd3
  libjbig0 libjpeg-turbo8 libjpeg8 libnginx-mod-http-geoip
  libnginx-mod-http-image-filter libnginx-mod-http-xslt-filter
  libnginx-mod-mail libnginx-mod-stream libtiff5 libwebp6 libxpm4 libxslt1.1
  nginx nginx-common nginx-core
0 upgraded, 20 newly installed, 0 to remove and 0 not upgraded.
Need to get 2700 kB of archives.
After this operation, 8907 kB of additional disk space will be used.
Do you want to continue? [Y/n] 
```

Confirmamos la instalacion presionando la tecla Y y dejamos correr el instalador. El servidor web se instalara junto con los paquetes que se indican en la seccion 'additiona packages' que se listan mas arriba. 

Una vez instalado podemos verificar el estado del servicio mediante el comando:
```
sudo systemctl status nginx
```
Y nos devolvera el estado del servicio: 
```
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2020-02-17 15:02:35 UTC; 1min 58s ago
     Docs: man:nginx(8)
 Main PID: 22795 (nginx)
    Tasks: 5 (limit: 1046)
   CGroup: /system.slice/nginx.service
           ├─22795 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           ├─22796 nginx: worker process
           ├─22797 nginx: worker process
           ├─22798 nginx: worker process
           └─22799 nginx: worker process

Feb 17 15:02:35 ubuntu systemd[1]: Starting A high performance web server and a reverse proxy server...
Feb 17 15:02:35 ubuntu systemd[1]: Started A high performance web server and a reverse proxy server.
```

#### Verificar que el servicio Nginx esta funcionando.
Una vez instalado, el servicio estará corriendo automáticamente. Para verificarlo, iniciamos un un navegador y navegamos a la IP publica de la maquina donde lo instalamos

```
http://45.200.23.543
```

SI todo funcionó bien, el navegador nos mostrará la página de bienvenida de Nginx

#### Comandos del servicio Nginx:

Iniciar el servicio en el arranque
```
sudo systemctl enable nginx
```

Iniciar el servicio manualmente
```
sudo systemctl start nginx
```

Reiniciar el servicio
```
sudo systemctl restart nginx
```

Detener el servicio
```
sudo systemctl restart nginx
```

## Instalar MySql - Base de datos

Instalamos el servidor usanto el siguiente comando:

```
sudo apt install mysql-server
```
Nos informará: 
```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  libaio1 libcgi-fast-perl libcgi-pm-perl libencode-locale-perl
  libevent-core-2.1-6 libfcgi-perl libhtml-parser-perl libhtml-tagset-perl
  libhtml-template-perl libhttp-date-perl libhttp-message-perl libio-html-perl
  liblwp-mediatypes-perl libtimedate-perl liburi-perl mysql-client-5.7
  mysql-client-core-5.7 mysql-common mysql-server-5.7 mysql-server-core-5.7
Suggested packages:
  libdata-dump-perl libipc-sharedcache-perl libwww-perl mailx tinyca
The following NEW packages will be installed:
  libaio1 libcgi-fast-perl libcgi-pm-perl libencode-locale-perl
  libevent-core-2.1-6 libfcgi-perl libhtml-parser-perl libhtml-tagset-perl
  libhtml-template-perl libhttp-date-perl libhttp-message-perl libio-html-perl
  liblwp-mediatypes-perl libtimedate-perl liburi-perl mysql-client-5.7
  mysql-client-core-5.7 mysql-common mysql-server mysql-server-5.7
  mysql-server-core-5.7
0 upgraded, 21 newly installed, 0 to remove and 0 not upgraded.
Need to get 18.8 MB of archives.
After this operation, 153 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
```

Presionamos la tecla Y y esperamos a que finalice la instalacion. Una vez terminada estarán disponibles el servidor y el cliente MySql.

El siguiente paso es asegurar la instalación del servidor de base de datos. Usamos el comando

```
sudo mysql_secure_installation
```

Este programa nos permitira habilitar/deshabilitar el plugin de validacion de passwords, dehabilitar el inicio de sesión remoto de la cuenta de administración, eliminar la cuenta 'anonymous', borrar la base de datos 'test' y recargar la nueva confirguracion del servidor de base de datos.
