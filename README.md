# Servidor-ElPaisano
Configuraciones necesarias para poder configurar correctamente el servidor del Paisano


Para poder configurar correctamente este servidor es necesario descargar y activar apache2 (sudo apt-get install apache2) y activarlo (sudo systemctl start apache), es necesario también añadir el archivo "index.html" en el directorio /var/www/html, así como crear 2 carpetas extras dentro de este directorio: /moodle /juegos. Quedando de la siguiente manera: /var/www/html/index.html, /var/www/html/moodle/, /var/www/html/juegos Para que no haya errores a la hora de ejecutar el código y todo conecte correctamente. También es necesario tener conocimiento de la dirección IP de la máquina (o configurarla manualmente para que coincida con la dirección IP que usará el servidor)
Para moodle se debe descargar php para apache (sudo apt install php libapache2-mod-php) y sus extensiones (sudo apt install php-mysql php-gd php-xml php-curl php-mbstring php-zip php-intl php-soap) 
Si por algún motivo falla o no se activa solo, usar los comandos: sudo a2enmod php8.x  # (Cambia x por la versión instalada, ej. php8.2) y sudo systemctl restart apache2

Una vez configurado todo lo de apache hay que preparar una base de datos para moodle, en este caso usarmeos mariadb, el cuál se descargará con los comandos: sudo apt install mariadb-server.
Una vez descargado se debe ejecutar el siguiente comando: sudo mysql_secure_installation para configurar la contraseña de root y eliminar los usuarios anonimos.

Para asegurar que apache está ejecutando correctamente php vamos a agregar un archivo de prueba: /var/www/html/prueba.php, pegaremos el siguiente comando: <?php phpinfo(); ?> y entraremos a http://localhost/prueba.php, si entra correctamente, significa que está todo correctamente configurado.

Para descargar moodle se usará el siguiente comando, preferiblemente en el directorio: /var/www/html, al descargar el archivo .tgz y descomprimirlo se creará automáticamente el directorio /moodle.
Comando: sudo wget https://download.moodle.org/download.php/direct/moodle/moodle-latest.tgz
Para descomprimir: sudo tar -zxvf moodle-latest.tgz
Borrar el archivo .tgz: sudo rm moodle-latest.tgz

Se debe crear la carpeta "moodledata" fuera de /html por seguridad: sudo mkdir /var/www/moodledata

Una vez creada, se le asignarán los permisos para que Apache (www-data) pueda escribir en ella sudo chown -R www-data:www-data /var/www/moodledata sudo chmod -R 777 /var/www/moodledata
Ahora hay que darle permiso a Apache para que lea los archivos de moodle: sudo chown -R www-data:www-data /var/www/html/moodle, sudo chmod -R 755 /var/www/html/moodle

Antes de poder usar moodle debemos configurar la base de datos, entrando en sudo mysql -u root -p, ejecutaremos los siguientes comandos: 
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'TuPasswordSegura'; #Puedes cambiar la contraseña por algo más seguro
GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

Configuración de red estática para el servidor:
Para configurar la red estática se deben seguir 2 sencillos pasos muy importantes para esto.
  -Paso 1: Ejecutar el comando "ip route | grep default" y buscar "default via xxx.xxx.x.xxx dev enp0s3 (o similares) para poder saber cuál es el gateway (dirección IP del router).
  -Paso 2: Editar el archivo en el siguiente directorio: /etc/network/interfaces y buscar la línea que coincida con la interfaz de salida (en este caso enp0s3) y cambiar los siguientes comandos
  iface enp0s3 inet static
    address 192.168.1.131
    netmask 255.255.255.0
    gateway 192.168.1.254    # <--- Cambiar por la dirección IP default revisada en el paso 1
