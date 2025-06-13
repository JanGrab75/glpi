# glpi
GLPI

Działający plik glpi.conf

Przekierowanie z HTTP na HTTPS
```bash
<VirtualHost *:80>
    ServerName 10.10.50.69
    Redirect permanent / https://10.10.50.69/glpi
</VirtualHost>
```bash
Obsługa HTTPS
```bash
<VirtualHost *:443>
    ServerName 10.10.50.69
    DocumentRoot /var/www/html/glpi

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/glpi.crt
    SSLCertificateKeyFile /etc/ssl/private/glpi.key

    <Directory /var/www/html/glpi/>
        Options FollowSymLinks
        AllowOverride All
        Require all granted

        RewriteEngine On
        RewriteCond %{HTTP:Authorization} ^(.+)$
        RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/glpi_error.log
    CustomLog ${APACHE_LOG_DIR}/glpi_access.log combined
</VirtualHost>
```


```bash
cd /var/www/html/glpi
```
```bash
php bin/console doctrine:migrations:migrate
```
```bash
cd /var/www/html/glpi
```
```bash
sudo php bin/console doctrine:migrations:migrate
```
```bash
sudo nano /etc/apache2/sites-available/glpi.conf
```
```bash
  sudo nano /etc/apache2/sites-available/glpi.conf
```

  <VirtualHost *:80>
    ServerName glpi.localhost
    DocumentRoot /var/www/glpi/public

    <Directory /var/www/glpi/public>
        Require all granted
        RewriteEngine On
        # Przekieruj wszystkie żądania do index.php, jeśli plik nie istnieje
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>

    # Zabezpiecz katalogi config i files przed dostępem z internetu
    <Directory /var/www/glpi/config>
        Require all denied
    </Directory>

    <Directory /var/www/glpi/files>
        Require all denied
    </Directory>
</VirtualHost>

```bash
sudo systemctl reload apache2
```
