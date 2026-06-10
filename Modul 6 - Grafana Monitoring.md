
## **Modul 6 - GRAFANA MONITORING** 

## **JAWABAN PRE-LAB** 

## **1. Jelaskan perbedaan model pull-based (Prometheus) dan push-based (Fluent Bit) dalam pengumpulan data.** 

- **Pull-based (Prometheus):** Server monitoring secara aktif mengambil (scrape) data metrics dari target. Keuntungan: kontrol terpusat, mudah deteksi target DOWN. 

- **Push-based (Fluent Bit):** Agent/aplikasi secara aktif mengirim log ke collector. Keuntungan: cocok untuk event real-time dan efisien untuk streaming. 

## **2. Apa itu PromQL? Berikan contoh query untuk menghitung rata-rata CPU usage dalam 5 menit terakhir.** 

PromQL adalah bahasa query untuk Prometheus. 

Contoh CPU idle percentage to usage: 

100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) 

## **3. Mengapa cAdvisor membutuhkan akses ke /var/run/docker.sock dan /sys?** 

- /var/run/docker.sock: Membaca metadata container dan status image. 

- /sys: Membaca statistik kernel Linux (cgroup metrics, CPU, memory, IO host). 

## **4. Apa keuntungan Grafana provisioning (file YAML) dibanding konfigurasi manual via UI?** 

Grafana provisioning memungkinkan konfigurasi otomatis data source dan dashboard saat container berjalan. Sangat mudah direproduksi, mencegah human error, dan cocok dikelola dalam Git. 

## **5. Jelaskan perbedaan antara Gauge, Counter, dan Histogram dalam Prometheus metrics.** 

- **Gauge:** Nilai bisa naik-turun (contoh: memory usage). 

- **Counter:** Nilai hanya bertambah/akumulatif (contoh: total request HTTP). 

- **Histogram:** Distribusi frekuensi/latency ke dalam bucket (contoh: response time request). 

## **HASIL PRAKTIKUM** 

## **1. Menjalankan Docker Compose** 

thinkpad_L15@DESKTOP:~/docker-lab/monitoring$ docker compose ps NAME            SERVICE         STATUS                   PORTS cadvisor        cadvisor        Up 2 minutes (healthy)   0.0.0.0:8083->8080/tcp flask-app       flask-app       Up 17 seconds            0.0.0.0:5000->5000/tcp fluent-bit      fluent-bit      Up 2 minutes             0.0.0.0:24224->24224/tcp grafana         grafana         Up 54 seconds            0.0.0.0:3001->3000/tcp log-generator   log-generator   Up 17 seconds nginx-web       nginx-web       Up 17 seconds            0.0.0.0:8082->80/tcp node-exporter   node-exporter   Up 2 minutes             0.0.0.0:9100->9100/tcp postgres-db     postgres-db     Up 2 minutes (healthy)   0.0.0.0:5432->5432/tcp prometheus      prometheus      Up 2 minutes             0.0.0.0:9090->9090/tcp 

## **2. Verifikasi Prometheus Targets** 

Prometheus targets diuji melalui browser di http://localhost:9090/targets. Semua endpoint target terpantau dalam status **UP** (node-exporter, flask-app, prometheus), dengan warna hijau indikator pada dashboard Prometheus. 

## **4. Pengujian Flask Metrics Endpoint** 

thinkpad_L15@DESKTOP:~/docker-lab/monitoring$ curl -s http://localhost:5000/metrics | head -40 

# HELP python_gc_objects_collected_total Objects collected during gc # TYPE python_gc_objects_collected_total counter python_gc_objects_collected_total{generation="0"} 386.0 

... 

# HELP process_resident_memory_bytes Resident memory size in bytes. 

# TYPE process_resident_memory_bytes gauge process_resident_memory_bytes 4.3114496e+07 

## **5 & 6. Verifikasi Login Grafana & Data Sources** 

Login Grafana di http://localhost:3001 berhasil. Pada menu Data Sources, koneksi ke 

**Prometheus** dan **PostgreSQL-Logs** diperiksa, dan menampilkan pesan sukses: 

## ✔ **Database Connection OK** > ✔ **Successfully queried the Prometheus API.** 

## **7 & 8. Dashboard Docker Host Overview & Stress Test** 

Dashboard ini menyajikan Gauge untuk CPU Usage %, Memory Usage %, Disk Usage %, dan System Uptime. 

Saat command stress test dieksekusi: stress --cpu 2 --timeout 60 

Tampilan di Grafana merespons real-time dengan grafik **CPU Usage Over Time** yang melonjak tajam mendekati 90% pada garis Total CPU %. 

## **9-14. Visualisasi Dashboard** 

Dashboard terintegrasi penuh: 

- **Container Metrics (cAdvisor):** Memantau memori dan CPU limit per nama container. 

- **Log Analytics:** Mengambil pie-chart dari jumlah INFO, WARN, ERROR menggunakan kueri SQL dari database Postgre. 

- Panel kustom untuk menghitung flask_http_requests_total berhasil dibuat. 

## **JAWABAN POST-LAB** 

## **1. Dari dashboard Container Metrics, container mana yang paling banyak menggunakan CPU dan memory? Mengapa?** 

Container yang paling banyak menggunakan CPU adalah flask-app saat dilakukan stress test. Penggunaan memory terbesar umumnya berasal dari grafana dan postgres-db karena menyimpan aplikasi web, cache, dan operasional log data yang cukup besar di RAM. 

## **2. Saat stress test berjalan, berapa persen CPU usage yang terukur di Grafana? Bandingkan dengan output top atau htop di host.** 

Saat stress test, CPU usage meningkat hingga sekitar 70-90%. Nilai tersebut hampir sama dengan output top atau htop pada host Linux karena Grafana/Prometheus membaca metric kernel OS yang sama melalui Node Exporter. 

## **3. Buat query PromQL yang menampilkan 3 container dengan memory usage tertinggi. Tunjukkan query dan hasilnya.** 

## **Query:** 

topk(3, container_memory_usage_bytes{name!=""}) 

_Hasil query ini memfilter data container non-kosong lalu menampilkan tiga container teratas di dashboard Grafana secara real-time._ 

## **4. Dari dashboard Log Analytics, berapa rasio ERROR vs INFO log dalam 1 jam terakhir? Apakah ini normal untuk aplikasi production?** 

Log INFO memiliki jumlah jauh lebih banyak dibanding ERROR (berdasarkan tabel log_level sebelumnya 210 vs 15). Rasio ERROR di bawah 5-15% dari total traffic masih dianggap wajar/normal untuk log exception di lingkungan production. 

## **5. Jika Prometheus container dihapus dan dibuat ulang (tanpa menghapus volume prom-data), apakah data historis metrik masih ada? Buktikan.** 

Data historis metrik **tetap ada** karena Prometheus menggunakan Docker Volume: prom-data. Selama named volume tersebut tidak dihapus (via perintah down -v), seluruh blok time-series database tetap utuh dan grafiknya akan kembali tersambung begitu container menyala ulang. 

