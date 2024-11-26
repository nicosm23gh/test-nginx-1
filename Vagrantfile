Vagrant.configure("2") do |config| 
  config.vm.define "nginx-ftps-server" do |nginx_server|
    nginx_server.vm.box = "debian/bookworm64"

    # Red privada para la VM
    nginx_server.vm.network "private_network", ip: "192.168.56.101"

    # Configuración de la memoria
    nginx_server.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end

    # Configuración del nombre del servidor
    nginx_server.vm.hostname = "nginx-ftps-server"

    # Carpetas sincronizadas
    nginx_server.vm.synced_folder "site1", "/var/www/site1", create: true
    nginx_server.vm.synced_folder "site2", "/var/www/site2", create: true
    nginx_server.vm.synced_folder "./perfect-html-education/html", "/var/www/perfect-html-education/html", create: true

    # Aprovisionamiento de la máquina
    nginx_server.vm.provision "shell", inline: <<-SHELL
      # Actualizar los repositorios e instalar dependencias
      sudo apt update
      sudo apt install -y nginx vsftpd openssl git

      # Configuración de Nginx (solo para site1)
      sudo mkdir -p /var/www/site1/html
      sudo git clone https://github.com/cloudacademy/static-website-example /var/www/site1/html
      sudo chown -R www-data:www-data /var/www/site1/html
      sudo chmod -R 755 /var/www/site1
      echo 'server { 
              listen 80; 
              listen [::]:80; 
              root /var/www/site1/html; 
              index index.html index.htm index.nginx-debian.html; 
              server_name site1; 
              location / { 
                try_files $uri $uri/ =404; 
              }
            }' | sudo tee /etc/nginx/sites-available/site1
      sudo ln -sf /etc/nginx/sites-available/site1 /etc/nginx/sites-enabled/
      echo "192.168.56.101    site1" | sudo tee -a /etc/hosts
      sudo systemctl restart nginx

      # Configuración de Nginx para site2 (servidor FTP)
      echo 'server { 
              listen 80; 
              listen [::]:80; 
              root /var/www/site2/html; 
              index index.html index.htm index.nginx-debian.html; 
              server_name site2; 
              location / { 
                try_files $uri $uri/ =404; 
              } 
            }' | sudo tee /etc/nginx/sites-available/site2
      sudo ln -sf /etc/nginx/sites-available/site2 /etc/nginx/sites-enabled/

      # Configuración de FTP (solo para site2)
      sudo mkdir -p /var/www/site2/html
      sudo chown -R "ftpuser:ftpuser" /var/www/site2/html
      sudo chmod -R 755 /var/www/site2

      # Crear un usuario FTP para site2
      sudo useradd -m ftpuser -s /bin/bash
      echo "ftpuser:password" | sudo chpasswd

      # Crear certificados SSL para FTP
      sudo mkdir -p /etc/ssl/private
      sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt 
      
      # Configurar vsftpd para site2
      sudo cp -v /vagrant/vsftpd.conf /etc/vsftpd.conf
      sudo systemctl restart vsftpd
      sudo systemctl enable vsftpd

      # Crear logs para vsftpd
      sudo touch /var/log/vsftpd.log
      sudo chmod 640 /var/log/vsftpd.log
      sudo chown root:adm /var/log/vsftpd.log

      # Configuración de Nginx para perfect-html-education
      sudo mkdir -p /var/www/perfect-html-education/html
      sudo chown -R www-data:www-data /var/www/perfect-html-education/html
      sudo chmod -R 755 /var/www/perfect-html-education
      sudo nano /etc/nginx/sites-available/perfect-html-education
      echo 'server { 
              listen 80; 
              listen [::]:80; 
              root /var/www/perfect-html-education/html; 
              index index.html index.htm index.nginx-debian.html; 
              server_name perfect-html-education; 
              location / { 
                auth_basic "Área restringida"; 
                auth_basic_user_file /etc/nginx/.htpasswd; 
                try_files $uri $uri/ =404; 
                } 
            }' | sudo tee /etc/nginx/sites-available/perfect-html-education
      sudo ln -sf /etc/nginx/sites-available/perfect-html-education /etc/nginx/sites-enabled/
      echo "192.168.56.101    perfect-html-education" | sudo tee -a /etc/hosts

      sudo systemctl restart nginx
    SHELL
  end
end