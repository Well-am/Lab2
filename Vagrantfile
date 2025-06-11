# -*- mode: ruby -*-
# vi: set ft=ruby :
#Установка переменной окружения, задающей российское зеркало для box-файлов
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
Vagrant.configure("2") do |config|
# Создание виртуальной машины веб-сервера
config.vm.define "webserver" do |web|
 web.vm.box = "bento/ubuntu-24.04"
 web.vm.network "private_network", ip: "192.168.50.10"
# Установка Apache и необходимых модулей
 web.vm.provision "shell", inline: <<-SHELL
sudo apt update
sudo apt install -y apache2 mysql-client php libapache2-mod-php php-mysql

sudo cat > /var/www/html/test_db.php << EOF
<?php
$host = "192.168.50.11";
$user = "vagrant_test";
$pass = "Tusur123";
$db = "testdb";
$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
 die("Connection failed: " . $conn->connect_error);
}
echo "Connected to MySQL successfully!";
?>
EOF

sudo systemctl enable apache2
sudo systemctl restart apache2
 SHELL
end
# Создание виртуальной машины базы данных
 config.vm.define "dbserver" do |db|
db.vm.box = "bento/ubuntu-24.04"
 db.vm.network "private_network", ip: "192.168.50.11"
# Установка MySQL
 db.vm.provision "shell", inline: <<-SHELL
sudo apt update
sudo apt install -y mysql-server
sudo systemctl enable mysql

sudo cat > /tmp/init.sql << EOF
CREATE DATABASE testdb;
USE testdb;
CREATE TABLE users (id INT, name VARCHAR(20));
INSERT INTO users VALUES (1, 'vagrant_test');
CREATE USER 'vagrant_test'@'192.168.50.10' IDENTIFIED BY 'Tusur123';
GRANT ALL PRIVILEGES ON testdb.* TO 'vagrant_test'@'192.168.50.10';
FLUSH PRIVILEGES;
EOF

sudo mysql -h "localhost" -u "root" < /tmp/init.sql
sudo rm /tmp/init.sql
sudo sed 's/^bind-address[^ ]*/bind-address\ =\ 192\.168\.50\.11/' /etc/mysql/mysql.conf.d/mysqld.cnf
sudo systemctl restart mysql
 SHELL
end
# Создание виртуальной машины балансировщика нагрузки
 config.vm.define "loadbalancer" do |lb|
 lb.vm.box = "bento/ubuntu-24.04"
 lb.vm.network "public_network", bridge: "Realtek PCIe GbE Family Controller"
# Установка HAProxy
 lb.vm.provision "shell", inline: <<-SHELL
sudo apt update
sudo apt install -y haproxy
sudo systemctl enable haproxy

sudo cat >> /etc/haproxy/haproxy.cfg << EOF

frontend http_front
 bind *:80
 default_backend http_back
backend http_back
 balance roundrobin
 server webserver1 192.168.50.10:80 check inter 5000 rise 2 fall 3
EOF
sudo systemctl restart haproxy
 SHELL
end
end
