
## **Modul 5 - Logging Service Docker dengan PostgreSQL** 

## **JAWABAN PRE-LAB** 

## **1. Mengapa centralized logging penting di lingkungan container?** 

Di lingkungan tradisional, membaca log cukup membuka satu file teks. Di lingkungan container, centralized logging wajib karena container bersifat dinamis dan dapat dihapus kapan saja; log perlu dikirim ke pusat agar tidak ikut terhapus bersama containernya. 

## **2. Apa perbedaan antara Docker logging driver json-file dan fluentd?** 

- json-file: Docker menyimpan log di file json di host. Berisiko memenuhi kapasitas hardisk jika tidak dirotasi. 

- fluentd: Docker langsung mengirim aliran log secara jaringan ke service Fluentd/Fluent Bit tanpa menyimpannya ke disk host, mencegah disk penuh. 

## **3. Jelaskan keuntungan menyimpan log di database (PostgreSQL) vs file text.** 

Kemampuan Query (SQL) presisi tinggi. Contoh: SELECT * FROM logs WHERE severity = 'ERROR'. Di file text, Anda harus menggunakan perintah grep dan regex yang rumit. 

## **4. Apa itu structured logging dan mengapa lebih baik daripada plain text log?** 

Structured logging adalah praktik menulis log dalam format data yang bisa dibaca dan diparsing oleh mesin (biasanya JSON), bukan sekadar kalimat panjang. Ini memudahkan indexing dan query. 

## **5. Mengapa Fluent Bit lebih cocok untuk sidecar/edge collection dibanding Fluentd?** 

Fluent Bit sangat ringan (C, ~1 MB memori) cocok untuk edge atau sidecar. Fluentd lebih kaya fitur tapi butuh resource lebih besar (~40MB, Ruby/C) cocok sebagai layer aggregator pusat. 

## **HASIL PRAKTIKUM** 

## **1. docker compose ps** 

zuzustack@zuzustack:~/docker-lab/logging$ docker-compose ps 

NAME            IMAGE                   SERVICE         STATUS                   PORTS flask-app       logging-flask-app       flask-app       Up 2 minutes             0.0.0.0:5000->5000/tcp fluent-bit      fluent/fluent-bit:latest fluent-bit     Up 2 minutes             0.0.0.0:24224->24224/tcp, 0.0.0.0:24224->24224/udp log-generator   logging-log-generator   log-generator   Up 2 minutes nginx-web       nginx:alpine            nginx-web       Up 2 minutes             0.0.0.0:8080->80/tcp postgres-db     postgres:16-alpine      postgres-db     Up 2 minutes (healthy) 0.0.0.0:5432->5432/tcp 

## **2. Fluent Bit menerima log** 

zuzustack@zuzustack:~/docker-lab/logging$ docker-compose logs fluent-bit fluent-bit | [2026/05/10 16:56:57.083] [error] [engine] chunk 1-1778432206... cannot be retried fluent-bit | {"date":1778432214.8, "timestamp": "2026-05-10T16:56:54.986", "level": "WARN", "service": "log-generator", "message": "Deprecated API endpoint called..."} 

## **3. SELECT COUNT** 

labdb=> SELECT COUNT(*) FROM logs.container_logs; count ------342 (1 row) 

## **4. Sample Log Terbaru** 

labdb=> SELECT * FROM logs.recent_logs LIMIT 10; id  |        time         |   container   |  level   |             message -----+---------------------+---------------+----------+--------------------------------342 | 2026-05-11 00:12:15 | log-generator | INFO     | Background job complete 341 | 2026-05-11 00:12:14 | nginx-web     | INFO     | 192.168.65.1 [11/Ma... 340 | 2026-05-11 00:12:13 | flask-app     | INFO     | Index accessed from 192... 339 | 2026-05-11 00:12:12 | log-generator | WARN     | Memory usage at 92% a... 

## **5 & 6. Distribusi per container dan level** 

labdb=> SELECT container_name, COUNT(*) AS total FROM logs.container_logs GROUP BY container_name ORDER BY total DESC; container_name | total ----------------+------log-generator  |   180 nginx-web      |   110 flask-app      |    52 

labdb=> SELECT log_level, COUNT(*) AS total FROM logs.container_logs GROUP BY log_level ORDER BY total DESC; 

log_level | total 

-----------+------- 

INFO      |   210 DEBUG     |    70 WARN      |    40 ERROR     |    15 CRITICAL  |     7 

## **9. curl /api/logs/stats** 

$ curl -s http://localhost:5000/api/logs/stats | python3 -m json.tool { "last_hour": [ { "count": 210, "level": "INFO" }, { "count": 70, "level": "DEBUG" } ] } 

## **11. Fluentbit Logs JSON Dump** 

labdb=> SELECT tag, time, data FROM logs.fluentbit LIMIT 3; -[ RECORD 1 ]---------------------------------------------------------tag  | docker.nginx time | 2026-05-25 21:36:13 data | {"log": "/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty...", "source": "stdout"} 

## **JAWABAN POST-LAB** 

## **1. Berapa total log yang masuk ke PostgreSQL setelah 5 menit? Tunjukkan distribusi per container dan per level.** 

Total log ada 342. (Distribusi sudah ditunjukkan di query poin 5 & 6 Hasil Praktikum). 

## **2. Tulis query SQL yang menampilkan log rate per menit selama 10 menit terakhir.** 

SELECT 

date_trunc('minute', received_at) AS minute, 

COUNT(*) AS logs_per_minute FROM logs.container_logs 

WHERE received_at > NOW() - INTERVAL '10 minutes' GROUP BY minute 

ORDER BY minute; 

## **3. Apa yang terjadi jika container fluent-bit di-stop? Apakah container lain juga stop? Apakah log hilang?** 

- **Apakah container lain stop?** Tidak. Container lain akan tetap berjalan normal karena dikonfigurasi menggunakan fluentd-async: "true". 

- **Apakah log hilang?** Ya. Karena Fluent Bit mati, log baru yang diproduksi akan dibuang (dropped) oleh Docker setelah buffer sementara penuh. 

## **4. Jelaskan alur sebuah log entry dari log-generator stdout sampai masuk ke tabel** 

## **container_logs.** 

1. Aplikasi mengeksekusi print JSON ke stdout. 

2. Docker Logging Driver (fluentd) menangkap stdout dan mengirimnya via TCP ke port 24224. 

3. Fluent Bit menerima data. 

4. Fluent Bit Filter mengekstrak JSON dan mengubah nama field. 

5. Fluent Bit Output (pgsql) melakukan koneksi ke DB dan eksekusi INSERT INTO logs.container_logs. 

## **6. Modifikasi LOG_INTERVAL menjadi 0.5 detik. Berapa log rate per menit yang dihasilkan?** 

1 log setiap 0.5 detik = 2 log/detik. 

Dalam 1 menit: 2 * 60 = 120 log/menit murni dari log-generator. 
