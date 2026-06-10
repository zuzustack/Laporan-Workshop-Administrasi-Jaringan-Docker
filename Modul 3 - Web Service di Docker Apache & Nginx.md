## **Modul 3 — Web Service di Docker Apache & Nginx** 

## **JAWABAN PRE-LAB** 

## **1. Apa keuntungan menjalankan web server di container dibandingkan langsung di host?** 

Menjalankan web server di container memberikan isolasi antar aplikasi sehingga konfigurasi tidak saling bentrok. Container juga mempermudah deployment, scaling, backup, dan migrasi aplikasi karena environment dapat direproduksi dengan mudah. 

## **2. Jelaskan perbedaan document root Apache (/usr/local/apache2/htdocs/) vs Nginx (/usr/share/nginx/html/).** 

Document root Apache berada di: 

/usr/local/apache2/htdocs/ 

Sedangkan document root Nginx berada di: 

/usr/share/nginx/html/ 

Kedua direktori digunakan untuk menyimpan file website yang akan disajikan oleh web server, tetapi lokasi defaultnya berbeda sesuai implementasi masing-masing web server. 

## **3. Apa itu SSL Termination dan mengapa dilakukan di reverse proxy?** 

SSL Termination adalah proses decrypt koneksi HTTPS pada reverse proxy sebelum request diteruskan ke backend service. Hal 

ini dilakukan agar backend tidak perlu menangani SSL secara langsung sehingga konfigurasi menjadi lebih sederhana dan performa lebih efisien. 

## **4. Apa perbedaan name-based dan IP-based virtual hosting?** 

Name-based virtual hosting menggunakan nama domain berbeda pada satu alamat IP yang sama. Sedangkan IP-based virtual hosting menggunakan alamat IP berbeda untuk setiap website. 

Name-based virtual hosting lebih umum digunakan karena lebih hemat penggunaan IP address. 

## **5. Mengapa self-signed certificate menghasilkan warning di browser?** 

Karena self-signed certificate tidak ditandatangani oleh Certificate Authority (CA) resmi sehingga browser tidak dapat memverifikasi keaslian sertifikat tersebut. 

## **HASIL PRAKTIKUM** 

## **1. Menjalankan Docker Compose** 

Seluruh service dijalankan menggunakan Docker Compose: 

docker compose up --build -d 

Status seluruh container dicek menggunakan: 

## docker compose ps 

## **2. Pengujian Virtual Host Site 1** 

Pengujian virtual host pertama dilakukan menggunakan: 

curl -k https://site1.lab:8443 

Hasil menunjukkan halaman “Site 1 — Company Profile” berhasil ditampilkan melalui Nginx reverse proxy. 

## **3. Pengujian Virtual Host Site 2** 

Pengujian virtual host kedua dilakukan menggunakan: 

curl -k https://site2.lab:8443 

Hasil menunjukkan halaman “Site 2 — Blog” berhasil ditampilkan. 

## **4. Pengujian HTTP ke HTTPS Redirect** 

Redirect HTTP ke HTTPS diuji menggunakan: 

curl -I http://site1.lab:8084 

Hasil menunjukkan status 301 Moved Permanently sehingga redirect berhasil berjalan. 

## **5. Pengujian SSL Certificate** 

Detail SSL certificate dicek menggunakan: 

echo **|** openssl s_client -connect site1.lab:8443 -servername site1.lab 2>/dev/null **|** \ 

openssl x509 -noout -subject -issuer -dates 

Hasil menunjukkan self-signed certificate berhasil digunakan oleh Nginx. 

## **6. Pengujian API Health** 

Koneksi Flask dengan PostgreSQL diuji menggunakan: 

curl -k https://app.lab:8443/api/health **|** python3 -m json.tool 

Hasil menunjukkan status database connected. 

## **7. Pengujian API POST Visitor** 

Penambahan visitor dilakukan menggunakan: 

curl -k -X POST https://app.lab:8443/api/visitors \ 

- -H "Content-Type: application/json" \ 

- -d '{"name":"Mahasiswa PENS"}' 

Data visitor berhasil ditambahkan ke database PostgreSQL. 

## **8. Pengujian API GET Visitor** 

Daftar visitor ditampilkan menggunakan: 

curl -k https://app.lab:8443/api/visitors **|** python3 -m json.tool 

Hasil menunjukkan data visitor berhasil tersimpan dan dapat ditampilkan kembali. 

## **9. Analisis Log Nginx** 

Log akses Nginx dicek menggunakan: 

docker exec nginx-proxy cat /var/log/nginx/site1-access.log 

Hasil menunjukkan request client berhasil tercatat pada log Nginx. 

## **10. Analisis Log Apache** 

Log Apache dicek menggunakan: 

docker exec apache-web cat /var/log/apache2/site1-access.log 

Hasil menunjukkan request yang diteruskan dari Nginx berhasil dicatat pada log Apache. 

## **JAWABAN POST-LAB** 

# **1. Bandingkan response header dari Apache vs Nginx. Header apa yang menunjukkan software web server?** 

Header yang menunjukkan software web server adalah: 

Server: 

Contoh: 

Server: nginx 

atau: 

Server: Apache/2.4 

Header tersebut menunjukkan web server yang memproses request. 

## **2. Jika Nginx proxy down, apakah Apache masih bisa diakses langsung? Bagaimana cara testnya?** 

Apache masih dapat diakses langsung jika port Apache dipublish ke host. Pengujian dapat dilakukan menggunakan: 

docker ps 

curl http://localhost:<port-apache> 

Namun pada praktikum ini Apache tidak langsung dipublish sehingga akses utama tetap melalui Nginx proxy. 

## **3. Tunjukkan bahwa X-Real-IP header diteruskan dengan benar dari Nginx ke Flask.** 

Header diteruskan menggunakan konfigurasi: 

proxy_set_header X-Real-IP $remote_addr; 

Flask berhasil membaca IP client melalui: 

request.headers.get("X-Real-IP") 

Hal ini terlihat pada response JSON Flask API. 

## **4. Jelaskan mengapa Flask app perlu terhubung ke dua network (web-net dan db-net).** 

Flask perlu terhubung ke: 

## · web-net agar dapat menerima request dari Nginx 

· db-net agar dapat terhubung ke PostgreSQL 

Dengan pemisahan network, keamanan dan isolasi service menjadi lebih baik. 

## **5. Apa yang terjadi jika file server.key atau server.crt dihapus saat container running?** 

Nginx yang sedang berjalan masih dapat menggunakan certificate yang sudah dimuat di memory. Namun setelah container restart, HTTPS akan gagal karena file certificate tidak ditemukan. 