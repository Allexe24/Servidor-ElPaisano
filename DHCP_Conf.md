## Servidor de DHCP
+ Mantener la IP estatica más una IP por dhcp
```bash
# Editar el archivo
 sudo $EDITOR /etc/network/interfaces
# Agregar interfaz por dhcp
auto enp0s3
iface enp0s3 inet dhcp

# Agregar la interfaz virtual
auto enp0s:0
iface enp0s3:0 inet static
        address 192.168.100.2
        netmask 255.255.255.0
# Reiniciar el Servicio
sudo systemctl restart networking
# O limpoiar IP
sudo ip addr flush dev enp0s3
```
+ Mantener solo la IP estatica
```bash
 auto enp0s3 --- (wlp8s0 | enp7s0)
  iface enp0s3 inet static
    address 192.168.100.2
    netmask 255.255.255.0
    gateway 192.168.100.1
```

## Instalar el servicio
```bash
# Instalar el servicio
sudo apt install isc-dhcp-server
# El servicio fallara tras la instalacion debido a que no esta configurado

# Editar la interfaz a escuchar
sudo $EDITOR /etc/default/isc-dhcp-server
# Cambiar en INTERFACESv4:
INTERFACESv4="enp0s3"
```
### Configurar el rango de direcciones y parametros de red
```bash
#Configurar el archivo principal de configuracion
sudo $EDITOR /etc/dhcp/dhcpd.conf

subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.10 192.168.100.100;      # Rango de IPs para los clientes
    option routers 192.168.100.1;              # La puerta de enlace (Gateway) de la red
    option subnet-mask 255.255.255.0;          # Máscara de subred
    option domain-name-servers 192.168.100.2;  # Tu propio servidor DNS (o puedes usar 8.8.8.8)
    option domain-name "licic.tux";            # Tu dominio local
    default-lease-time 600;
    max-lease-time 7200;
}

# Reiniciar servicio
sudo systemctl restart isc-dhcp-server
# Vrificar el estado del servicio
sudo systemctl status isc-dhcp-server

# Verificar funcionamiento en el servidor
sudo journalctl -u isc-dhcp-server -f

# Solicitar nueva IP desde cliente linux
nmcli device connect wlp8s0

# Forzar a clienta a tomar una direccion IP del servidor
sudo dhclient -v -s 192.168.100.2 "interfaz"



