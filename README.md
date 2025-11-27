# GenieACS Parameter Configuration

Tutorial ini menjelaskan cara **memperbaiki MongoDB GenieACS**, mengambil parameter dari GitHub, dan melakukan restore ke database.

---

## 1. Hentikan dan Hapus Container MongoDB Lama

```bash
docker stop mongodb
docker rm mongodb
```

---

## 2. Buat Folder untuk Menyimpan Data MongoDB (Volume)

```bash
mkdir -p /root/mongo-data
chmod 777 /root/mongo-data
```

> **Catatan:** Folder ini akan menyimpan seluruh data database secara persisten, sehingga aman saat restart atau reinstall Docker.

---

## 3. Jalankan Container MongoDB Baru dengan Volume

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -v /root/mongo-data:/data/db \
  --restart=always \
  arm64v8/mongo:4.4.18
```

Periksa container:

```bash
docker ps
```

---

## 4. Cek MongoDB dari Dalam Container

```bash
docker exec -it mongodb mongo
use genieacs
show collections
```

---

## 5. Ambil Parameter dari GitHub

```bash
mkdir -p /root/gacs-parameter
cd /root/gacs-parameter
wget https://github.com/cerdasbarus/parametergenieacs/archive/refs/heads/main.zip -O parametergenieacs.zip
unzip parametergenieacs.zip
```

---

## 6. Restore Parameter ke MongoDB

1. Stop container jika berjalan:

```bash
docker stop mongodb
docker rm mongodb
```

2. Jalankan MongoDB dengan folder parameter sebagai volume **read-only**:

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -v /root/mongo-data:/data/db \
  -v /root/gacs-parameter/parametergenieacs-main/parameter:/data/parameter:ro \
  --restart=always \
  arm64v8/mongo:4.4.18
```

3. Restore parameter ke database `genieacs`:

```bash
docker exec -i mongodb mongorestore --db genieacs --drop /data/parameter
```

> Output akan menampilkan jumlah dokumen yang berhasil di-restore.

---

## 7. Cek Apakah Parameter Berhasil Diimport

```bash
docker exec -it mongodb mongo
use genieacs
show collections
db.virtualParameters.find().pretty()
```

---

## 8. Backup & Tips

* Data tersimpan di `/root/mongo-data`, aman saat restart atau reinstall Docker.
* Backup manual:

```bash
docker exec -i mongodb mongodump --db genieacs --out /data/db-backup
```

* Restore dari backup:

```bash
docker exec -i mongodb mongorestore --db genieacs --drop /data/db-backup/genieacs
```

---

✅ **Selesai!**
Database MongoDB GenieACS sudah diperbaiki, parameter berhasil diambil dan di-restore.
