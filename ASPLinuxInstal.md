**ASP .NET linux install guide**
==============

**.NET Install on Ubuntu**
==============

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
#### Enabling/Disabling
```sh
a2ensite sitename
```
```sh
a2dissite sitename
```

#### CONTENT
```
<VirtualHost *:80>
ServerName www.DOMAIN.COM
ProxyPreserveHost On
ProxyPass / http://127.0.0.1:5000/
ProxyPassReverse / http://127.0.0.1:5000/
RewriteEngine on 
RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC] 
RewriteCond %{HTTP:CONNECTION} Upgrade$ [NC] 
RewriteRule /(.*) ws://127.0.0.1:5000/$1 [P]
ErrorLog /var/log/apache2/netcore-error.log
CustomLog /var/log/apache2/netcore-access.log common
</VirtualHost>
```
#### BLAZOR SERVER ADITIONAL VALUES
```
ProxyRequests       On
ProxyPreserveHost   On
ProxyPassMatch      ^/_blazor/(.*) http://localhost:5000/_blazor/$1
ProxyPass           /_blazor ws://localhost:5000/_blazor
ProxyPass           / http://localhost:5000/
ProxyPassReverse    / http://localhost:5000/
```
**SERVICE CREATING**
==============
```
sudo nano /etc/systemd/system/servicename.service
```
#### CONTENT
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
sudo systemctl enable kestrel-netcore.service
sudo systemctl start kestrel-netcore.service
sudo systemctl restart kestrel-netcore.service
```


