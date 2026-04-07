# Snips

Lista de comandos y scripts de uso regular en el entorno de desarrllo.
Para ayudar a las mentes dispersas por la creatividad y eliminar la necesidad de los memoriosos tragalibros. 

+ [Apache](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#apache)
+ [Mysql](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#mysql)
+ [Wordpress](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#wordpress)
+ [Shell Commands](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#shell-commands)
+ [Variables de Entorno](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#variables-de-entorno)

## Apache

### Gracefully reload

```bash
sudo service apache2 reload
```

### Virtual hosts: Definidos en `/etc/apache2/sites-available` del host. Es recomendado para Ubuntu.

```bash
# Ensure that Apache listens on port 80
Listen 80
<VirtualHost *:80>
    DocumentRoot "/www/example1"
    ServerName www.example.com

    # Other directives here
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot "/www/example2"
    ServerName www.example.org

    # Other directives here
</VirtualHost>
```

## MySql 

### Dump database

```bash
mysqldump -u username -p db1 --single-transaction --quick --lock-tables=false > db1-backup-$(date +%F).sql
```

### Otorgar privilegion a un usuario sobre todos los elementos de una base de datos

```bash
mysql> GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';
```

### Mostrar todos los usuarios del servidor

```sql
SELECT User FROM mysql.user;
```

## Wordpress

### Instalar usando curl 
(actualmente usamos: https://es.wordpress.org/wordpress-4.9.9-es_ES.zip)

```bash
curl -O https://wordpress.org/[version].zip
unzip [version].zip 
mv wordpress/* [destino]
rm [version].zip 
```

## Shell Commands

### Comprimir una carpeta y sus subcarpetas 

```bash
zip -r name_of_your_directory.zip name_of_your_directory
```

### Determinar el espacio libre de un volumen 

```bash
df
```

### Determinar el tamaño total del un directorio

El comando `du` sumariza el uso de disco para cada archivo de un directorio, por ej.:

```bash
du -hs /path/to/directory
```

* `-h` para obtener los numeros en formato "legible para humanos", por ejemplo 140M en lugar de 143260 (tamaño en KBytes)
* `-s` para sumarizar el total, de otra manera se obtiene no solo el tamaño del directorio sino tambien el de cada subdirectorio. 

Utilizando -h se puede ordenar el resultado por el valor del tamaño "legible para humanos":

```bash
du -h | sort -h
```

Si quieres evitar el listado recursivo de archivos y directorios, puedes utilizar el parametro `--max-depth` para limitar cuantos items se mostraran. Generalmente, `--max-depth=1`

```bash
du -h --max-depth=1 /path/to/directory
```

## Variables de Entorno

Para agregar variables de entorno en Ubutnu (18.0.4) se necesita agregar un archivo de script en la carpeta ```/etc/profile.d```. Se necesitan privilegios de root por lo cual debemos usar el comando ```sudo touch nuevo_archivo.sh``` para poder crear el archivo de script. El contenido del archivo debe ser algo como:

```
export MYPLANT_DB_NAME=my_plants
export MYPLANT_DB_USER=my_plants
export MYPLANT_DB_PASSWORD=my_plants
export MYPLANT_DB_HOST=localhost
export MYPLANT_DB_PORT=3006
```
Una vez guardado el archivo de script, se puede ejecutar ```source /etc/profile``` para que se corran todos los scripts de inicio. 

Para verificar que las variables esten cargadas correctamente se puede ejectuar por ejemplo el comando:
```
echo $MYPLANT_DB_NAME
```
