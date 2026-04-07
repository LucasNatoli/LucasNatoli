# Módulos PHP recomendados para Wordpress

El núcleo de WordPress utiliza módulos de PHP. En caso de que una módulo recomendada no este presente, WordPress deberá realizar mas operaciones para llevar a cabo la tarea del módulo, o en el peor de los casos, quitar alguna funcionalidad.


Para operar correctamente, Wordpress necesita los siguientes

* json – Utilizada para la comunicación con otros servidores. 
* mysqli – Conectarse a MySQL para interacción con la base de datos.


Por lo tanto la siguiente se recomienda la siguiente lista de módulos:

* curl – Realiza operaciones con peticiones remotas.
* dom – Utilizada para validar el contenido de Widget de Texto y configuración automática del IIS7 +.
* exif – Trabajar con metadatos almacenados en las imágenes.
* fileinfo – Utilizada para detectar el tipo mime de los archivos que se suben al site. 
* hash – Utilizada para encriptación, incluyendo passwords y paquetes de actualización.
* mbstring – Utilizada para manejar texto UTF8 correctamente. 
* openssl – Permitir conexiones SSL a otros hosts.
* xml – Utilizado para análisis XML, por ejemplo desde un sitio de un tercero.
* zip – Utilizado para descomprimir Plugins, Themes y paquetes de actualización de WordPress. 


* pcre – Aumentar la performance en la comparación de patrones en la búsqueda de código. Él módulo PCRE forma parte del núcleo de PHP a partir de la version 7 por lo que no es necesaria ninguna instalación extra si utilizas una version mayor o igual a PHP7.0.

* imagick – Provee mejor calidad de imagenes en la subida de medios (ver  WP_Image_Editor esta llegando! para mas detalle). Mejorar el cambio de tamaño de imagenes (para imagenes pequeñas) y brindar soporte para miniaturas de PDF, cuando Ghost Script tambien este disponible. 


Para instalar estos modulos ingresa el comando:

```
sudo apt install php-json php-mysql php-curl php-dom php-exif php-fileinfo php-mbstring php-xml php-zip 
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

