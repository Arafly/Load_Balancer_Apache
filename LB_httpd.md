In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

* Archi image

### Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu intance, and ensure that users can be served by Web servers through the Load Balancer.

To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.

## Configure Apache As A Load Balancer
- Create an Ubuntu Server 20.04 EC2
- Open TCP port 80 on *Project-8-apache-lb* by creating an inbound rule.
- Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

```
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
Make sure apache2 is up and running

sudo systemctl status apache2
```

### Configure the Load Balancer

```
<VirtualHost *:80>
  <Proxy "balancer://mycluster">
    BalancerMember http://10.154.0.7:80 loadfactor=5 timeout=1
    BalancerMember http://10.154.0.12:80 loadfactor=5 timeout=1
    ProxySet lbmethod=bytraffic
    # ProxySet lbmethod=byrequests
  </Proxy>

  ProxyPreserveHost On
  ProxyPass / balancer://mycluster/
  ProxyPassReverse / balancer://mycluster/

  #ServerName www.example.com
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html

  #LogLevel info ssl:warn
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  # For most configuration files from conf-available/, which are
  # enabled or disabled at a global level, it is possible to
  # include a line for only one particular virtual host. For example
  the
  # following line enables the CGI configuration for this host only
  # after it has been globally disabled with "a2disconf".
  #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

Restart the Apache server

`sudo systemctl restart apache2`

- Verify that your configuration works - try to access your LB’s public IP address or Public DNS name from your browser:
  
<<http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php>>

*LB image

- Unmount the /var/log/httpd/ from your Web Servers to the NFS server - unmount them and make sure that each Web Server 

`sudo umount -t nfs rw,nosuid ip:/mnt/apps /v
ar/www`