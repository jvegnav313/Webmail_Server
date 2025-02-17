Vagrant.configure("2") do |config|
  
  config.vm.box = "debian/bullseye64"

  config.vm.define "mail" do |mail|
      
      mail.vm.hostname = "mail"
      mail.vm.network "private_network", ip: "192.168.56.10"
      mail.vm.provision "shell", inline: <<-SHELL

      ### Agregar repositorios adicionales en caso de que no funcionen los que trae por defecto ###

      echo "deb http://deb.debian.org/debian bullseye main contrib non-free" > /etc/apt/sources.list
      echo "deb http://security.debian.org/debian-security bullseye-security main contrib non-free" >> /etc/apt/sources.list
      echo "deb http://deb.debian.org/debian bullseye-updates main contrib non-free" >> /etc/apt/sources.list
      echo "deb http://deb.debian.org/debian bullseye-backports main contrib non-free" >> /etc/apt/sources.list

      ### Actualizar repositorios y paquetes ###

      apt update && apt upgrade -y

      ### Crear los usuarios ###

      useradd -m -s /bin/bash -p $(openssl passwd -1 usuario1) usuario1
      useradd -m -s /bin/bash -p $(openssl passwd -1 usuario2) usuario2

      ### Instalar Servicio DNS (bind9) ###

      apt install bind9 -y

      ### Copiar archivos bind9 ###

      cp -v /vagrant/config/bind9/named /etc/default/
      cp -v /vagrant/config/bind9/mail.izv.test.dns /etc/bind
      cp -v /vagrant/config/bind9/mail.izv.test.rev /etc/bind
      cp -v /vagrant/config/bind9/named.conf.local /var/lib/bind
      cp -v /vagrant/config/bind9/named.conf.options /var/lib/bind

      ### Configurar hosts y resolv.conf ###

      cp -v /vagrant/config/etc/hostname /etc/hostname
      cp -v /vagrant/config/etc/hosts /etc/hosts
      cp -v /vagrant/config/etc/resolv.conf /etc/resolv.conf

      ### Instalar Servicio Web (apache2) ###

      apt install apache2 -y
      apt install php libapache2-mod-php -y
      
      ### Configurar apache2 y habilitar sitios ###

      cp -v /vagrant/config/apache2/apache2.conf /etc/apache2/apache2.conf
      cp -v /vagrant/config/apache2/mail.conf /etc/apache2/sites-available/mail.conf
      a2ensite mail.conf
      a2dissite 000-default.conf
      systemctl restart apache2

      ### Instalar Dovecot ###
 
      apt install dovecot-core dovecot-imapd dovecot-pop3d -y
      
      ### Instalar servicio de mail (SquirrelMail) ###
      
      tar xvfz /vagrant/config/squirrelmail/squirrelmail.tgz -C /usr/local
      ln -s /usr/local/squirrelmail-webmail-1.4.22 /var/www/mail
      sudo mkdir -p /var/local/squirrelmail/data
      chown www-data:www-data /var/local/squirrelmail/data
      mkdir -p /var/local/squirrelmail/attach
      chown www-data:www-data /var/local/squirrelmail/attach
      
      cp -v /vagrant/config/squirrelmail/config.php /usr/local/squirrelmail-webmail-1.4.22/config/config.php
      cp -r /usr/local/squirrelmail-webmail-1.4.22/ /var/www/mail/

      ### Establecer idioma ###
      
      locale-gen es_ES
     
      ### Configurar Postfix ###

      cp -v /vagrant/config/postfix/main.cf /etc/postfix/

      ### Instalar postfix con debconf ###

      apt install debconf -y 
      debconf-set-selections <<< "postfix postfix/mailname string izv.test"
      debconf-set-selections <<< "postfix postfix/main_mailer_type string 'internet site'"
      apt install postfix -y
      systemctl restart postfix
      
      SHELL

  end

end