# Servidor Web con Nginx

Este repositorio contiene una práctica para la instalación y configuración de un servidor web Nginx en Debian. Además, se incluye la configuración para conectar mediante FTPES y realizar transferencias seguras de archivos al servidor.


## 1. Instalación del Servidor Web Nginx

1. Actualizar los repositorios e instalar Nginx:
```bash
   sudo apt update
   sudo apt install nginx
```

### 1. Instalación del Servidor Web Nginx
```bash
    systemctl status nginx
```

## 2. Configuración del Sitio Web

    Crear la estructura de carpetas para el sitio web:

```bash
    sudo mkdir -p /var/www/nginxS/html
```
Clonar el repositorio de ejemplo en la carpeta del sitio web:

```bash
    git clone https://github.com/cloudacademy/static-website-example /var/www/nginxS/html
 ```
Asignar permisos y propietario a los archivos del sitio web:
```bash

    sudo chown -R www-data:www-data /var/www/nginxS/html
    sudo chmod -R 755 /var/www/nginxS
```

Configurar el bloque de servidor para el sitio en Nginx:

```bash
    sudo nano /etc/nginx/sites-available/nginxS
 ```
Consiguraciion de dominio:
 ```bash
    server {
        listen 80;
        listen [::]:80;
        root /var/www/nginxS/html/static-website-example;
        index index.html index.htm index.nginx-debian.html;
        server_name nginxS;
        location / {
            try_files $uri $uri/ =404;
        }
    }

 ```
Crear un enlace simbólico para habilitar el sitio:

 ```bash
    sudo ln -s /etc/nginx/sites-available/nginxS /etc/nginx/sites-enabled/
 ```
Reiniciar Nginx para aplicar los cambios:

```bash
    sudo systemctl restart nginx
```

Hay que tener en cuenta que para que no nos de error nuestro navegador con el servidor de nginx, debemos de ir a:
```bash
    C:\Windows\System32\drivers\etc\hosts 
 ```
y añadir
```bash
    192.168.1.9 nginxS
```
Una vez hecho ya podemos comprobar como se esta acediendo correctamente a los archivos logs en: 
```bash
    /var/log/nginx/access.log
    /var/log/nginx/error.log
 ```

## 3. Configuración de FTP Seguro (FTPS)

Para una transferencia segura de archivos, configuraremos el servidor con FTPS mediante vsftpd.

Instalar el servidor vsftpd:

```bash
    sudo apt-get install vsftpd
```
Crear los certificados SSL para habilitar FTPS:
```bash
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt
```
Si en nuestro Vagrantfile añadiremos este comando para que se den por si solas las respuestas del certificado 
```bash
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt -subj "/C=ES/ST=./L=./O=./OU=./CN=nginxS/emailAddress=."
```

Configurar vsftpd para utilizar FTPS:

```bash
    sudo nano /etc/vsftpd.conf
```
Actualizar la configuración del vsftpd.conf, en el borraremos estas lineas:

```bash
    rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
    rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
    ssl_enable=NO
```

y las sustituiremos por las siguientes:

 ```bash
    rsa_cert_file=/etc/ssl/certs/vsftpd.crt
    rsa_private_key_file=/etc/ssl/private/vsftpd.key
    ssl_enable=YES
    allow_anon_ssl=NO
    force_local_data_ssl=YES
    force_local_logins_ssl=YES
    ssl_tlsv1=YES
    ssl_sslv2=NO
    ssl_sslv3=NO
    require_ssl_reuse=NO
    ssl_ciphers=HIGH

    local_root=/home/vagrant/ftp
    write_enable=YES
 ```
Tambien seria comveniente descomentar esta linea, para que WinSCP no nos de problemas: 
 ```bash
  write_enable=YES
```
Verificar los permisos de los archivos de certificados y de clave privada:

 ```bash
    ls -l /etc/ssl/certs/vsftpd.crt /etc/ssl/private/vsftpd.key
    sudo chmod 755 /etc/ssl/private
 ```
Reiniciar vsftpd para aplicar los cambios:

 ```bash
    sudo systemctl restart vsftpd
 ```
## 4. Transferencia de Archivos mediante FTPES

Para realizar la transferencia segura de archivos, he obtado por utilizar WinSCP.

Para que me deje acceder por WinSCP, he establecido en el vagrant file una contraseña par el servidor:

 ```bash
    config.vm.provision "shell", name: "set_password", inline: <<-SHELL
    echo "vagrant:caracol" | sudo chpasswd
 ```

Una vez conectados con el servidor a traves de WinSCP, se trasnfieren los archivos necesarios en:
 ```bash
    /home/vagrant/ftp/
 ```
En mi caso, he transferido el index.html y la carpeta assets, que contiene el css, las font, y las img de la web.

Una vez transferidos los archivos, debemos de crear otro provisionamiento para la web de ejemplo que voy a utilizar.

Primero, eliminamos el enlace simbolico que haya en nuestro servidor de cualquier otra web:
 ```bash
    sudo rm /etc/nginx/sites-enabled/nginxS
  ```
Y realizamos los cambios que sean oportunos en todos los archivos para que el nuevo dominio funcione, usando un provisionamiento en el vagrantfile:
 ```bash
  config.vm.provision "shell", name: "paramoreweb", run: "never", inline: <<-SHELL
    sudo rm /etc/nginx/sites-enabled/nginxS
    sudo mkdir -p /var/www/paramoreweb/html
    sudo cp -r /home/vagrant/ftp/* /var/www/paramoreweb/html
    sudo chown -R www-data:www-data /var/www/paramoreweb/html
    sudo chmod -R 755 /var/www/paramoreweb
    cp -v /vagrant/paramoreweb /etc/nginx/sites-available/
    sudo ln -s /etc/nginx/sites-available/paramoreweb /etc/nginx/sites-enabled/
    sudo systemctl restart nginx
```

Creo el nuevo archivo paramoreweb que ira destinado a 
```bash
    /etc/nginx/sites-available/
```
El cual contiene lo siguiente:
```bash 
  server {
	listen 80;
	listen [::]:80;
	root /var/www/paramoreweb/html/static-website-example;
	index index.html index.htm index.nginx-debian.html;
	server_name paramoreweb;
	location / {
		try_files $uri $uri/ =404;
	}
}

```
Vuelvo a modificar el archivo hosts de mi Windows, para que asocie la IP a paramoreweb:

```bash
   C:\Windows\System32\drivers\etc\hosts
```
Y ya tendramos lista nuestra Web.

## 5. Cuestiones finales

¿Qué sucede si no realizo el enlace simbólico entre sites-available y sites-enabled para mi sitio en Nginx?

Sin este enlace simbólico, Nginx no puede reconocer ni cargar la configuración del sitio porque solo se fija en las configuraciones dentro de sites-enabled. Al no vincular el archivo de configuración, el sitio no estará disponible y cualquier intento de acceder a él generará un error o redirigirá a la página de configuración predeterminada. Es decir, Nginx actuará como si esa configuración específica no existiera.

¿Qué ocurre si no asigno los permisos correctos a la carpeta /var/www/nombre_web?

Sin los permisos adecuados, Nginx no podrá leer o acceder a los archivos necesarios para mostrar el contenido del sitio. Es importante que el usuario www-data, bajo el cual opera Nginx, tenga permisos de lectura en los archivos y de ejecución en los directorios. Sin estos permisos, el acceso al sitio dará errores. Para evitar problemas, los directorios suelen configurarse con permisos 755, lo cual permite tanto el acceso como la navegación entre carpetas.
