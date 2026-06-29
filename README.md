
<img width="1237" height="567" alt="image" src="https://github.com/user-attachments/assets/17cac388-328e-415c-a854-3abf8181a49d" />



```
ubuntu@ubuntu-1:~$ sudo sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1

   36  sudo sysctl -w net.ipv4.ip_forward=1
   37  sudo apt update
   38  sudo apt install nginx -y
   39  sudo apt install php-fpm -y

ubuntu@ubuntu-1:~$ cd /var/www
ubuntu@ubuntu-1:/var/www$ sudo mkdir testweb
ubuntu@ubuntu-1:/var/www$ sudo chown -R $USER:$USER /var/www/testweb

ubuntu@ubuntu-1:/var/www$ sudo gedit /etc/nginx/sites-available/testweb




ubuntu@ubuntu-1:/etc/nginx$ sudo gedit /etc/nginx/sites-available/testweb

server {
	listen 80;
	server_name testweb www.testweb;
	root /var/www/testweb;
	
	index index.html index.htm index.php;
	
	location / {
		try_files $uri $uri/ =404;
	}
	
	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
	
	}
}




ubuntu@ubuntu-1:/etc/nginx$ sudo ln -s /etc/nginx/sites-available/testweb /etc/nginx/sites-enabled/
ubuntu@ubuntu-1:/etc/nginx$ ll sites-enabled/
total 8
drwxr-xr-x 2 root root 4096  六  13 16:18 ./
drwxr-xr-x 8 root root 4096  六  13 15:50 ../
lrwxrwxrwx 1 root root   34  六  13 15:50 default -> /etc/nginx/sites-available/default
lrwxrwxrwx 1 root root   34  六  13 16:18 testweb -> /etc/nginx/sites-available/testweb

ubuntu@ubuntu-1:/etc/nginx/sites-enabled$ sudo unlink /etc/nginx/sites-enabled/default

ubuntu@ubuntu-1:/etc/nginx/sites-enabled$ sudo nginx -t

[sudo] password for ubuntu: 
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

ubuntu@ubuntu-1:/etc/nginx/sites-enabled$ sudo systemctl reload nginx

ubuntu@ubuntu-1:/var/www/html$ cat index.html 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
h1>Welcome to nginx!</h1>
<p>Hello World!</p>
</html>


```
<img width="832" height="366" alt="image" src="https://github.com/user-attachments/assets/f2c9cd33-7432-440e-b1fd-be68e5bfac39" />

```
ubuntu@ubuntu-1:/var/www/testweb$ cat info.php 
<?php
phpinfo();

?>

```
<img width="1425" height="666" alt="image" src="https://github.com/user-attachments/assets/e314db0d-7868-4325-a4ca-bda59af994d7" />



<img width="478" height="302" alt="image" src="https://github.com/user-attachments/assets/0abbe246-ea71-4549-a7fb-fa2d8888e4c1" />







==特別注意==

sudo sysctl -w net.ipv4.ip_forward=1
或
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o ens160 -j MASQUERADE
或
sudo iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE

```
ubuntu@ubuntu-1:/var/www/testweb$ ip route
default via 172.25.250.2 dev ens160 proto dhcp metric 100 
169.254.0.0/16 dev ens160 scope link metric 1000 
172.25.250.0/24 dev ens160 proto kernel scope link src 172.25.250.129 metric 100 
192.168.0.0/24 dev ens192 proto kernel scope link src 192.168.0.254 metric 101 
ubuntu@ubuntu-1:/var/www/testweb$ sudo ip route add 192.168.100.0/24 via 192.168.0.1
[sudo] password for ubuntu: 
ubuntu@ubuntu-1:/var/www/testweb$ sudo ip route add 192.168.200.0/24 via 192.168.0.1
ubuntu@ubuntu-1:/var/www/testweb$ sudo ip route add 220.196.200.0/24 via 192.168.0.1
ubuntu@ubuntu-1:/var/www/testweb$ ip route
default via 172.25.250.2 dev ens160 proto dhcp metric 100 
169.254.0.0/16 dev ens160 scope link metric 1000 
172.25.250.0/24 dev ens160 proto kernel scope link src 172.25.250.129 metric 100 
192.168.0.0/24 dev ens192 proto kernel scope link src 192.168.0.254 metric 101 
192.168.100.0/24 via 192.168.0.1 dev ens192 
192.168.200.0/24 via 192.168.0.1 dev ens192 
220.196.200.0/24 via 192.168.0.1 dev ens192 
```

==ACL==
<img width="846" height="262" alt="image" src="https://github.com/user-attachments/assets/bf58297c-9fa5-4ecc-9ef1-2c6645d9a72b" />


## Web-term1 (192.168.100.3)可連外網
```
ubuntu@ubuntu-1:/var/www/testweb$ sudo iptables -t nat -A POSTROUTING -s 192.168.100.3 -o ens160 -j MASQUERADE
[sudo] password for ubuntu: 
```
## PC2 (192.168.100.2) 不能連外網, 不可連Nginx
```
ubuntu@ubuntu-1:/var/www/testweb$ sudo iptables -A FORWARD -s 192.168.100.2 -o ens160 -j DROP
ubuntu@ubuntu-1:/var/www/testweb$ sudo iptables -A FORWARD -s 192.168.100.2 -d 192.168.0.254 -p tcp --dport 80 -j DROP
```
## PC3 (220.196.200.1) 不能連外網, 但可連Nginx
```
ubuntu@ubuntu-1:/var/www/testweb$ sudo iptables -A FORWARD -s 220.196.200.1 -o ens160 -j DROP
```
ACL table 如下:
```
ubuntu@ubuntu-1:/var/www/testweb$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
DROP       all  --  192.168.100.2        anywhere            
DROP       tcp  --  192.168.100.2        ubuntu-1             tcp dpt:http
DROP       all  --  220.196.200.1        anywhere            

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```

==R1 route setting==

<img width="992" height="350" alt="image" src="https://github.com/user-attachments/assets/46cda8df-81d1-430e-9fe9-c9f8edf72e49" />




DHCP Server
```
ubuntu@ubuntu-1:/etc/dhcp$ cat dhcpd.conf
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;
authoritative;
option domain-name-servers 192.168.100.1;
subnet 192.168.100.0 netmask 255.255.255.0 {
           range 192.168.100.10 192.168.100.50;
           option routers 192.168.100.254;
           option subnet-mask 255.255.255.0;
           option domain-name-servers 192.168.100.1, 8.8.8.8;
}
```

<img width="992" height="350" alt="image" src="https://github.com/user-attachments/assets/9a12021a-d524-42fe-a0c1-584c343cdeae" />

