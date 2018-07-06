# instalar_mariadb_alongside_mysql
Estas son mis memorias para instalar MARIADB dentro de un servidor que contiene MYSQL con mucha informacion
esta URl contiene informacion , de una version anterior
https://mariadb.com/kb/en/library/installing-mariadb-alongside-mysql/

el problema principal en la instalacion es el hecho de crear todo y de error en un archivo .PID que tiene el mismo nombre que el mysql y causa un conflicto 

lo primero es descargar  la version 10 de maria db

https://downloads.mariadb.org/interstitial/mariadb-10.3.8/bintar-linux-x86_64/mariadb-10.2.16-linux-x86_64.tar.gz/from/http%3A//mirror.edatel.net.co/mariadb/?serve&change_mirror_from=1
```
cd /opt
wget -O mariadb-10.2.16-linux-x86_64.tar.gz https://downloads.mariadb.org/interstitial/mariadb-10.3.8/bintar-linux-x86_64/mariadb-10.2.16-linux-x86_64.tar.gz/from/http%3A//mirror.edatel.net.co/mariadb/?serve&change_mirror_from=1
```
mientras descarga se procede a realizar la creacion del usuario mariadb y el grupo descritas en la documentacion y el enlace

descargamos el archivo

```
tar -xzvf mariadb-10.2.16-linux-x86_64.tar.gz
```

*creamos e la carpeta de datos /opt/mariadb-data 
* link de la carpeta *mariadb-10.2.16-linux-x86_64* a solo mariadb
*creamos los grupos y usuarios
*copiamos el archivo de configuracioon
```
mkdir mariadb-data
ln -s mariadb-10.2.16-linux-x86_64 mariadb

groupadd --system mariadb
useradd -c "MariaDB Server" -d /opt/mariadb -g mariadb --system mariadb

chown -R mariadb:mariadb mariadb-10.2.16-linux-x86_64/
chown -R mariadb:mariadb mariadb-data/
chown -R mariadb:mariadb mariadb/

cp mariadb/support-files/my-medium.cnf mariadb-data/my.cnf
```
