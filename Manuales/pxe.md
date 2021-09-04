# Servidor de arranque de entorno de ejecución, previo al arranque

---

## Contenido

- [Servidor de arranque de entorno de ejecución, previo al arranque](#servidor-de-arranque-de-entorno-de-ejecución-previo-al-arranque)
  - [Contenido](#contenido)
  - [Que es un servidor PXE?](#que-es-un-servidor-pxe)
  - [Configuracion General](#configuracion-general)
  - [Referencias](#referencias)

---

## Que es un servidor PXE?

Es una interfaz cliente-servidor que permite que los equipos de una red se inicien desde el servidor antes de implementar la imagen de PC obtenida en oficinas locales y remotas, para clientes habilitados para PXE. El arranque de red PXE se realiza utilizando protocolos cliente-servidor como DHCP (Dynamic Host Configuration Protocol) y TFTP (Trivial File Transfer Protocol). PXE estará habilitado de forma predeterminada en todos los equipos. **Esto ayuda en la carga y el inicio de los archivos de arranque para el equipo cliente**.[^1].

A continuacion, se configurara un sistema de instalación con PXE/TFTP

## Configuracion General

Dado que en nuestra red local ya tenemos un servicio DHCP por nuestro provedor de internet, crear un escenario aislado de la red doméstica o empresarial, mediante maquinas virtuales.

> [!WARNING]
> Asegurece de apagar las maquinas virtuales

Para ello, es necesario establecer un nombre de la nueva red asignada a la maquina virtual que provera el servicio desde nuestro **VagranFile**

```ruby
Vagrant.configure("2") do |config|
  config.vm.define :[maquina] do |[maquina]|
    ...
    [maquina].vm.network :private_network, ip: "255.255.255.255", virtualbox__intent: "[lan_name]"
    ...
  end
  ...
end
```

Una vez establecida la configuracion, levantamos nuestra maquina y establecemos una conexion `ssh` a la maquina servidora.

```apache
vagrant up
vagrant ssh [maquina_virtual]
```

Ya en la maquina virtual servidora, crearemos nuestro servicios **TFTP** y **DHCP** los cuales son necesarios para la creacion del servicio, mediante el paquete `dnsmasq`

```apache
yum update && yum upgrade && yum install dnsmasq
```

> [!IMPORTANT]
> Es necesario tener instalado previamente `yum`

Ahora, configuramos el paquete de servicios mediante `vim`

```apache
vim /etc/dnsmasq.conf
```

> [!IMPORTANT]
> Es necesario tener instalado previamente `vim`

De todas las configuraciones, nos interesaremos en las siguientes, dado que nos permitiran:

- Establecer un rango de conexiones libres en nuestra red que los clientes van a usar para arrancar a través de la red
- Establecer una descarga inmediata
- Habilitar el servicio TFTP
- Especificar el directorio que contendrá los ficheros del servicio

```apache
...
dhcp-range=192.168.100.50,192.168.100.150,255.255.255.0,12h
...
dhcp-boot=pxelinux.0
...
enable-tftp
...
tftp-root=/srv/tftp
```

No olvidemos, crear la ruta establecida paa la conexion `tftp`

```apache
mkdir /srv/tftp
```

Ahora, activamos los servicios `dnsmasq` y verificamos

```apache
service dnsmasq start
service dnsmasq status
```

Una vez que los servicios se inicien correctamente, procedemos a descargar los ficheros necesarios para servir, que en este caso sera de **Debian Buster**. Para ello:

- Ingresamos al directorio donde estara nuestro fichero `cd`
- Descargamos el comprimido mediante `wget`
- Descomprimimos mediante el comando `tar`
- Verificamos el contenido `ls`

```apache
cd /srv/tftp
wget http://ftp.debian.org/debian/dists/buster/main/installer-amd64/current/images/netboot/netboot.tar.gz
tar -zxf netboot.tar.gz && rm netboot.tar.gz
ls -l
```

Luego, creamos una maquina nueva, mediante `ORACLE VM` la cual contenga:

- En nuestro privilegios de configuracion:
  - El orden de arranque en Red por preferencia
    - Red
    - Disco duro
    - Habilitar I/O APIC
    - Reloj hardware en tiempo UTC
- En los privilegios de red:
  - Se establesca la red interna la red lan definida para la maquina virtual en el `VagrantFile`

Posterior, iniciamos la nueva maquina donde notaremos la solicitud de conexión a la red lan que espera una respuesta por DHCP.

Finalmente, obtendremos na pantalla de fondo azul que nos solicitara la confirmacion de instalacion del sistema operativo [^2]

---

## Referencias

[^1]: [Preboot Execution Enviornment (PXE) Boot Server](https://www.manageengine.com/products/os-deployer/pxe-preboot-execution-environment.html)
[^2]: [Sistema de instalación PXE/TFTP](https://www.alvarovf.com/sistemas/2020/10/17/instalacion-automatica-pxe.html)
