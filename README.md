# glpi
GLPI

1. Jak w glpi zwiększyć ilość pamięci dla Zend OPcache

```bash
[opcache]
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.max_wasted_percentage=5
```

```bash
php -i | grep opcache.memory_consumption
```
```bash
sudo nano /etc/php/8.3/apache2/php.ini
```
;opcache.memory_consumption=128
opcache.memory_consumption=256
memory_limit = 512M?

```bash
sudo systemctl restart apache2
```
sudo systemctl restart php-fpm?
```bash
sudo nano /etc/apache2/sites-available/glpi.conf
```
2. Działający plik glpi.conf

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

* Aby wyłączyć w menu GLPI pozycję „Rezerwacje”, należy odebrać użytkownikom uprawnienia do tej sekcji. GLPI nie pozwala na bezpośrednie usuwanie lub ukrywanie pozycji menu bezpośrednio z poziomu konfiguracji, ale ukrycie ich jest możliwe poprzez zarządzanie uprawnieniami użytkowników lub grup.

Jak to zrobić:

Wejdź w konfigurację profili użytkowników (Administration → Profiles).

Wybierz profil (Self-Service), dla którego chcesz ukryć „Reservation”.

W sekcji uprawnień odznacz dostęp do modułu „Reservation”.

Po zapisaniu zmian użytkownicy z tym profilem nie będą widzieć tej pozycji w menu.

* Aby w GLPI umożliwić rezerwację sprzętu, takiego jak rzutnik, należy wykonać następujące kroki:

Otwórz kartę sprzętu
Przejdź do zasobu, który chcesz udostępnić do rezerwacji (np. rzutnika) w module zarządzania zasobami.

Włącz możliwość rezerwacji
Na karcie danego sprzętu znajdź zakładkę „Reservation” i kliknij „Allow reservations” – dopiero wtedy sprzęt pojawi się na liście możliwych do rezerwacji.

Zarządzaj uprawnieniami użytkowników

Uprawnienia do rezerwacji są przypisywane na poziomie profilu użytkownika. Aby użytkownicy mogli rezerwować sprzęt, ich profil musi mieć odpowiednie uprawnienia do modułu rezerwacji. Ustawisz to w sekcji „Administracja > Profile”, wybierając odpowiedni profil i zaznaczając dostęp do rezerwacji.


Rezerwacja sprzętu
Po wykonaniu powyższych kroków użytkownicy mogą zarezerwować sprzęt przez „Rezerwacje”, wybierając dostępny termin i sprzęt z listy
