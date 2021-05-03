# Load Balancer Solution With Apache

In this project we'll enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers.

![](https://github.com/Arafly/Load_Balancer_Apache/blob/master/assets/Tooling-Website-Infrastructure-wLB.png)

### Aim
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu intance, and ensure that users can be served by Web servers through the Load Balancer.

To simplify, we'd implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.

## Configure Apache as a Load Balancer
1. Spin up an Ubuntu Server 20.04 EC2
2. Open TCP port 80 on *Project-8-apache-lb* by creating an inbound rule.
3. Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

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
```

Make sure apache2 is up and running

`sudo systemctl status apache2`

### Configure the Load Balancer

1. Open up the default Apache config file and edit:

`sudo vi /etc/apache2/sites-available/000-default.conf`

```
<VirtualHost *:80>
  <Proxy "balancer://mycluster">
    BalancerMember http://Webserver1ip:80 loadfactor=5 timeout=1
    BalancerMember http://Webserver2ip:80  loadfactor=5 timeout=1
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

2. Verify that your configuration works by trying to access your LB’s public IP address or Public DNS name from your browser:
  
<http://Load-Balancer-Public-IP-Address-or-Public-DNS-Name/index.php>

![](https://github.com/Arafly/Load_Balancer_Apache/blob/master/assets/lb.PNG)

> Unmount the /var/log/httpd/ from your Web Servers to the NFS server - unmount them and make sure that each Web Server has its own log directory

`sudo umount -t nfs rw,nosuid ip:/mnt/logs /v
ar/log/httpd`

3. Open two SSH consoles for both Web Servers and run following command:

`sudo tail -f /var/log/httpd/access_log`

Try to refresh your browser page on 
<http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name/index.php>
multiple times so that both servers receive HTTP GET requests from your LB - new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers - it means that traffic will be disctributed evenly between them.

![](https://github.com/Arafly/Load_Balancer_Apache/blob/master/assets/Duo_webservers.PNG)

### Optional Step - Configure Local DNS Names Resolution
Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
The logical alternative is to configure local domain name resolution (DNS), and the easiest way in our case, is to use **/etc/hosts** file. So let us configure IP address to domain name mapping for our LB.

- Open this file on your LB server

`sudo vi /etc/hosts`

- Add two records into this file with Local IP address and arbitrary name for both of your Web Servers

```
<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```

- Now you can update your LB config file with those names instead of IP addresses.

`sudo vi /etc/apache2/sites-available/000-default.conf`

```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```

- You can now try to curl your Web Servers from the LB locally 

`curl http://Web1`

```
Output:

<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" type="text/css" href="tooling_stylesheets.css">
  <script src="script.js"></script>
  <title> PROPITIX TOOLING</title>
</head>

<body>
  <div class="header">
  </div>

  <div class="box">
    <a href="https://prometheus.infra.zooto.io/" target="_blank">
      <img src="img/prometheus.png" alt="Snow" width="400" height="150">
    </a>
  </div>
  <div class="box">
    <a href="https://k8s-metrics.infra.zooto.io/" target="_blank">
      <img src="img/kubernetes.png" alt="Snow" width="400" height="120">
    </a>
  </div>
  <div class="box">
    <a href="https://kibana.infra.zooto.io/" target="_blank">
      <img src="img/kibana.png" alt="Snow" width="400" height="10
0">
    </a>
  </div>
  </div>
  <div class="container">
    <div class="box">
      <a href="https://artifactory.infra.zooto.io/" target="_blank">
        <img src="img/jfrog.png" alt="snow" width="400" height="100
">
      </a>
    </div>
  </div>
  </div>
  </section>
</body>

</html>

````