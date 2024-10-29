
# Práctica (1 / 3): Instalación/migración de aplicaciones web PHP

## Escenario

Crearemos un escenario con vagrant, kvm o openstack con las siguientes características:

Dos máquinas virtuales que se llamen servidor_web_tunombre y servidor_bd_tunombre.

Las máquinas estarán conectadas a una red que les proporcione salida a internet.

Las dos máquinas están conectadas entre si por una red muy aislada.

### Archivo Vagrant (Vagrantfile)
Con lo que se nos quedará un esqueleto de la siguiente manera:

```
vagrant.configure("2") do |config|

  # Servidor Web
  config.vm.define :web do |web| # Cambiado a "web"
    web.vm.box = "debian/bookworm64" # Imagen de Debian 12 (bookworm)
    web.vm.hostname = "web" # Nombre del host simplificado
    web.vm.synced_folder ".", "/vagrant", disabled: true
    
    # Red pública (con salida a Internet)
    web.vm.network :public_network,
      :dev => "br0", # Interfaz de puente
      :mode => "bridge",
      :type => "bridge" 

    # Red privada (muy aislada)
    web.vm.network :private_network,
      :libvirt__network_name => "red_intra", # Nombre de la red privada
      :libvirt__dhcp_enabled => false, # DHCP deshabilitado
      :ip => "10.0.0.2", # IP del servidor web
      :libvirt__forward_mode => "veryisolated" 
  end

  # Servidor de Base de Datos
  config.vm.define :bd do |db| # Cambiado a "bd"
    db.vm.box = "debian/bookworm64" # Imagen de Debian 12 (bookworm)
    db.vm.hostname = "bd" # Nombre del host simplificado
    db.vm.synced_folder ".", "/vagrant", disabled: true

    # Red pública (con salida a Internet)
    db.vm.network :public_network,
      :dev => "br0", # Interfaz de puente
      :mode => "bridge",
      :type => "bridge" 

    # Red privada (muy aislada)
    db.vm.network :private_network,
      :libvirt__network_name => "red_intra", # Nombre de la red privada
      :libvirt__dhcp_enabled => false, # DHCP deshabilitado
      :ip => "10.0.0.3", # IP del servidor de base de datos
      :libvirt__forward_mode => "veryisolated" 
  end
end

```

Una vez que lo tenemos ya todo definido, obviamente se podria simplificar y se quedaria tal que asi:

```
Vagrant.configure("2") do |config|

  # Definir máquinas
  %w[web bd].each do |tipo|
    config.vm.define tipo do |maquina|
      maquina.vm.box = "debian/bookworm64" # Imagen de Debian 12
      maquina.vm.hostname = tipo # Nombre de la máquina

      # Carpeta sincronizada (deshabilitada)
      maquina.vm.synced_folder ".", "/vagrant", disabled: true

      # Red pública (con salida a Internet)
      maquina.vm.network :public_network, dev: "br0", mode: "bridge"

      # Red privada (muy aislada)
      maquina.vm.network :private_network,
        ip: tipo == "web" ? "10.0.0.2" : "10.0.0.3", # IP según el tipo
        libvirt__network_name: "red_intra", # Nombre de la red privada
        libvirt__dhcp_enabled: false, # DHCP deshabilitado
        libvirt__forward_mode: "veryisolated" # Modo de reenvío
    end
  end
end

```

una vez contruido ya nuestro escenario para esta parte lo que tendremos que realizar es la conexión por ssh, desde el directorio donde tenemos el Vagrantfile, en mi caso es el siguiente directorio:

```
madandy@toyota-hilux:~/Documentos/SegundoASIR/github/iweb/Migración$
```
Una vez estando en el directorio lo que haremos será entrar por ssh a través de los comando siguientes que pongo a continuación:

```
vagrant ssh web
vagrant ssh bd 
```

A continuación muestro el como se ve desde dentro con su direccionamiento:

*Vagrant web*

```
vagrant@web:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:a3:d1:91 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 192.168.121.144/24 brd 192.168.121.255 scope global dynamic eth0
       valid_lft 2632sec preferred_lft 2632sec
    inet6 fe80::5054:ff:fea3:d191/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:7d:48:92 brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet 172.22.6.147/16 brd 172.22.255.255 scope global dynamic eth1
       valid_lft 82721sec preferred_lft 82721sec
    inet6 fe80::5054:ff:fe7d:4892/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:5c:a6:fe brd ff:ff:ff:ff:ff:ff
    altname enp0s7
    altname ens7
    inet 10.0.0.2/24 brd 10.0.0.255 scope global eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe5c:a6fe/64 scope link 
       valid_lft forever preferred_lft forever

```

*Vagrant bd*

```
vagrant@bd:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:cf:a0:23 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    altname ens5
    inet 192.168.121.15/24 brd 192.168.121.255 scope global dynamic eth0
       valid_lft 2603sec preferred_lft 2603sec
    inet6 fe80::5054:ff:fecf:a023/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:70:9a:1a brd ff:ff:ff:ff:ff:ff
    altname enp0s6
    altname ens6
    inet 172.22.6.148/16 brd 172.22.255.255 scope global dynamic eth1
       valid_lft 82697sec preferred_lft 82697sec
    inet6 fe80::5054:ff:fe70:9a1a/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:08:56:09 brd ff:ff:ff:ff:ff:ff
    altname enp0s7
    altname ens7
    inet 10.0.0.3/24 brd 10.0.0.255 scope global eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe08:5609/64 scope link 
       valid_lft forever preferred_lft forever
vagrant@bd:~$ 

```