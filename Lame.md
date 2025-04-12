# Lame - TryHackMe

**IP:** 10.10.10.3

## Escaneo Inicial
- Nmap revela:
  - VSFTPd 2.3.4
  - Samba 3.0.20-Debian

## Explotación

### VSFTPD 2.3.4
- Vulnerable a backdoor en conexiones anónimas.
- Exploit disponible: [ExploitDB #17491](https://www.exploit-db.com/exploits/17491)

### Samba 3.0.20
- Vulnerable a ejecución remota (`usermap_script`)
- Exploit en Metasploit: `usermap_script`

## Post-Explotación
- Enumeración de usuarios
- Uso de smbclient / enum4linux
