# Módulos PHP recomendados para Wordpress

El núcleo de WordPress utiliza módulos de PHP. En caso de que una módulo recomendada no este presente, WordPress deberá realizar mas operaciones para llevar a cabo la tarea del módulo, o en el peor de los casos, quitar alguna funcionalidad.

Por lo tanto la siguiente se recomienda la siguiente lista de módulos:

* curl – Realiza operaciones con peticiones remotas.
* dom – Utilizada para validar el contenido de Widget de Texto y configuracion automática del IIS7 +.
* exif – Trabajar con metadatos almacenados en las imagenes.
* fileinfo – Utilizada para detectar el tipo mime de los archivos que se suben al site. 
* hash – Utilizada para encriptacion, incluyendo passwords y paquetes de actualizacion.
* json – Utilizada para la comunicacion con otros servidores. 
* mbstring – Utilizada para manejar texto UTF8 correctamente. 
* mysqli – Conectarse a MySQL para interaccion con la base de datos.
* libsodium – Validar Firmas y proveer bytes aleatorios seguros.
* openssl – Permitir conexiones SSL a otros hosts.
* pcre – Aumentar la performance en la comparacion de patrones en la busqueda de codigo. Él módulo PCRE forma parte del núcleo de PHP a partir de la version 7 por lo que no es necesaria ninguna instalación extra si utilizas una version mayor o igual a PHP7.0.
* imagick – Provee mejor calidad de imagenes en la subida de medios (ver  WP_Image_Editor esta llegando! para mas detalle). Mejorar el cambio de tamaño de imagenes (para imagenes pequeñas) y brindar soporte para miniaturas de PDF, cuando Ghost Script tambien este disponible. 
* xml – Utilizado para análisis XML, por ejemplo desde un sitio de un tercero.
* zip – Utilizado para descomprimir Plugins, Themes y paquetes de actualizacion de WordPress. 

Para instalar estos modulos ingresa el comando:

```
sudo apt install php-curl php-dom php-exif php-common php-json php-mbstring php-mysql php-libsodium libapache2-mod-php  php-imagick php-xml php-zip 
```


Para hacer la lista mas exhaustiva, la siguiente es una lista de modulo PHP que WordPress puede llegar a utilzar en ciertas situaciones o si otros módulos no se encuentras disponibles. Son opcionales y no del todo necesarios en un entorno optimizado, pero su instalacion no hace daño. 

* filter – Utilizado para filtrar el ingreso de informacion de los usuarios. 
* gd – Si Imagick no se encuentra presente, la librería gráfica GD se utiliza como funcionalidad limitada para el manejo de imágenes. 
* iconv – Utilizada para conversión entre conjuntos de caracteres. 
* mcrypt – Generar bytes aleatorios cuando libsodium y /dev/urandom no se encuentran disponibles.
* simplexml – Utilizada para análisis XML.
* xmlreader – Used for XML parsing.
* zlib – Compresion y descompresion Gzip.

Las siguientes módulos se utilizan para realizar cambion en los archivos, tales como actualizaciones e instalacion de prugins y themes, cuando los archivos en el servidor no sean escribibles.

* ssh2
* ftp sockets (Para cuando la módulo ftp no este disponible)

La priorida del transpote es: Lectura/Escritura Directa, SSH2, Extension PHP FTP, FTP implementado con Sockets y FTP implementado a traves de PHP. 

