# Snips

Lista de comandos y scripts de uso regular en el entorno de desarrllo.
Para ayudar a las mentes dispersas por la creatividad y eliminar la necesidad de los memoriosos tragalibros. 

+ [Apache](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#apache)
+ [Mysql](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#mysql)
+ [Wordpress](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#wordpress)
+ [Shell Commands](https://github.com/LucasNatoli/LucasNatoli/blob/master/snips.md#shell-commands)

## Apache

Gracefully reload

```bash
sudo service apache2 reload
```


Virtual hosts: Definidos en `/etc/apache2/sites-available` del host. Es recomendado para Ubuntu.

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

Dump database

```bash
mysqldump -u username -p db1 --single-transaction --quick --lock-tables=false > db1-backup-$(date +%F).sql
```

GRANT access usuario a database

```bash
mysql> GRANT ALL PRIVILEGES ON database_name.* TO 'username'@'localhost';
```

Mostrar todos los usuarios del servidor

```sql
SELECT User FROM mysql.user;
```

## Wordpress

Instalar usando curl 
(actualmente usamos: https://es.wordpress.org/wordpress-4.9.9-es_ES.zip)

```bash
curl -O https://wordpress.org/[version].zip
unzip [version].zip 
mv wordpress/* [destino]
rm [version].zip 
```

## Shell Commands

```bash
zip -r name_of_your_directory.zip name_of_your_directory
```