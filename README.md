# glpi
GLPI

```bash
cd /var/www/html/glpi
```
```bash
php bin/console doctrine:migrations:migrate
```

`shell`, `console`, `sh`

```bash
cd /var/www/html/glpi
```

```shell
sudo php bin/console doctrine:migrations:migrate
```

```console
sudo nano /etc/apache2/sites-available/glpi.conf
```

```sh
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

sudo systemctl reload apache2
