Vagrant.configure("2") do |config|
  config.vm.define "nginx-ftps-server" do |nginx_server|
    nginx_server.vm.box = "debian/bookworm64"

    nginx_server.vm.network "private_network", ip: "192.168.56.101"

    # Configuración de la memoria
    nginx_server.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end

    # Configuración del nombre del servidor
    nginx_server.vm.hostname = "nginx-ftps-server"

    # Carpetas sincronizadas entre el host y la VM
    nginx_server.vm.synced_folder "site1", "/var/www/site1", create: true
    nginx_server.vm.synced_folder "site2", "/var/www/site2", create: true

    nginx_server.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt install -y nginx vsftpd openssl git

      sudo systemctl enable nginx
      sudo systemctl start nginx

      # Crear directorios para los sitios web
      sudo mkdir -p /var/www/site1/html
      sudo mkdir -p /var/www/site2/html

      # Clonación de Git para site1
      sudo git clone https://github.com/cloudacademy/static-website-example /var/www/site1/html
      sudo chown -R www-data:www-data /var/www/site1/html
      sudo chmod -R 755 /var/www/site1

      # Configuración de Nginx para site1 y site2
      echo 'server { listen 80; listen [::]:80; root /var/www/site1/html; index index.html index.htm index.nginx-debian.html; server_name site1; location / { try_files $uri $uri/ =404; } }' | sudo tee /etc/nginx/sites-available/site1
      echo 'server { listen 80; listen [::]:80; root /var/www/site2/html; index index.html index.htm index.nginx-debian.html; server_name site2; location / { try_files $uri $uri/ =404; } }' | sudo tee /etc/nginx/sites-available/site2

      sudo ln -sf /etc/nginx/sites-available/site1 /etc/nginx/sites-enabled/
      sudo ln -sf /etc/nginx/sites-available/site2 /etc/nginx/sites-enabled/
      echo "192.168.56.101    site1" | sudo tee -a /etc/hosts
      echo "192.168.56.101    site2" | sudo tee -a /etc/hosts
      sudo systemctl restart nginx

      # Generación de los certificados SSL para FTPES
      sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt

      # Configuración de FTPES (FTP explícito sobre SSL)
      echo "rsa_cert_file=/etc/ssl/certs/vsftpd.crt" | sudo tee -a /etc/vsftpd.conf
      echo "rsa_private_key_file=/etc/ssl/private/vsftpd.key" | sudo tee -a /etc/vsftpd.conf
      echo "ssl_enable=YES" | sudo tee -a /etc/vsftpd.conf
      echo "allow_anon_ssl=NO" | sudo tee -a /etc/vsftpd.conf
      echo "force_local_data_ssl=YES" | sudo tee -a /etc/vsftpd.conf
      echo "force_local_logins_ssl=YES" | sudo tee -a /etc/vsftpd.conf
      echo "ssl_tlsv1=YES" | sudo tee -a /etc/vsftpd.conf
      echo "ssl_ciphers=HIGH" | sudo tee -a /etc/vsftpd.conf
      echo "local_enable=YES" | sudo tee -a /etc/vsftpd.conf
      echo "write_enable=YES" | sudo tee -a /etc/vsftpd.conf
      echo "chroot_local_user=YES" | sudo tee -a /etc/vsftpd.conf
      echo "allow_writeable_chroot=YES" | sudo tee -a /etc/vsftpd.conf

      # Reiniciar los servicios
      sudo systemctl restart nginx
      sudo systemctl restart vsftpd

      # Crear directorio para los archivos FTP en site2
      sudo mkdir -p /home/ftpuser
      sudo chown root:root /home/ftpuser
      sudo chmod 755 /home/ftpuser

      # Crear un usuario FTP para site2
      sudo useradd -m ftpuser -s /bin/bash
      echo "ftpuser:password" | sudo chpasswd
      sudo mkdir -p /home/ftpuser/uploads
      sudo chown ftpuser:ftpuser /home/ftpuser/uploads
      sudo chmod 755 /home/ftpuser/uploads
    SHELL
  end
end