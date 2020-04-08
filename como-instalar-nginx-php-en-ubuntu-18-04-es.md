# Cómo instalar Nginx y PHP en Ubuntu 18.04

## Introducción

Esta guía muesta la forma de instalar el servidor web Nginx y PHP para que corran en Ubuntu 18.04.

La guía esta pensada para servidores que se encuentren alojados en internet y cuenten con una dirección IP pública o un nombre de dominio propio. Para el caso de un nombre de dominio se recomienda enfáticamente utilizar certificados TLS/SSL de Let's Encrypt (mover esto mas adelante porque depende de que se haya instalado el servidor). 

## Requisitos previos

Para completar esta guia, debera contar con una instalación de Ubuntu 18.04 conteniendo una cuenta con privilegios de administrador (`sudo`). La guia de configracion de configuración inicial para servidores Ubuntu te ayudará en caso de que no sepas como hacerlo. 

Una vez que tengas tu usario disponible, estarás listo para comenzar con los pasos enumerados en esta guía.

## Paso 1 - Instalar el servider web Nginx

Para mostrar páginas web a los visitantes de nuestro sitio, emplearemos Nginx, un servidor web moderno y eficaz.

Todos los programas de software utilizados en este procedimiento provienen de los repositorios de paquetes predeterminados de Ubuntu. Esto significa que podemos utilizar el conjunto de programas de administración de paquetes `apt` para completar las instalaciones necesarias.

Ya que esta es la primera vez que usamos `apt` para esta sesión, comienza actualizando el índice de paquetes de tu servidor. Ingresa el comando:

```
$ sudo apt update
```

Una vez finalizada la ejecucion instalaremos el servidor web Nginx:

```
$ sudo apt install nginx
```
Una vez finalizada la ejecución Nginx comenzará a ejecutarse automáticamente.

Si tienes activo algún firewall recuerda habilitar el servicio correspondientes a Nginx para permitir el tráfico a través del puerto 80 (http).

Una vez agregada la regla del firewall, podrás probar si el sevidor se encuentra en ejecución accediendo al nombre de dominio o a la dirección IP pública mediante tu navegador web:

```
http://nombre_de_dominio_o_IP
```
 
 [IMAGEN Welcome to nginx]

 Si logras ver esta página, significa que Nginx se instaló correctamente.

 ## Paso 2 - Instalar PHP y configurar Nginx para utlizar el procesador de PHP

 Ya que Nginx no provee procesamiento nativo de PHP como otros servidores web, instalaremos `php-fpm` (FastCGI Process Manager) e indicaremos a Nginx que transmita las solicitudes de PHP a este software para su procesamiento.

 Para instalar el módulo php-fpm que extraera los archivos necesario ejecuta el siguiente comando:

```
 $ sudo apt install php-fpm
```
Una vez finalizada la instalacion, nos queda realizar algunos cambos en la configuración de Nginx para indicarle que utilice el procesador PHP para el contenido dinámico.

Esta configuración debe realizarse a nivel de bloque del servidor (similar a los hosts virutales de Apache). Para hacerlo, crea un nuevo archivo de configuración dentro del directorio `/etc/nginx/sites-available/`. Agregar este archivo de configuración permite restaurar fácilmente la configuración predeterminada si fuera necesario. Es de gran ayuda darle al archivo de configuración un nombre reconocible para poder identificarlo posteriormente, por ejemplo el nombre de dominio de tu sitio. En este caso utilizaremos el nombre  `mi-sitio.com`. Ingresa:

```
sudo nano /etc/nginx/sites-available/mi-sitio.com
```

Agrega el siguiente contenido, tomado y modificado ligeramente a partir del archivo de configuración predeterminado de bloques de servidor, a tu nuevo archivo de configuración de bloques de servidor:

```
server {
        listen 80;
        root /var/www/html;
        index index.php index.html index.htm index.nginx-debian.html;
        server_name mi-sitio.com;

        location / {
                try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        }

        location ~ /\.ht {
                deny all;
        }
}
```

Estos es lo que hacen estas directivas y bloques de ubicación:

* `listen`: define en qué puerto escuchará Nginx las peticiones de los visitantes. En este caso, escuchará en el puerto `80` que es el predeterminado para HTTP.
* `root`: define la raiz de documentos en donde se almacenan los archivos de tu sitio web.
* `index`: configura la prioridad de los archivos cuando se solicite un archivo indice si estan disponibles. En este caso el de mayor prioridad es `index.php`. 
* `server_name`: define el bloque de servidor que debe utilizarse para una solicitud enviada al tu servidor. Introduce en esta directiva tu nombre de dominio o tu IP pública. En este caso `mi-sitio.com`.
* `location /`: el primer bloque de ubicación incluye una directiva `try_files`, la cual comprueba la existencia de archivos que coincidan con una solicitud de URI. Si Nginx no puede encontrar el archivo apropiado, se mostrará un error 404.
* `location ~ \.php$`: este bloque de ubicación administra el procesamiento de PHP real orientando Nginx al archivo de configuración `fastcgi-php.conf` y al archivo `php7.2-fpm.sock`, que declara el socket que se asocia con `php-fpm`.
* `location ~ /\.ht`: el último bloque de ubicación maneja archivos `.htaccess`, que Nginx no procesa. Al agregar la directiva deny all, si algunos de los archivos `.htaccess` ingresa de alguna forma a la raiz de documentos, estos no se pondrán a disposición de los visitantes.

Una vez agregado este contenido, guarda y cierra el archivo. 

Para habilitar el nuevo bloque de servidor, crea un enlace simbólico entre el nuevo archivo de configuración de bloques de servidor (en el directorio `/etc/nginx/sites-available/`) y el directorio `/etc/nginx/sites-enabled/`:

```
$ sudo ln -s /etc/nginx/sites-available/mi-sitio.com /etc/nginx/sites-enabled/
```

Por último, desvincula el archivo de configuración predeterminado del directorio `/etc/nginx/sites-enabled/`:

```
$ sudo unlink /etc/nginx/sites-enabled/default
```

Nota: Si necesitas restablecer la configuración predeterminada, puedes hacerlo recreando el enlace simbólico como se muestra a continuación.

```
$ sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
```

Verifica si hay errores de sintaxis en tu nuevo archivo de configuración escribiendo lo siguiente:

```
$ sudo nginx -t
```

Si se notifica algún error, revisa el archivo de configuración de bloques antes de continuar.

Cuando esté listo, recarga Nginx para que tome los cambios realizados:

```
$ sudo systemctl reload nginx
```

## Paso 3 - Crear un archivo PHP para comprobar la configuración

En esta instancia, el servidor web debería estar configurado por completo. Puedes probarlo para validar la ejecución correcta de archivos `.php` a partir del procesador PHP por parte de Nginx.

Para hacerlo, utiliza el editor de texto para crear un archivo PHP de prueba llamado info.php en la raiz de documentos:

```
$ sudo nano /var/www/html/info.php
```

Introduce las líneas que se muestran a continuación en el archivo nuevo. Se incluye código PHP válido que mostrará información sobre su servidor:

```php
<?php
phpinfo();
```

Guarda y cierre el archivo.

Luego, visita esta página en tu navegador web accediendo al nombre de dominio o a la dirección IP pública de su servidor agregando `/info.php` al final:

```
http://nombre_de_dominio_o_IP/info.php
```

Debería ver una página web generada por PHP con información sobre su servidor:

[IMAGEN]

Si logras ver una página con este aspecto, significa que haz configurado el procesamiento de PHP con Nginx satisfactoriamente.

Tras verificar que Nginx represente la página de forma correcta, la mejor opción será eliminar el archivo creamos ya que puede dar indicios sobre la configuración que podrían ser útiles para que usuarios no autorizados intenten ingresar. Siempres puedes recrear este archivo si lo necesitas más adelante.

Por ahora, elimina el archivo introduciendo el siguiente comando:

```
sudo rm /var/www/html/info.php
```

Con esto, dispondrás de un servidor web completamente funcional en tu servidor Ubuntu 18.04.