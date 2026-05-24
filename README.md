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

```
## Configuracion de IP del servidor previo al DNS
+ Asignar una direccion estatica permanente al servidor
```bash
  # Editar el archivo:
  sudo $EDITOR /etc/network/interfaces
 # NOTA: Eliminar o comentar las lineas de dhcp
 # Escribir en la interfaz a editar:
  auto enp0s3 --- (wlp8s0 | enp7s0)
  iface enp0s3 inet static
    address 192.168.100.2
    netmask 255.255.255.0
    gateway 192.168.100.1
    dns-nameservers 8.8.8.8 1.1.1.1
    dns-nameservers 192.168.100.2

  # Reiniciar servicio de networking y verificar IP: 
  sudo systemctl restart networking
  ip a
```
## Instalar el servicio de DNS
```bash
sudo apt install bind9
# Verificar el estado sel servicio:
sudo systemctl stop|start|restart|status named.service
```
+ Configurar el archivo de configuracion
```bash
   # Editar el archivo:
  sudo $EDITOR /etc/bind/named.conf.options
  options {

	directory "/var/cache/bind";
	recursion yes;
	allow-query { localhost; 127.0.0.1;
	192.168.100.2/24; };   ----- la red a la que estes conectado
	forwarders {
		192.168.100.1;   ---- IP del gateway
		8.8.8.8;
    1.1.1.1;
	};
	dnssec-validation no;
	listen-on { 127.0.0.1; 192.168.100.2; };
	listen-on-v6 { ::1; };
};

# Verificar configuracion:
sudo named-checkconf
# Reiniciar demonio:
systemctl daemon-reload
# Reiniciar servicio:
Sudo systemctl restart named.service
```
+ Configurar la zona
```bash
# Editar el archivo
sudo $EDITOR /etc/bind/named.conf.local

  zone "licic.tux"{
	type master;
	file "/etc/bind/db.licic.tux";
	allow-transfer{none;};
};
```
+ Crear la base de datos de la zona
```bash
# Crear el archivo de base de datos:
sudo $EDITOR /etc/bind/db.licic.tux

$TTL 		604800
@			  IN		SOA		licic.tux. root.licic.tux. (
							0422202601		; Serial
							604800		  	; Refresh
							86400			    ; Retry
							2419200		  	; Expire
							604800 )			; Negative Cache TTL

@ 			IN 		NS		ns.licic.tux.
ns			IN		A		  192.169.100.2
@       IN    A     192.168.100.2
www			IN 		A 		192.168.100.2

# Verificar que la zona y la base de datos estan bien configuradas:
sudo named-checkzone licic.tux db.licic.tux
# Reiniciar el servicio
sudo systemctl restart named.service
```
+ Crear la zona inversa
```bash
# Agregar en el archivo:
sudo $EDITOR /etc/bind/named.conf.local

  zone "100.168.192.in-addr.arpa" { --- IP al revez sin la parte de host
	type master;
	file "/etc/bind/db.192.168.100";
};
```
+ Crear la base de datos de la zona inversa
```bash
# Crear el archivo:
sudo $EDITOR /etc/binf/ db.100.168.192

$TTL    604800
@       IN      SOA     licic.tux. root.licic.tux. (
                        2026052101      ; Serial 
                            604800      ; Refresh
                             86400      ; Retry
                           2419200      ; Expire
                            604800 )    ; Negative Cache TTL

@       IN      NS      ns.licic.tux.
2       IN      PTR     licic.tux.
2       IN      PTR     www.licic.tux.
2       IN      PTR     ns.licic.tux.

# Verificar que la zona inversa y su base de datos estan bien configuradas
sudo named-checkzone 100.168.192.in-addr.arpa db.100.168.192
# Reiniciar el servicio
sudo systemctl restart named.service
```
+ Comprobar que el DNS funciona
  - En los clientes con Linux, agrear la IP al archivo
    ```bash
    sudo $EDITOR /etc/resolv.conf
    nameserver 192.168.100.2
    ```
  - En la barra del navegador verificar si el DNS resuleve el dominio
    http://www.licic.tux
    http://licic.tux

  - Comprobar el DNS desde la linea de comandos
    ```bash
    dig www.licic.tux @192.168.100.2
    nslookup www.licic.tux
    ```
  - Comprobar en linea de comandos (zona inversa)
    ```bash
    nslookup 192.168.100.2
    ```
## Servidor de proxy


