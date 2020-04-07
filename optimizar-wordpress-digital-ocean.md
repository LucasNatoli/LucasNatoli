# Optimizar un site hecho en WordPress utilizando Digital Ocean

### Introduccón 

A medida que tu sitio crezca, puede alcanzar el punto en el que supere los recursos y configuración del servidor. Ya sea que tengas un hosting básico o que estes hosteando tu servidor web y base de datos en una misma máquina, podría ser una buena idea separar estas dos funciones para que cada una pueda operar en su propio hardware y compartir la carga de responder a los pedidos de los visitantes. 

En esta guía vamos a aprender a configurar una base de datos MySQL y un servidor web Nginx en los que pueda correr tu WordPress. La técnica tambien es aplicable a otras configuraciónes como Apache y PostgreSQL o para otras aplicaciones ademas de WordPress.

## Prerequisitos

Antes de empezar con esta guía necesitas:

* Dos servidores Ubuntu 18.0.4. Cada uno debe tener un usuario con privilegios de administrador (root) y además un Firewall habilitado (como se describe en ...). Uno de los servidores va a hostear el backend Mysql al que nos referiremos como **servidor de base de datos**. El otro funcionará como servidor web y se conectara a la base de datos remotamente. Nos referiremos este último como **servidor web**


* Nginx y PHP instalados en el **servidor web**. La guía __Como instalar Nginx y PHP en Ubuntu 18.0.4__ te ayudara para completar este proceso.
* MySQL instalado en el **servidor de base de datos**. La guía __Como instalar Mysql en Ubuntu 18.0.4__ te ayudara a completar este paso. Opcionalmente (aunque se recomienda enfáticamente), certificados TLS/SSL de Let's Encrypt instalados en el servidor web. Los certificados son gratuitos aunque ya deberás contar con un nombre de dominio pago y haber configurados los registros DNS para tu servidor. La guía __Cómo proteger Nginx con Let's Encrypt__ te ayudara a obtener estos certificados.

## Paso 1 - Configurar MySQL para que atienda conexiones remotas

Almacenar los datos en un servidor separado es una buena manera de escalar facilmente luego de tocar el techo de performance de la configuración de una máquina. También provee la estructura básica para balancear la carga y expandir la infrastructura aún más en el futuro. Luego de instalar MySQL siguiendo la guía indicada en los prerequisitos, necesitraś cambiar algunos valores en la configuración de MySQL para que atienda conexiones realizadas desde otra otras computadoras

La mayoria de estos cambios se realizadrán en el archivo `mysqld.cnf`, ubicado por defecto en el directorio `/etc/mysql/mysql.conf.d/`. Abre este archivo utilizando privilegios administrativos en servidor de base de datos utilizando tu editor preferido. Aquí utilizaremos `nano`: 

`$ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf`

Este archivos esta dividido en secciones identificadas por etiquetas entre corchetes (`[` y `]`). Encuentra la sección con la etiqueta `mysqld`:

```
. . .
[mysqld]
. . .
```
Dentro de esta seccion ubica el parámetro `bind-address`. Este parámetro le indica al software de base de datos la direccion de la red en la que debe esperar las conexiones.

Por defecto, este valor es `127.0.0.1` indicando que MySQL esta configurado para recibir conexiones locales únicamente. Necesitas cambiar esto para referenciar una dirección IP externa adonde se encuentre el servidor web. 

Si ambos servidores ser encuentras en el mismo datacenter y los dos disponen de direccion IP privadas, utilizaremos la IP privada. De otra forma, utilizaremos la dirección IP pública:

```
[mysqld]
. . .
bind-address = ip_server_db
```

Ya que la conexión a la base de datos se realiza a través de internet, se recomienda requerir conexiones encriptadas como medida para mantener la información segura. Si no se encripta la conexión a MySQL, alguien en tu misma red podría obtener información sensible en la comunicación entre el servidor web y el servidor de base de datos. Para encriptar las conexiónes a MySQL, agrega la siguiente linea luego del parámentro `bind-address` que acabas de actualizar: 

```
[mysqld]
. . .
require_secure_transport = on
. . .
```

Cuando termines, guarda y cierra el archivo. Si estas usando `nano`, hazlo presionando `CTRL+X`, `Y`, y luego `ENTER`.

Para que las conexiones SSL funcionen, necesitas crear algunas claves y certificados. MySQL viene con un comando que las creará automáticamente. Ejecuta el siguiente comando, que creará los archivos necesarios. También los hara legibles por el servidor MySQL al utilizar el UID del usuario *mysql*: 

```
$ sudo mysql_ssl_rsa_setup --uid=mysql
```
Para forzar la actualizacón y lectura de la nueva información por parte de MySQL, reinicia la base de datos. 

```
$ sudo systemctl restart mysql
```

Para confirmar que el servidor esté escuchando en la interfaz externa, ejecuta el siguiente comando `netstat`:

```
$ sudo netstat -plunt | grep mysqld

```

Salida

```
tcp        0      0 ip_server_db:3306     0.0.0.0:*               LISTEN      27328/mysqld
```

`netstat` imprime estadisticas del sistema de red del servidor. Esta salida muestra que un proceso llamado `mysqld` esta adjuntado a la direccion `ip_server_db` en el puerto `3306`, el puerto standard de MySQL, confirmando que el servidor esta escuchando en la interfaz apropiada.

A continuacón se debe abrir dicho puerto en el firewall de la máquina para permitir el tráfico entrante. 

Esos son todos los cambios necesarios en MySQL. A continuacón vamos a crear una base de datos y un par de perfiles de usuario, uno de los cuales será utilizado para acceder al servidor remotamente.

## Paso 2 - Crear una base de datos para WordPress y sus credenciales remotas

Aunque MySQL esta escuchando en una dirección IP externa, todavía no existe un usuario remoto habilitado ni una base de dato configurada. Vamos a crear una base de datos para WordPress y un par de usuarios que puedan acceder a esta última.

Comienza conectandote a MySQL como el usuario **root**:

```
$ sudo mysql
```

**ATENCION:** Si has habilitado la autenticación con password, necesitaras en cambio conectarte a MySQL mediante el siguiente comando:

```
$ mysql -u root -p
```

Desde el prompt de MySQL, creamos la base de datos que utilizará WordPress. Es de gran de ayuda darle a la base de datos un nombre reconocible para identificarla posteriormente. En este caso la nombraremos `wordpress`

```
$ mysql> CREATE DATABSE wordpress;
```

Ahora que has creado tu base de datos, necesitas a continuación crear un par de usuarios. Vamos a crear un usuario con acceso local únicamente y también un usuario ligado a la direccion IP del servidor.

Primero, crea tu usuario local, **wpuser**, y haz que la cuenta solo permita la conexión local utilizando `localhost` en la siguiente declaracion:

```
mysql> CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password_seguro';
```

Luego otorgaremos acceso total a la base de datos `wordpress` a este usuario:

```
mysql> GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
```

Ahora este usuario puede realizar cualquier operacion en la base de datos de WordPress, pero esta cuenta no puede ser utilizada remotamente, ya que solo permite conexiones desde la máquina local. Con ento en mente crearemos una cuenta que permitira conexiones desde el servidor web. Para esto, necesitamos la direccion IP del servidor web.


???????????????????????????????????

Hay que tener en cuenta que debemos indicar la misma dirección IP indicada en el archivo `mysqld.cnf`. Esto implica que si utilizaste la dirección IP privada en el  archivo `mysqld.cnf` necesitaras incluir dicha dirección privada del servidor web en los siguientes dos comandos. Si hubieras configurado MySQL para utilizar la IP pública, entonces utilizaresmos esta última dirección IP.

????????????????????

```
mysql> CREATE USER 'remotewpuser'@'web_server_ip' IDENTIFIED BY 'password_seguro';
```

Una vez creada la cuenta de acceso remoto le otorgaremos los mismo permisos que la cuenta de acceso local:

```
mysql> GRANT ALL PRIVILEGES ON wordpress.* TO 'remotewpuser'@'web_server_ip';
```

Por ultimo, recargamos los privilegios para que MySQL comience a utilizarlos:

```
mysql> FLUSH PRIVILEGES;
```

Finalmente abandonaremos el prompt MySQL escribiendo

```
mysql> exit;
```

## Paso 3 - Testear las conexiones local y remota

Antes de continuar es conveniente verificar que puedas conectarte a la base de datos desde ambas máquinas: el servidor de base de datos y el servidor web.

Primero probamos la conexión local desde el **servidor de base de datos** intentando iniciar sesion con la nueva cuenta:

```
$ mysql -u wpuser -p
```

Cuando lo pida, ingresaremos el password indicado para esta cuenta.

Si haz obtenido el prompt MySQL, entonces la conexión ha sido exitosa. Podemos verificar el acceso a la base de datos de WordPress usando el comando:

```
mysql> SHOW DATABASES;
```

Salida:
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| wordpress          |
+--------------------+
2 rows in set (0.00 sec)
```

Si la salida muestra estas dos entradas, podemos estar seguros de que la cuenta local esta bien configurada y entonces podemos abandonar el prompt MySQL escribiendo:

```
mysql> exit;
```

A continuacion, iniciaremos sesión en el **servidor web** para testear la conexión remota. Abre una nueva terminal en tu computadora e ingresa el siguiente comando:

```
$ ssh usuario_administrador@ip_servidor_web
```

Necesitarás instalar algunas herramientas para MySQL en el **servidor web** para poder conectarte a la base de datos remota. Primero, actualiza el cache de paquetes locales si no los haz hecho recientemente:

```
$ sudo apt update
```

Luego instala las herramientas cliente del MySQL:

```
$ sudo apt install mysql-client
```

A continuación, conectate al **servidor de base de datos** utilizando la siguiente sintaxis:

```
$ mysql -u remotewpuser -h ip_server_db -p
```

Nuevamente debes asegurarte de utilizar la dirección IP correcta para el servidor de base de datos. Si configuraste MySQL para esperar conexiones desde la red privada, ingresa la direccion privada de tu servidor de base de datos. De otra forma, ingresa la direccion IP pública de tu servidor de base de datos. 

Se te pedirá el password de la cuenta `remotewpuser`. Luego de ingresarla, y si todo funciona como esperamos, verás el prompt MySQL. Verifica que la conexión esta usando SSL ingresando el siguiente comando:

```
mysql> status
```

Si la conexión esta utilizando efectivamente SSL, la linea `SSL:` los indicará como se muestra a continuación:

```
mysql  Ver 14.14 Distrib 5.7.18, for Linux (x86_64) using  EditLine wrapper

Connection id:      52
Current database:
Current user:       remotewpuser@203.0.113.111
SSL:         Cipher in use is DHE-RSA-AES256-SHA
Current pager:      stdout
Using outfile:      ''
Using delimiter:    ;
Server version:     5.7.18-0ubuntu0.16.04.1 (Ubuntu)
Protocol version:   10
Connection:     203.0.113.111 via TCP/IP
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    utf8
Conn.  characterset:    utf8
TCP port:       3306
Uptime:         3 hours 43 min 40 sec

Threads: 1  Questions: 1858  Slow queries: 0  Opens: 276  Flush tables: 1  Open tables: 184  Queries per second avg: 0.138
--------------
```

Una vez verificada la conexion remota, salimos del prompt MySQL

```
mysql> exit
```

Despues a haber probado la conexión remota con éxito, procedemos a instalar WordPress en el servidor web.

## Paso 4 - Instalar WordPress

En este paso vamos a descargar y extraer el software de WordPress, configurar la conexion a la base de datos y correr la instalacion de WordPress en la web.

En tu servidor web, descarga la ultima version de Wordpres en el directorio home

```
$ cd ~
$ curl -O https://wordpress.org/latest.tar.gz
```

Extrae los archivos, que quedarán almacenados en el directorio `wordpress` de tu directorio home:

```
$ tar xzvf latest.tar.gz
```

WordPress incluye un archivo de configuración de ejemplo que utilizaremos como punto de partida. Haz una copia de este archivo, quitando `-sample` del nombre para que pueda ser leido por WordPress:

```
$ cp ~/wordpress/wp-config-sample.php ~/worpdress/wp-config.php
```

Cuando edites tu archivo de configuración, la primera tarea sera ajustar algunas claves secretas para darle mas seguridad a la instalación. WordPress prové un generador de estos valores de manera que no hace falte que intentes inventar algunos buenos valores por ti mismo. Estos valores se utilizarán internamente, por lo que usar valores bien complejos no daña la usabilidad.

Para obtener los valores seguros desde el generador de claves de WordPress, ingresa:

```
$ curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

Esto imprimira algunas claves en la salida que agregaremos en el archivo `wp-config.php`:

Salida: 

```
define('AUTH_KEY',         'L4|2Yh(giOtMLHg3#] DO NOT COPY THESE VALUES %G00o|te^5YG@)');
define('SECURE_AUTH_KEY',  'DCs-k+MwB90/-E(=!/ DO NOT COPY THESE VALUES +WBzDq:7U[#Wn9');
define('LOGGED_IN_KEY',    '*0kP!|VS.K=;#fPMlO DO NOT COPY THESE VALUES +&[%8xF*,18c @');
define('NONCE_KEY',        'fmFPF?UJi&(j-{8=$- DO NOT COPY THESE VALUES CCZ?Q+_~1ZU~;G');
define('AUTH_SALT',        '@qA7f}2utTEFNdnbEa DO NOT COPY THESE VALUES t}Vw+8=K%20s=a');
define('SECURE_AUTH_SALT', '%BW6s+d:7K?-`C%zw4 DO NOT COPY THESE VALUES 70U}PO1ejW+7|8');
define('LOGGED_IN_SALT',   '-l>F:-dbcWof%4kKmj DO NOT COPY THESE VALUES 8Ypslin3~d|wLD');
define('NONCE_SALT',       '4J(<`4&&F (WiK9K#] DO NOT COPY THESE VALUES ^ZikS`es#Fo:V6');
```

Copia la salida que recibiste en el portapapeles y a continuación abriremos el archivo de configuración en el editor de texto:

```
$ nano ~/wordpress/wp-config.php
```

Ubica la seccion que contiene los valores default para estos parametros. Borra esas lineas y pega los valores que copiaste de la linea de comando. 

Luego, ingresa la informacion de conexion de tu base de datos remoa. Estas lineas de configuracion estan al inicio del archivo, justo arriva de donde pegaste las claves. Recuerda utilizar la misma direccion IP que utilizaste durante la prueba remata de base de datos anteriormente:

```
. . .
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'remotewpuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');

/** MySQL hostname */
define('DB_HOST', 'ip_servidor_db');
. . .
```

Finalmente, en cualquier parte del archivo, agrega la siguiente linea que le indicará a WordPress que utilice una conexión SSL (encriptada) con tu base de datos MySQL:

```
define('MYSQL_CLIENT_FLAGS', MYSQLI_CLIENT_SSL);
```

Guarda y cierra el archivo. A continuación, copia los archivos y directorios que se encuentran en el directorio `~/wordpress` a la raiz de los documentos Nginx. NOTA: Este comando utiliza el flag `-a` para asegurarnos de que todos los permisos se transladen al directorio destino:

```
$ sudo cp -a ~/wordpress/* /var/www/html
```

Una vez hecho esto, solo queda darle al usuario www-data los permisos de todos los archivos contenidos en la raiz de los documentos Nginx para que el servidor web pueda hacer modificaciones en los mismos:

```
$ sudo chown -R www-data:www-data /var/www/html
```
Ahora WordPress ya esta instalado y estamos listos para correr su setup a través de la web.

## Paso 5 - Configurar WordPress a través de la interfaz web.

WordPress cuenta con un proceso de setup a traves de la web. A traves del mismo, nos hará algunas preguntas y creara todas las tablas necesarias en la base de datos. En este paso, navegaremos a través de este setup, que podras utilizar como punto de partida para construir tu propio sitio web que utiliza una base de datos remota.

Dirígete con tu navegador al nombre de dominio (o IP publica) asociada a tu web server:

```
http://tu-sitio.com
```

Veras una pantalla de selección de idioma para el instalador de WordPress. Selecciona el lenguaje apropiado y haz click en a travéz de la pantalla de instalación.

IMAGEN

Una vez que hayas completado la informacion pertinente, necesitarás loguearte a la interfaz de administracion de WordPress. Serás llevado al Escritorio donde puedes personalizar tu nuevo sitio WordPress.

## Conclusión

Siguiendo esta guía, haz instalado y configurado una base de datos MySQL para que acepte conexiones protegidas por SSL desde una computadora remota. Los comandos y técnicas utilizados en esta guia son aplicables a cualquier aplicación web escrita en cualquier lenguaje, pero los detalles de cada implementacion específica serán diferentes. Refiérete a la documentación de tu aplicación o lenguaje de programación para más informacion sobre como conectarse a la base de datos.
