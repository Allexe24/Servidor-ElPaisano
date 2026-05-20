# Sistema El Paisano - Servidor Educativo Comunitario (Offline)

**El Paisano** es un proyecto de infraestructura de red y software diseñado para proveer educación y entretenimiento a comunidades remotas sin acceso a Internet. Utiliza un servidor virtualizado basado en Debian (Linux) para alojar un portal cautivo, un LMS (Moodle) y un repositorio de videojuegos locales.

---

## Arquitectura del Sistema (Stack Tecnológico)
* **Sistema Operativo:** Debian Linux (Virtualizado en Oracle VirtualBox)
* **Servidor Web:** Apache2
* **Base de Datos:** MariaDB
* **Lenguaje Backend:** PHP (Stack LAMP)
* **LMS:** Moodle 5.1

---

# Guía de Instalación y Despliegue

### Fase 1: Instalación de Dependencias (Stack LAMP)
Actualización del sistema e instalación de los paquetes necesarios para que el servidor web y la base de datos funcionen correctamente.

```bash
# Actualizar repositorios del sistema
sudo apt update && sudo apt upgrade -y

# Instalar Apache, MariaDB y PHP con sus extensiones clave para Moodle
sudo apt install apache2 mariadb-server php libapache2-mod-php php-cli php-mysql php-xml php-mbstring php-curl php-zip php-gd php-intl -y

# Reiniciar servicios para aplicar cambios
sudo systemctl restart apache2

# Entrar al gestor de MariaDB
sudo mariadb -u root -p

-- Ejecutar dentro de la consola de MariaDB:
CREATE DATABASE basePaisano DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'tu_contraseña_segura';
GRANT ALL PRIVILEGES ON basePaisano.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

# Se creó el archivo en la raíz del servidor web
sudo nano /var/www/html/index.html

# Crear la carpeta para los datos de Moodle (fuera del acceso público web por seguridad)
sudo mkdir -p /var/www/moodledata
sudo chown -R www-data:www-data /var/www/moodledata
sudo chmod -R 0777 /var/www/moodledata

# Extraer Moodle en el directorio web (asumiendo que el .tgz ya fue descargado)
sudo tar -zxvf moodle-latest.tgz -C /var/www/html/
sudo chown -R www-data:www-data /var/www/html/moodle
sudo chmod -R 0755 /var/www/html/moodle

# Modificar archivo de interfaces de red
sudo nano /etc/network/interfaces

# Modificación de parámetros para red estática:
auto enp0s3 <--- Notar que esta interfaz puede cambiar según el dispositivo, verificar
iface enp0s3 inet static
  address 192.168.1.131 <--- Puede variar según el router
  netmask 255.255.255.0
  gateway 192.168.1.1 <--- Puede variar según el router

# Reiniciar el sistema de red
sudo systemctl restart networking

# Comandos
sudo ufw allow 80/tcp
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw reload
sudo ufw status # Verificar que diga "Active"

# Archivo a modificar
sudo nano /var/www/html/moodle/config.php

# Comando para cambiar la dirección IP dinámica de Moodle
$CFG->wwwroot   = 'http://' . $_SERVER['HTTP_HOST'] . '/moodle';

# Descargar e integrar el logotipo de la UV en el directorio raíz
sudo wget -O /var/www/html/logo_uv.png "URL_DEL_ESCUDO_OFICIAL"
sudo chmod 644 /var/www/html/logo_uv.png

# Crear el directorio base para la suite de juegos
sudo mkdir -p /var/www/html/juegos

# Otorgar permisos de propiedad y lectura al servidor web Apache (www-data)
sudo chown -R www-data:www-data /var/www/html/juegos
sudo chmod -R 755 /var/www/html/juegos

