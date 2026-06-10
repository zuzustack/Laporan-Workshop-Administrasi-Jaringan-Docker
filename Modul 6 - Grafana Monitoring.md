

## **Modul 6 — GRAFANA MONITORING** 

## **JAWABAN PRE-LAB** 

## **1. Jelaskan perbedaan model pull-based (Prometheus) dan push-based (Fluent Bit) dalam pengumpulan data.** 

Pada model pull-based, server monitoring secara aktif mengambil data metrics dari target menggunakan HTTP request ke endpoint /metrics pada interval tertentu. 

## Contoh: 

- Prometheus melakukan scrape ke Node Exporter 

- Prometheus melakukan scrape ke cAdvisor 

- Prometheus melakukan scrape ke Flask metrics endpoint 

Keuntungan: 

- kontrol scraping terpusat 

- mudah mengetahui target DOWN 

- tidak perlu agent mengirim data terus-menerus 

Pada model push-based, agent atau aplikasi secara aktif mengirim log atau data ke collector. 

## Contoh: 

- container Docker mengirim log ke Fluent Bit menggunakan fluentd logging driver 

- Fluent Bit meneruskan log ke PostgreSQL 

## Keuntungan: 

- cocok untuk event/log real-time 

- efisien untuk streaming log 

Perbedaan utama: 

**Pull-based Push-based** Server Client mengambil data mengirim data Cocok untuk Cocok untuk metrics logging Contoh: Contoh: Fluent Prometheus Bit 

## **2. Apa itu PromQL? Berikan contoh query untuk menghitung rata-rata CPU usage dalam 5 menit terakhir.** 

PromQL (Prometheus Query Language) adalah bahasa query yang digunakan oleh Prometheus untuk mengambil, menghitung, dan menganalisis data metrics time-series. 

Contoh query rata-rata CPU usage 5 menit terakhir: 

100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) Penjelasan: 

- rate(...[5m]) menghitung rate perubahan dalam 5 menit 

- mode="idle" mengambil idle CPU 

- 100 - idle menghasilkan persentase CPU usage 

## **3. Mengapa cAdvisor membutuhkan akses ke /var/run/docker.sock dan /sys ?** 

cAdvisor membutuhkan akses tersebut untuk membaca informasi container Docker dan resource host. 

/var/run/docker.sock digunakan untuk: 

   - membaca metadata container 

   - mengetahui container aktif 

   - membaca informasi image/container 

- /sys digunakan untuk: 

   - membaca statistik kernel Linux 

   - CPU usage 

   - memory usage 

   - cgroup metrics 

   - disk dan network metrics 

Tanpa akses tersebut cAdvisor tidak dapat mengumpulkan metrics container secara lengkap. 

## **4. Apa keuntungan Grafana provisioning (file YAML) dibanding konfigurasi manual via UI?** 

Grafana provisioning memungkinkan konfigurasi otomatis data source dan dashboard saat container pertama kali dijalankan. 

Keuntungan: 

- otomatis tanpa setup manual 

- mudah direproduksi 

- cocok untuk deployment Docker 

- konfigurasi dapat disimpan di Git 

- mempercepat setup environment baru 

- mengurangi human error 

## **5. Jelaskan perbedaan antara Gauge, Counter, dan Histogram dalam Prometheus metrics.** 

**Metric Fungsi Contoh Type** 

|**Metric**<br>**Type**|**Fungsi**|**Contoh**|
|---|---|---|
|Gauge|Nilai dapat naik dan|memory|
||turun|usage|
|Counter|Nilai hanya|total HTTP|
||bertambah|request|
|Histogra|Distribusi data|request|
|m|dalam bucket|latency|



Gauge digunakan untuk nilai yang berubah-ubah seperti CPU dan memory usage. Counter digunakan untuk menghitung total event yang terus bertambah seperti jumlah request. Histogram digunakan untuk menganalisis distribusi latency atau response time aplikasi. 

## **HASIL PRAKTIKUM** 

## **1. Menjalankan Docker Compose** 

Seluruh monitoring stack dijalankan menggunakan: 

docker compose up --build -d 

Status seluruh container dicek menggunakan: 

docker compose ps 

Hasil menunjukkan seluruh service berhasil berjalan: 

- prometheus 

- node-exporter 

- cadvisor 

- grafana 

- fluent-bit 

- postgres-db 

- nginx-web 

- flask-app 

- log-generator 

## **2. Verifikasi Prometheus Targets** 

Prometheus targets diuji melalui browser: 

## http://localhost:9090/targets 

Hasil menunjukkan seluruh target monitoring: 

- node-exporter 

- cadvisor 

- flask-app 

- prometheus 

## berstatus UP . 

## **3. Pengujian Query PromQL CPU Usage** 

Query PromQL dijalankan pada Prometheus: 

100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) 

Hasil menunjukkan persentase penggunaan CPU host secara real-time. 

## **4. Pengujian Flask Metrics Endpoint** 

Metrics Flask diuji menggunakan: 

curl -s http://localhost:5000/metrics | head -40 

Hasil menunjukkan metrics Flask berhasil di-export ke Prometheus, seperti: 

- flask_http_requests_total 

- flask_http_request_duration_seconds 

## **5. Verifikasi Login Grafana** 

Grafana berhasil diakses melalui browser: 

http://localhost:3000 

## Login menggunakan: 

- username: admin 

- password: admin123 

## Dashboard Grafana berhasil ditampilkan.. 

## **6. Verifikasi Data Sources Grafana** 

Grafana berhasil terhubung dengan: 

- Prometheus 

- PostgreSQL-Logs 

Kedua data source berhasil menampilkan status: 

Data source is working. 

## **7. Dashboard Docker Host Overview** 

Dashboard Docker Host Overview berhasil menampilkan: 

- CPU usage 

- memory usage 

- disk usage 

- uptime 

- network traffic 

## Data metrics diperoleh dari Node Exporter dan Prometheus. 

## **8. Pengujian Stress Test CPU** 

Stress test dilakukan menggunakan: 

stress --cpu 2 --timeout 60 

Saat stress test berjalan, dashboard Grafana menunjukkan lonjakan CPU usage secara real-time. 

## **9. Dashboard Container Metrics CPU** 

Dashboard Container Metrics berhasil menampilkan penggunaan CPU setiap container Docker secara real-time menggunakan data dari cAdvisor. 

## **10. Dashboard Container Metrics Memory** 

Dashboard Container Metrics berhasil menampilkan penggunaan memory setiap container Docker secara real-time. 

## **11. Dashboard Log Analytics** 

Dashboard Log Analytics berhasil menampilkan: 

- log volume time-series 

- distribusi level log 

- recent log activity 

Data log diperoleh dari PostgreSQL hasil logging Fluent Bit. 

## **12. Pie Chart Distribution Log** 

Pie chart pada dashboard Log Analytics berhasil menampilkan distribusi level log seperti: 

- INFO 

- WARNING 

- ERROR 

Grafik menunjukkan mayoritas log berada pada level INFO. 

## **13. Pembuatan Custom Panel Grafana** 

Custom panel berhasil dibuat menggunakan query: 

flask_http_requests_total 

Panel menampilkan jumlah request HTTP Flask berdasarkan endpoint aplikasi. 

## **14. Verifikasi Alert Rules** 

Alert Rules pada Grafana berhasil dibuat dan ditampilkan pada menu Alerting. 

Alert digunakan untuk monitoring kondisi tertentu seperti penggunaan CPU tinggi. 

## **JAWABAN POST-LAB** 

## **1. Dari dashboard Container Metrics, container mana yang paling banyak menggunakan CPU dan memory? Mengapa?** 

Container yang paling banyak menggunakan CPU adalah flask-app saat dilakukan stress test dan generate HTTP request secara terus-menerus. Sedangkan penggunaan memory terbesar umumnya berasal dari grafana dan postgres-db karena menyimpan dashboard, cache, dan data logging. 

## **2. Saat stress test berjalan, berapa persen CPU usage yang terukur di Grafana? Bandingkan dengan output top atau htop di host.** 

Saat stress test berjalan CPU usage meningkat hingga sekitar 70–90% tergantung spesifikasi host. Nilai tersebut hampir sama dengan output top atau htop pada host Linux karena keduanya membaca metrics kernel yang sama. 

## **3. Buat query PromQL yang menampilkan 3 container dengan memory usage tertinggi. Tunjukkan query dan hasilnya.** 

Query: 

topk(3, container_memory_usage_bytes{name!=""}) 

Hasil query menampilkan tiga container dengan penggunaan memory terbesar secara real-time. 

## **4. Dari dashboard Log Analytics, berapa rasio ERROR vs INFO log dalam 1 jam terakhir? Apakah ini normal untuk aplikasi production?** 

Log INFO memiliki jumlah jauh lebih banyak dibanding ERROR karena sebagian besar aktivitas aplikasi berjalan normal. ERROR biasanya hanya sekitar 5–15% dari total log sehingga masih dianggap normal untuk aplikasi production. 

## **5. Jika Prometheus container dihapus dan dibuat ulang (tanpa menghapus volume prom-data ), apakah data historis metrik masih ada? Buktikan.** 

Data historis metrik tetap ada karena Prometheus menggunakan Docker Volume: 

prom-data 

Selama volume tidak dihapus, seluruh time-series database Prometheus tetap tersimpan dan dapat digunakan kembali setelah container dibuat ulang. 
