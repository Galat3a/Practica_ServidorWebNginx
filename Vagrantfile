Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.network "forwarded_port", guest: 80, host: 8081
  config.vm.network "private_network", ip: "192.168.1.9"#vip
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y nginx git
    mkdir -p /var/www/nginxServer/html
    cd /var/www/nginxServer/html
    git clone https://github.com/cloudacademy/static-website-example
    chown -R www-data:www-data /var/www/nginxServer/html
    chmod -R 755 /var/www/nginxServer
    cp -v /vagrant/nginxServer /etc/nginx/sites-available/
    ln -s /etc/nginx/sites-available/nginxServer /etc/nginx/sites-enabled
    rm /etc/nginx/sites-enabled/default
    systemctl restart nginx
  SHELL
end