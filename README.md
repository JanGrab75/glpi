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

"Timezone usage has not been activated. Run the 'php bin/console database:enable_timezones' command to activate it":

```bash
sudo mysql_tzinfo_to_sql /usr/share/zoneinfo
```
```bash
cd /var/www/html/glpi/
```
```bash
sudo php bin/console glpi:database:enable_timezones
```
```bash
mysql -u root -p
```
```bash
GRANT SELECT ON mysql.time_zone_name TO 'glpi'@'localhost';
FLUSH PRIVILEGES;
```
```bash
sudo systemctl restart mysql
```
Montowanie udział nfs z Qnap
1. Utworzenie katalogu montowania.
```bash
sudo mkdir -p /mnt/qnap_glpi_backup
```
2. Montowanie.
```bash
sudo mount -t nfs -o vers=3 10.10.50.243:/glpi_backup /mnt/qnap_glpi_backup
```
3. Dodanie montowania do fstab
```bash
nano /etc/fstab
```
```bash
10.10.50.243:/glpi_backup  /mnt/qnap_glpi_backup  nfs  defaults,nofail,noatime,intr,_netdev  0 0
```


Backup
1. Bazy
```bash
sudo mysqldump -u root -p glpidb | gzip > /mnt/gnap_glpi_backup/glpidb_$(date +"%Y_%m_%d_%H_%M").sql.gz
```
2. Plików
```bash
sudo tar -czvf /mnt/gnap_glpi_backup/glpi_files_$(date +"%Y_%m_%d_%H_%M").tar.gz /var/www/html/glpi
```
glpi-backup.sh
```bash
#!/bin/bash

# Backup plików GLPI
tar -czvf /mnt/qnap_glpi_backup/glpi_backup/glpi_files_$(date +"%Y_%m_%d_%H_%M").tar.gz /var/www/html/glpi

# Backup bazy danych
mysqldump -u root glpidb | gzip > /mnt/qnap_glpi_backup/glpi_backup/glpi_db_$(date +"%Y_%m_%d_%H_%M").sql.gz

# Usuwanie backupów starszych niż 7 dni
find /mnt/qnap_glpi_backup/ -type f -mtime +7 -delete
```
Dodanie skryptu glpi-backup.sh do cron-a.
```bash
sudo crontab -e
```
```bash
53 23 * * * sudo /mnt/qnap_glpi_backup/glpi-backup.sh
```
