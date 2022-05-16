**ASP .NET linux install guide**
==============

1. [.NET install on Ubuntu](https://github.com/BadMandarin/Guides/blob/main/ASPLinuxInstal.md#net-install-on-ubuntu)
2. [Apache configuration](https://github.com/BadMandarin/Guides/blob/main/ASPLinuxInstal.md#apache)
3. [Service creating](https://github.com/BadMandarin/Guides/blob/main/ASPLinuxInstal.md#service-creating)

**.NET Install on Ubuntu**
==============

First of all we need to instal .NET.
It is important to choose correct versions of Ubuntu (20.04 in example) and .Net installs (6.0 in example).
[Source](https://docs.microsoft.com/ru-ru/dotnet/core/install/linux-ubuntu#2004)

#### Packages

```sh
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
```

#### SDK

```sh
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0
```
#### Runtime

```sh
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y aspnetcore-runtime-6.0
```

**APACHE**
==============

Next we need to install apache and setup it. 
After apache install we enabling mods for correct site work. 
Also, create config file with site properties (domain, local address, logs names).
If we have blazor server we need to setup additional proxies for socket connection.

```sh
sudo apt-get install apache2
```
#### UTIL CMDS
```sh
sudo service apache2 restart
sudo apachectl configtest
```
#### APACHE MODS
```sh
sudo a2enmod proxy proxy_http proxy_html proxy_wstunnel
sudo a2enmod rewrite
```
#### SITE CONFIG FILE
```sh
sudo nano /etc/apache2/conf-available/sitename.conf
```
Do not forget to enable config file with commands below. Also, restart apache after that.
#### Enabling/Disabling
```sh
a2ensite sitename
```
```sh
a2dissite sitename
```

#### CONFIG CONTENT
```
<VirtualHost *:80>
ServerName www.DOMAIN.COM

ProxyPreserveHost On
ProxyPass / http://localhost:5000/
ProxyPassReverse / http://localhost:5000/

RewriteEngine on 
RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC] 
RewriteCond %{HTTP:CONNECTION} Upgrade$ [NC] 
RewriteRule /(.*) ws://127.0.0.1:5000/$1 [P]

ErrorLog /var/log/apache2/netcore-error.log
CustomLog /var/log/apache2/netcore-access.log common
</VirtualHost>
```
#### BLAZOR SERVER CONFIG

[Source](https://docs.microsoft.com/ru-ru/aspnet/core/blazor/host-and-deploy/server?view=aspnetcore-6.0)

```
<VirtualHost *:80>
ServerName www.DOMAIN.COM

ProxyRequests       On
ProxyPreserveHost   On
ProxyPassMatch      ^/_blazor/(.*) http://localhost:5000/_blazor/$1
ProxyPass           /_blazor ws://localhost:5000/_blazor
ProxyPass           / http://localhost:5000/
ProxyPassReverse    / http://localhost:5000/

RewriteEngine on 
RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC] 
RewriteCond %{HTTP:CONNECTION} Upgrade$ [NC] 
RewriteRule /(.*) ws://127.0.0.1:5000/$1 [P]

ErrorLog /var/log/apache2/netcore-error.log
CustomLog /var/log/apache2/netcore-access.log common
</VirtualHost>
```
**SERVICE CREATING**
==============

Final step is creating service that will work in background and start, restart on crash our site.

```
sudo nano /etc/systemd/system/servicename.service
```
#### SERVICE FILE CONTENT
```
[Unit]
Description=ASP .NET Web Application
[Service]
WorkingDirectory=/var/netcore
ExecStart=/usr/bin/dotnet /var/netcore/MainDllName.dll
Restart=always
RestartSec=10
SyslogIdentifier=netcore-demo
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
[Install]
WantedBy=multi-user.target
```
#### Service Cmds
```
sudo systemctl enable servicename.service
sudo systemctl start servicename.service
sudo systemctl restart servicename.service
```


