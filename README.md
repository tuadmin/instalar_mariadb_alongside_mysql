# instalar_mariadb_alongside_mysql
Estas son mis memorias para instalar MARIADB dentro de un servidor que contiene MYSQL con mucha informacion
esta URl contiene informacion , de una version anterior
https://mariadb.com/kb/en/library/installing-mariadb-alongside-mysql/

el problema principal en la instalacion es el hecho de crear todo y de error en un archivo .PID que tiene el mismo nombre que el mysql y causa un conflicto 

lo primero es descargar  la version 10 de maria db

https://downloads.mariadb.org/interstitial/mariadb-10.3.8/bintar-linux-x86_64/mariadb-10.2.16-linux-x86_64.tar.gz/from/http%3A//mirror.edatel.net.co/mariadb/?serve&change_mirror_from=1
```
cd /opt
wget -O mariadb-10.2.16-linux-x86_64.tar.gz https://downloads.mariadb.com/MariaDB/mariadb-10.2.16/bintar-linux-x86_64/mariadb-10.2.16-linux-x86_64.tar.gz
```
mientras descarga se procede a realizar la creacion del usuario mariadb y el grupo descritas en la documentacion y el enlace

descargamos el archivo

```
tar -xzvf mariadb-10.2.16-linux-x86_64.tar.gz
```

-creamos e la carpeta de datos /opt/mariadb-data 
-link de la carpeta *mariadb-10.2.16-linux-x86_64* a solo mariadb
-creamos los grupos y usuarios
-copiamos el archivo de configuracioon

```
mkdir mariadb-data
ln -s mariadb-10.2.16-linux-x86_64 mariadb

groupadd --system mariadb
useradd -c "MariaDB Server" -d /opt/mariadb -g mariadb --system mariadb

chown -R mariadb:mariadb mariadb-10.2.16-linux-x86_64/
chown -R mariadb:mariadb mariadb-data/
chown -R mariadb:mariadb mariadb

cp mariadb/support-files/my-medium.cnf mariadb-data/my.cnf
chown mariadb:mariadb /opt/mariadb-data/my.cnf
```
**editamos el my.cnf**
```
[client]
port		= 3307
socket		= /opt/mariadb-data/mariadb.sock

[mysqld]
datadir         = /opt/mariadb-data
basedir         = /opt/mariadb
port		= 3307
socket		= /opt/mariadb-data/mariadb.sock
user            = mariadb
```
coopiamos los archivos  para que se ejecute como demonio
```
cp mariadb/support-files/mysql.server /etc/init.d/mariadb
chmod +x /etc/init.d/mariadb
```
-Editamos /etc/init.d/mariadb reemplazamos **mysql** por **mariadb** como a continuación:
```
- # Provides: mysql
+ # Provides: mariadb
- basedir=
+ basedir=/opt/mariadb
- datadir=
+ datadir=/opt/mariadb-data
- lock_file_path="$lockdir/mysql"
+ lock_file_path="$lockdir/mariadb"
```

La parte más complicada serán los últimos cambios en este archivo. Necesitas decirle a mariadb que use solo un archivo cnf. En la sección de inicio después de  **$bindir/mysqld_safe** agregue **--defaults-file=/opt/mariadb-data/my.cnf**. Finalmente las líneas deberían verse así:

```
$bindir/mysqld_safe --defaults-file=/opt/mariadb-data/my.cnf --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null 2>&1 &
```
El mismo cambio debe realizarse en el comando mysqladmin a continuación en la función **wait_for_ready()** para que el comando mariadb start pueda escuchar correctamente el inicio del servidor. En la función **wait_for_ready()**, después de **$bindir/mysqladmin** agregar --defaults-file=/opt/mariadb-data/my.cnf. Las líneas deben verse así:
```
wait_for_ready () {
[...]
    if $bindir/mysqladmin --defaults-file=/opt/mariadb-data/my.cnf ping >/dev/null 2>&1; then
```

-Ejecute mysql_install_db dándole explícitamente el archivo my.cnf como argumento:
```
cd /opt/mariadb
scripts/mysql_install_db --defaults-file=/opt/mariadb-data/my.cnf
```

Ahora puedes iniciar el demonio de la base de datos MariaDB
## Existe un error en este apartado saltese hasta el final ERRORES 
```
# /etc/init.d/mariadb start
Starting MySQL...                                          [  OK  ]
```
una vez que hayas probado que inicia se detiene y reinicia con los comandos correspondientes puedes pasar a que se inicie automaticamente junto al sistema, solo cuando ya probaste todo lo anterior probando que reinicie tambien
```
# cd /etc/init.d
# chkconfig --add mariadb 
# chkconfig --levels 3 mariadb on
```
#ERRORES
en teoria debera de funcionar pero en la version que use y la de este tutorial existe un problema el PID que toma como nombre el del mysql, el cual le dara un falso postiivo al iniciar el servicio, 

busque
```
case "$mode" in
  'start')
    # Start daemon

    # Safeguard (relative paths, core dumps..)
    cd $basedir

    echo $echo_n "Starting MariaDB"
```
y agreguele
mysqld_pid_file_path=/opt/mariadb-data/mariadb-daemon.pid
```
mysqld_pid_file_path=/opt/mariadb-data/mariadb-daemon.pid
case "$mode" in
  'start')
    # Start daemon

    # Safeguard (relative paths, core dumps..)
    cd $basedir

    echo $echo_n "Starting MariaDB"
```
puede volver donde se quedo
