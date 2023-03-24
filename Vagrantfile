# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "debian/bullseye64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  
  #Atualizando pacotes
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt update -y
  SHELL

  #Instalando Apache
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt install -y apache2
  sudo systemctl enable --now apache2
  SHELL

  #Instalando Mariadb
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt install -y mariadb-server
  sudo systemctl enable --now mariadb
  SHELL
  
  #Instalando PHP
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt install -y php php-{bz2,mysqli,bcmath,mbstring,ldap,net-socket,pgsql,curl,gd,intl,common,mbstring,xml}
  SHELL
  
  #Adicionando Repositorio Zabbix
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt install -y wget
  wget https://repo.zabbix.com/zabbix/5.4/debian/pool/main/z/zabbix-release/zabbix-release_5.4-1+debian11_all.deb
  sudo dpkg -i zabbix-release_5.4-1+debian11_all.deb
  sudo apt update -y
  SHELL
  
  #Instalando Zabbix
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
  SHELL
  
  #Configurando Mariadb
  config.vm.provision "shell", inline: <<-SHELL
  sudo /usr/bin/mysqladmin -u root password 'zabbix'
  mysql -u root -ppassword -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION; FLUSH PRIVILEGES;"
  mysql -u root -e "CREATE DATABASE zabbix character set utf8 collate utf8_bin;"
  mysql -u root -e "grant all privileges on zabbix.* to zabbix@'localhost' identified by 'zabbix@';"
  mysql -u root -e "grant all privileges on zabbix.* to zabbix@'%' identified by 'zabbix@'; flush privileges;"
  SHELL
  
  #Importando esquema e dados inicias do Zabbix para a base de dados que criamos
  config.vm.provision "shell", inline: <<-SHELL
  zcat /usr/share/doc/zabbix-sql-scripts/mysql/create.sql.gz | sudo -u root mysql zabbix
  SHELL

  #Configurando Zabbix Server
  config.vm.provision "shell", inline: <<-SHELL
  sudo sed -i "s/# DBHost=localhost/DBHost=localhost/" /etc/zabbix/zabbix_server.conf
  sudo sed -i "s/# DBPassword=/DBPassword=zabbix/" /etc/zabbix/zabbix_server.conf
  sudo sed -i "s/DBUser=zabbix/DBUser=root/" /etc/zabbix/zabbix_server.conf

  sudo systemctl restart zabbix-server zabbix-agent apache2
  sudo systemctl enable zabbix-server zabbix-agent
  SHELL

  #Configurando Locais para alterar idioma do front do Zabbix
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt install -y locales-all
  SHELL

  #Verificando status dos serviços
  #config.vm.provision "shell", inline: <<-SHELL
  #systemctl status mariadb
  #systemctl status apache2
  #SHELL

  #login e senha padrão zabbix front-end (user: Admin | Password: zabbix)
end
