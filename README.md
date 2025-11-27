# GenieACS Parameter Configuration

## Cara Install Parameter

# MongoDB GenieACS Recovery & Parameter Restore - Tutorial Step by Step

1. **Hentikan dan hapus container MongoDB lama**

```bash
docker stop mongodb
docker rm mongodb
```

2. **Buat folder untuk menyimpan data MongoDB (volume)**

```bash
mkdir -p /root/mongo-data
chmod 777 /root/mongo-data
```

> Catatan: Folder ini akan menyimpan seluruh data database secara persisten.

3. **Jalankan container MongoDB baru dengan volume**

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -v /root/mongo-data:/data/db \
  --restart=always \
  arm64v8/mongo:4.4.18
```

> Periksa container:

```bash
docker ps
```

4. **Cek MongoDB dari dalam container**

```bash
docker exec -it mongodb mongo
use genieacs
show collections
```

5. **Ambil parameter dari GitHub**

```bash
mkdir -p /root/gacs-parameter
cd /root/gacs-parameter
wget https://github.com/cerdasbarus/parametergenieacs/archive/refs/heads/main.zip -O parametergenieacs.zip
unzip parametergenieacs.zip
```

6. **Restore parameter ke MongoDB**

```bash
# Stop container jika berjalan
docker stop mongodb
docker rm mongodb

# Jalankan MongoDB dengan folder parameter sebagai volume read-only
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -v /root/mongo-data:/data/db \
  -v /root/gacs-parameter/GACS-Ubuntu-22.04-main/parameter:/data/parameter:ro \
  --restart=always \
  arm64v8/mongo:4.4.18

# Restore ke database genieacs
docker exec -i mongodb mongorestore --db genieacs --drop /data/parameter
```

> Output akan menunjukkan jumlah dokumen yang berhasil di-restore.

7. **Cek apakah parameter berhasil diimport**

```bash
docker exec -it mongodb mongo
use genieacs
show collections
db.virtualParameters.find().pretty()
```

8. **Backup & Tips**

* Data sekarang tersimpan di `/root/mongo-data`, jadi aman saat restart atau reinstall Docker.
* Jika ingin backup manual:

```bash
docker exec -i mongodb mongodump --db genieacs --out /data/db-backup
```

* Restore dari backup:

```bash
docker exec -i mongodb mongorestore --db genieacs --drop /data/db-backup/genieacs
```

Selesai! Database MongoDB GenieACS sudah diperbaiki, parameter berhasil diambil dan di-restore.

