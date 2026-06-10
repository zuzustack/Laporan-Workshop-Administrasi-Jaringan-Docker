
## **Modul 3 - Web Service di Docker (Apache & Nginx)** 

## **JAWABAN PRE-LAB** 

## **1. Apa keuntungan menjalankan web server di container dibandingkan langsung di host?** 

Menjalankan web server di container memberikan isolasi antar aplikasi sehingga konfigurasi tidak saling bentrok. Container juga mempermudah deployment, scaling, backup, dan migrasi aplikasi karena environment dapat direproduksi dengan mudah. 

## **2. Jelaskan perbedaan document root Apache (/usr/local/apache2/htdocs/) vs Nginx** 

## **(/usr/share/nginx/html/).** 

Kedua direktori digunakan untuk menyimpan file website yang akan disajikan oleh web server, tetapi lokasi defaultnya berbeda sesuai implementasi masing-masing web server. 

## **3. Apa itu SSL Termination dan mengapa dilakukan di reverse proxy?** 

SSL Termination adalah proses decrypt koneksi HTTPS pada reverse proxy sebelum request diteruskan ke backend service. Hal ini dilakukan agar backend tidak perlu menangani SSL secara langsung sehingga konfigurasi menjadi lebih sederhana dan performa lebih efisien. 

## **4. Apa perbedaan name-based dan IP-based virtual hosting?** 

Name-based virtual hosting menggunakan nama domain berbeda pada satu alamat IP yang sama. Sedangkan IP-based virtual hosting menggunakan alamat IP berbeda untuk setiap website. Name-based virtual hosting lebih umum digunakan karena lebih hemat penggunaan IP address. 

## **5. Mengapa self-signed certificate menghasilkan warning di browser?** 

Karena self-signed certificate tidak ditandatangani oleh Certificate Authority (CA) resmi sehingga browser tidak dapat memverifikasi keaslian sertifikat tersebut. 

## **HASIL PRAKTIKUM** 

## **1. Menjalankan Docker Compose** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ docker compose ps NAME          IMAGE                COMMAND                  SERVICE   STATUS                    PORTS apache-web    httpd:alpine         "httpd-foreground"       apache    Up 17 seconds             80/tcp flask-app     python:3.11-slim     "python app.py"          flask     Up 17 seconds             5000/tcp nginx-proxy   nginx:alpine         "/docker-entrypoint.…"   proxy     Up 16 seconds 0.0.0.0:8084->80/tcp, 0.0.0.0:8443->443/tcp postgres-db   postgres:16-alpine   "docker-entrypoint.s…"   db        Up 2 minutes (healthy) 5432/tcp 

## **2. Pengujian Virtual Host Site 1** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ curl -k [https://site1.lab:8443](https://site1.lab:8443) <!DOCTYPE html> 

<html lang="id"> <head><title>Site 1 - Company Profile</title></head> 

<body> <div class="box"> <h1>Site 1 Company Profile</h1> <p>Virtual Host: <strong>site1.lab</strong></p> <p>Server: Apache httpd di Docker</p> </div> </body> </html> 

## **3. Pengujian Virtual Host Site 2** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ curl -k [https://site2.lab:8443](https://site2.lab:8443) 

<!DOCTYPE html> 

<html lang="id"> 

<head><title>Site 2 - Blog</title></head> 

<body> <div class="box"> 

<h1>Site 2 Blog</h1> 

<p>Virtual Host: <strong>site2.lab</strong></p> <p>Server: Apache httpd di Docker</p> </div> </body> </html> 

## **4. Pengujian HTTP ke HTTPS Redirect** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ curl -I [http://site1.lab:8084](http://site1.lab:8084) HTTP/1.1 301 Moved Permanently Server: nginx/1.29.8 Location: [https://site1.lab/](https://site1.lab/) 

## **5. Pengujian SSL Certificate** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ echo | openssl s_client -connect site1.lab:8443 -servername site1.lab 2>/dev/null | openssl x509 -noout -subject -issuer -dates subject=C=ID, ST=Jawa Timur, L=Surabaya, O=PENS Lab, CN=Lab issuer=C=ID, ST=Jawa Timur, L=Surabaya, O=PENS Lab, CN=Lab notBefore=May  7 07:48:13 2026 GMT notAfter=May  7 07:48:13 2027 GMT 

## **6. Pengujian API Health** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ curl -k [https://app.lab:8443/api/health](https://app.lab:8443/api/health) | python3 -m json.tool { 

"database": "PostgreSQL 16.13 on x86_64-pc-linux-musl, compiled by gcc (Alpine 15.2.0) 15.2.0, 64-bit", 

"db_status": "connected", 

"status": "ok" 

} 

## **7. Pengujian API POST Visitor** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ curl -k -X POST [https://app.lab:8443/api/visitors](https://app.lab:8443/api/visitors) -H "Content-Type: application/json" -d '{"name":"Mahasiswa PENS"}' {"id":1,"name":"Mahasiswa PENS","visited_at":"2026-05-07 07:56:58.120025"} 

## **8. Pengujian API GET Visitor** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ curl -k [https://app.lab:8443/api/visitors](https://app.lab:8443/api/visitors) | python3 -m json.tool [ 

{ 

"id": 1, 

"name": "Mahasiswa PENS", "visited_at": "2026-05-07 07:56:58.120025" 

} 

] 

## **9. Analisis Log Nginx** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ docker exec nginx-proxy cat /var/log/nginx/site1-access.log 172.21.0.1 - - [07/May/2026:07:52:40 +0000] "GET / HTTP/1.1" 200 482 "-" "curl/8.18.0" 

## **10. Analisis Log Apache** 

thinkpad_L15@DESKTOP:~/docker-lab/web-service$ docker exec apache-web cat /var/log/apache2/site1-access.log 172.21.0.4 - - [07/May/2026:07:52:40 +0000] "GET / HTTP/1.1" 200 482 "-" "curl/8.18.0" 

## **JAWABAN POST-LAB** 

## **1. Bandingkan response header dari Apache vs Nginx. Header apa yang menunjukkan software web server?** 

Header yang menunjukkan software web server adalah: Server:. Contoh: Server: nginx atau Server: Apache/2.4. 

## **2. Jika Nginx proxy down, apakah Apache masih bisa diakses langsung? Bagaimana cara testnya?** 

Apache masih dapat diakses langsung jika port Apache dipublish ke host. Pengujian dapat dilakukan menggunakan curl http://localhost:<port-apache>. Namun pada praktikum ini Apache 

tidak langsung dipublish sehingga akses utama tetap melalui Nginx proxy. 

## **3. Tunjukkan bahwa X-Real-IP header diteruskan dengan benar dari Nginx ke Flask.** 

Header diteruskan menggunakan konfigurasi: proxy_set_header X-Real-IP $remote_addr;. Flask berhasil membaca IP client melalui: request.headers.get("X-Real-IP"). 

## **4. Jelaskan mengapa Flask app perlu terhubung ke dua network (web-net dan db-net).** 

Flask perlu terhubung ke web-net agar dapat menerima request dari Nginx, dan db-net agar dapat terhubung ke PostgreSQL. Dengan pemisahan network, keamanan dan isolasi service menjadi lebih baik. 

## **5. Apa yang terjadi jika file server.key atau server.crt dihapus saat container running?** 

Nginx yang sedang berjalan masih dapat menggunakan certificate yang sudah dimuat di memory. Namun setelah container restart, HTTPS akan gagal karena file certificate tidak ditemukan. 
