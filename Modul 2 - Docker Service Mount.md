
## **Modul 2 — Docker Service Mount** 

## **JAWABAN SOAL PRE-LAB** 

## **1. Apa perbedaan default bridge dan user-defined bridge network?** 

Default bridge adalah network bawaan Docker yang otomatis dibuat saat Docker diinstal. Pada default bridge, container tidak dapat melakukan DNS resolution berdasarkan nama container. 

Sedangkan user-defined bridge adalah network yang dibuat sendiri oleh user menggunakan perintah docker network create. Network ini mendukung DNS resolution otomatis sehingga container dapat saling berkomunikasi menggunakan nama container. 

Selain itu user-defined bridge memiliki isolasi network yang lebih baik dan lebih fleksibel untuk pengelolaan container. 

## **2. Kapan menggunakan Volume vs Bind Mount vs tmpfs?** 

Docker Volume digunakan ketika membutuhkan penyimpanan data yang persisten dan dikelola Docker, misalnya database atau data aplikasi. 

Bind Mount digunakan ketika ingin menghubungkan file atau folder host dengan container, biasanya untuk development agar perubahan file langsung terlihat di container. 

Sedangkan tmpfs digunakan untuk data sementara yang sensitif atau cache karena data disimpan di RAM dan akan hilang ketika container dihentikan atau restart. 

## **3. Apa yang terjadi pada named volume saat docker compose down? Bagaimana jika pakai flag -v?** 

Saat menjalankan docker compose down, container dan network akan dihapus tetapi named volume tetap tersimpan sehingga data tidak hilang. 

Namun jika menggunakan flag -v: 

docker compose down -v 

maka volume juga ikut dihapus sehingga seluruh data di dalam volume akan hilang. 

## **4. Apa fungsi depends_on dan healthcheck di docker-compose.yml?** 

depends_on digunakan untuk menentukan urutan startup antar service sehingga service tertentu dijalankan lebih dahulu sebelum service lain. 

Sedangkan healthcheck digunakan untuk mengecek apakah service benar-benar siap digunakan. Contohnya database PostgreSQL dicek menggunakan pg_isready sebelum aplikasi Flask dijalankan. 

## **5. Mengapa user-defined bridge bisa DNS resolve nama container, sedangkan default bridge tidak?** 

Karena user-defined bridge memiliki embedded DNS server bawaan Docker yang otomatis melakukan mapping nama container ke IP address. 

Sedangkan default bridge tidak memiliki fitur DNS otomatis sehingga komunikasi antar container harus menggunakan IP address secara manual. 

## **HASIL PRAKTIKUM** 

## **1. Docker Network** 

## Eksplorasi network Docker dilakukan menggunakan: 

## docker network ls 

## docker network inspect bridge 

## Pengujian komunikasi antar container dilakukan menggunakan: 

docker exec server-a ping -c 3 server-b docker exec server-b ping -c 3 server-a 

Hasil menunjukkan kedua container berhasil saling terhubung menggunakan nama container. Hal ini membuktikan bahwa user-defined bridge mendukung DNS resolution otomatis. 

## **2. Docker Volume** 

docker volume inspect data-vol 

Volume digunakan untuk menyimpan data log: 

Data tetap tersedia walaupun container sudah dihapus. Hal ini menunjukkan Docker Volume bersifat persistent. 

## **3. Backup dan Restore Volume** 

Backup volume dilakukan menggunakan: 

docker run --rm \ 

-v data-vol:/source:ro \ 

-v $(pwd):/backup \ alpine:3.20 tar czf /backup/data-vol-backup.tar.gz -C /source . 

Restore dilakukan ke volume baru: 

docker volume create data-vol-restored 

Kemudian data dipulihkan menggunakan: 

docker run --rm \ 

-v data-vol-restored:/target \ 

-v $(pwd):/backup:ro \ alpine:3.20 tar xzf /backup/data-vol-backup.tar.gz -C /target 

Data berhasil dipulihkan dan dapat dibaca kembali. 

## **4. Bind Mount** 

Bind mount digunakan untuk menghubungkan direktori host dengan container: 

docker run -d --name dev-server \ 

-p 8080:80 \ 

-v $(pwd)/html:/usr/share/nginx/html:ro \ nginx:alpine 

File index.html yang berada di host dapat langsung diakses dari container. Ketika file diubah pada host, perubahan langsung terlihat pada browser tanpa restart container. 

Hal ini menunjukkan bind mount sangat berguna untuk development workflow dan live-reload. 

## **5. tmpfs Mount** 

Container dijalankan menggunakan tmpfs mount: 

docker run -d --name tmpfs-demo \ --tmpfs /app/cache:size=64m \ alpine:3.20 sh -c "echo 'secret-data' > /app/cache/token.txt && sleep 3600" 

Isi file dicek menggunakan: 

docker exec tmpfs-demo cat /app/cache/token.txt 

Setelah container di-restart: 

docker stop tmpfs-demo **&&** docker start tmpfs-demo 

File sudah tidak tersedia lagi. Hal ini karena tmpfs menggunakan RAM sehingga data bersifat sementara dan tidak persisten. 

## **6. Docker Compose** 

Docker Compose digunakan untuk menjalankan aplikasi multi-container yang terdiri dari: 

- Nginx Web Server 

- Flask Application 

- PostgreSQL Database 

Aplikasi dijalankan menggunakan: 

docker compose up --build -d 

Status service dicek menggunakan: 

docker compose ps 

Semua service berhasil berjalan dan saling terhubung melalui network Docker Compose. 

## **7. Pengujian Aplikasi** 

Aplikasi web berhasil diakses melalui browser: 

http://localhost:8081 

API backend berhasil diuji menggunakan: 

curl http://localhost:8080/api/health 

Hasil menunjukkan Flask berhasil terhubung dengan PostgreSQL.\ 

## **JAWABAN SOAL POST-LAB** 

## **1. Jalankan docker network inspect lab-frontend. Sebutkan container dan IP masing-masing.** 

Hasil docker network inspect menunjukkan container yang terhubung pada network frontend, misalnya: 

· lab-web → 172.20.0.2 

· lab-app → 172.20.0.3 

Setiap container mendapatkan IP virtual pada network Docker. 

## **2. Hapus container lab-db lalu docker compose up -d lagi. Apakah data PostgreSQL masih ada? Mengapa?** 

Data PostgreSQL tetap tersedia karena database menggunakan named volume pg-data. Volume disimpan secara terpisah dari container sehingga data tetap persisten walaupun container dihapus. 

## **3. Tunjukkan perbedaan output docker inspect untuk mount type volume vs bind.** 

Mount type volume menunjukkan lokasi penyimpanan yang dikelola Docker pada: 

## /var/lib/docker/volumes/ 

Sedangkan mount type bind menunjukkan path langsung menuju direktori host, misalnya: 

## /home/user/project/html 

Volume lebih portable sedangkan bind mount bergantung pada struktur direktori host. 

## **4. Jelaskan alur request dari browser → Nginx → Flask → PostgreSQL.** 

Browser mengakses Nginx pada port 8080. Nginx bertindak sebagai reverse proxy dan meneruskan request API ke Flask Application. Flask kemudian memproses request dan melakukan koneksi ke PostgreSQL untuk mengambil data database. Hasil query dikembalikan ke Flask, diteruskan ke Nginx, lalu dikirim kembali ke browser. 

## **5. Bandingkan ukuran image yang digunakan stack ini. Mana terbesar dan mengapa?** 

Image terbesar biasanya adalah postgres:16-alpine karena berisi database engine lengkap beserta dependency penyimpanan data. Image python:3.11-slim memiliki ukuran sedang karena digunakan untuk menjalankan Flask Application. Sedangkan nginx:alpine menjadi image paling kecil karena menggunakan Alpine Linux yang minimalis. 