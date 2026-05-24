# Configuración de Proxy


## Instalación de Squid

sudo apt update
sudo apt install squid

## Respaldar configuración 
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.backup

## Configuración Principal
sudo nano /etc/squid/squid.conf


El sistema se basa en la modificación y creación de dos archivos principales dentro del directorio de configuración de Squid (/etc/squid/):

1. **`squid.conf`**: El archivo de configuración principal de Squid. Aquí se definen las reglas de control de acceso (ACLs), los puertos de escucha y la lógica de bloqueo.
2. **`sitios_bloqueados.txt`**: Una lista negra en texto plano que contiene los dominios de los sitios web que deben ser restringidos para los menores.

### 1. `sitios_bloqueados.txt`
Este archivo debe crearse en el mismo directorio que squid.conf (/etc/squid/sitios_bloqueados.txt). Contiene los nombres de dominio de las páginas que se desean bloquear, un elemento por línea.

**Ejemplo de contenido:**
```text
.x.com
.reddit.com
.tiktok.com
.apuestas.com
```



## 2. DEFINICIÓN DE ACLs (Access Control Lists)

### ACL estándar para la red local
acl localnet src 192.168.100.0/24

### ACL Personalizada:
acl bloqueados dstdomain "/etc/squid/sitios_bloqueados.txt"

## 3. REGLAS DE CONTROL DE ACCESO (http_access)
http_access deny bloqueados

### Permitir el acceso a la red local
http_access allow localnet

### Denegar cualquier otra petición que no cumpla las condiciones anteriores
http_access deny all



# Gestión del Servicio


