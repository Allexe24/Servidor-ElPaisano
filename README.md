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

#Guía de Instalación y Despliegue#

Fase 1: Instalación de Dependencias (Stack LAMP)
Actualización del sistema e instalación de los paquetes necesarios para que el servidor web y la base de datos funcionen correctamente.

# Actualizar repositorios del sistema
sudo apt update && sudo apt upgrade -y

# Instalar Apache, MariaDB y PHP con sus extensiones clave para Moodle
sudo apt install apache2 mariadb-server php libapache2-mod-php php-cli php-mysql php-xml php-mbstring php-curl php-zip php-gd php-intl -y

# Reiniciar servicios para aplicar cambios
sudo systemctl restart apache2

Fase 2: Configuración de la base de datos
Creación de la base de datos y el usuario dedicado para la instalación de moodle.

# Entrar al gestor de MariaDB
sudo mariadb -u root -p

-- Ejecutar dentro de la consola de MariaDB:
CREATE DATABASE basePaisano DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'tu_contraseña_segura';
GRANT ALL PRIVILEGES ON basePaisano.* TO 'moodleuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

Fase 3: Creación del portal de entrada (index.html) 
Se desarrolló una página de aterrizaje (Landing Page) utilizando rutas relativas para evitar que los enlaces se rompan al cambiar la IP del servidor en diferentes redes.

# Se creó el archivo en la raíz del servidor web
sudo nano /var/www/html/index.html <--- Ver index.html en este repositorio

Fase 4: Despliegue de moodle y permisos de directorio
Se estructuraron los directorios para alojar la plataforma y se asignaron los permisos de escritura necesarios para que Apache pueda gestionar los archivos.

# Crear la carpeta para los datos de Moodle (fuera del acceso público web por seguridad)
sudo mkdir -p /var/www/moodledata
sudo chown -R www-data:www-data /var/www/moodledata
sudo chmod -R 0777 /var/www/moodledata

# Extraer Moodle en el directorio web (asumiendo que el .tgz ya fue descargado)
sudo tar -zxvf moodle-latest.tgz -C /var/www/html/
sudo chown -R www-data:www-data /var/www/html/moodle
sudo chmod -R 0755 /var/www/html/moodle

Fase 5: Configuración de red (IP estática)
Para manejar entornos estáticos se modificó la configuración de la interfaz de red.

# Modificar archivo de interfaces de red
sudo nano /etc/network/interface

# Modificación de parametros para red estática:
auto enp0s3 <--- Notar que esta interfaz puede cambiar según el dispositivo, verificar
iface enp0s3 inet static
  address 192.168.1.131 <--- Puede variar según el router
  netmask 255.255.255.0
  gatway 192.168.1.1 <--- Puede variar según el router

# Reiniciar el sistema de red
sudo systemctl restart networking

Fase 6: Reglas de Firewall (UFW)
Se configuró el cortafuegos para permitir únicamente el tráfico web (Puerto 80) y la administración remota (Puerto 22)

# Comandos
sudo ufw allow 80/tcp
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw reload
sudo ufw status # Verificar que diga "Active"

Fase 7: Arquitectura dinámica en PHP
Al cambiar de IP, moodle bloquea el acceso debido a redirecciones estáticas. Se inyectó una variable global en el código PHP para que la plataforma detecte automáticamente el host dinámico.

# Archivo a modificar
sudo nano /var/www/html/moodle/config.php

# Comando para cambiar la dirección IP dinámica de moodle
$CFG->wwwroot   = 'http://' . $_SERVER['HTTP_HOST'] . '/moodle';

