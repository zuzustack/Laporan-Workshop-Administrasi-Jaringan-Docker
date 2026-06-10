## **Modul 5 — Logging Service Docker dengan PostgreSQL JAWABAN PRE-LAB** 

## **1. Mengapa centralized logging penting di lingkungan container?** 

Di lingkungan tradisional (satu server monolitik), membaca log cukup dengan membuka satu file teks. Namun, di lingkungan container (seperti Docker/Kubernetes), centralized logging (log terpusat) menjadi hal yang wajib 

## **2. Apa perbedaan antara Docker logging driver json-file dan fluentd?** 

**json-file (Default):** Docker menangkap _output container_ dan menuliskannya ke dalam file berekstensi .json di _host machine_ (biasanya di /var/lib/docker/containers/ ). Ini mudah digunakan, tetapi berisiko memenuhi kapasitas _hardisk server host_ Anda jika file log tidak dirotasi (log rotation). 

**fluentd :** Docker **tidak menyimpan file log di** _**host disk**_ sama sekali. Sebaliknya, Docker langsung mengirimkan aliran _log_ tersebut secara jaringan (via TCP/UDP) ke sebuah _service_ 

Fluentd yang sedang berjalan. Ini sangat ideal untuk _production_ karena mencegah disk _server host_ penuh dan langsung mem-teruskan _log_ ke sistem terpusat. 

## **3. Jelaskan keuntungan menyimpan log di database (PostgreSQL) vs file text.** 

**Kemampuan** _**Query**_ **(SQL):** Anda bisa memfilter _log_ dengan sangat presisi. Contoh: SELECT * FROM logs WHERE severity = 'ERROR' AND timestamp > NOW() - 

INTERVAL '1 hour' . Di _file text_ , Anda harus menggunakan perintah grep dan _regex_ yang rumit. 

## **4. Apa itu structured logging dan mengapa lebih baik daripada plain text log?** 

_Structured logging_ adalah praktik menulis _log_ dalam format data yang bisa dibaca dan di- _parsing_ oleh mesin secara terstruktur (biasanya dalam format **JSON** ), bukan sekadar kalimat panjang. 

## **5. Mengapa Fluent Bit lebih cocok untuk sidecar/edge** 

## **collection dibanding Fluentd?** 

**Fluent Bit (Sang Pengumpul/Forwarder):** Ditulis dalam bahasa C. Sangat ringan, konsumsi memori sangat kecil (~1 MB), dan butuh daya CPU yang minim. Ini membuatnya sangat ideal dipasang di setiap _server_ kecil (edge) atau ditempelkan di sebelah aplikasi utama Anda di dalam satu _pod_ ( _sidecar container_ ). Tugas utamanya hanya mengambil log dan melemparkannya ke atas. 

**Fluentd (Sang Agregator/Processor):** Ditulis dalam bahasa Ruby dan C. Ekosistem _plugin_ -nya sangat masif dan mampu melakukan transformasi data (filtering, modifikasi, routing) yang sangat kompleks. Namun, ia lebih rakus _resource_ (butuh memori ~40MB+). Biasanya diletakkan di _layer_ tengah 

## sebagai penerima data dari agen-agen Fluent Bit, memprosesnya, lalu menyimpannya ke Elasticsearch atau PostgreSQL. 

## **HASIL PRAKTIKUM** 

## **1. docker compose ps — 5 service running** 

## **2. docker compose logs fluent-bit — Fluent Bit** 

## **menerima log** 

## **3. SELECT COUNT(*) FROM logs.container_logs —** 

## **jumlah total log** 

## **4. SELECT * FROM logs.recent_logs LIMIT 10 —** 

## **sample log terbaru** 

## **5. Query distribusi per container — output tabel** 

## **6. Query distribusi per level — output tabel** 

## **7. SELECT * FROM logs.error_summary — summary** 

## **error** 

## **8. Query log rate per menit — output tabel** 

## **9. curl /api/logs/stats — response JSON** 

## **10. curl /api/logs/search?q=error — response JSON** 

## **11. SELECT tag, time, data FROM logs.fluentbit** 

**==> picture [65 x 16] intentionally omitted <==**

**----- Start of picture text -----**<br>
LIMIT 3;<br>**----- End of picture text -----**<br>


## **12. SELECT * FROM logs.fluentbit; 13. SELECT tag, time, data FROM logs.fluentbit LIMIT 3;** 

## **14. SELECT * FROM logs.recent_logs LIMIT 10; 15. SELECT * FROM logs.structured_logsLIMIT 10;** 

## **16. SELECTdate_trunc('minute', time) AS minute,COUNT(*) AS logs_per_minuteFROM logs.fluentbitWHERE time > NOW() - INTERVAL '5 minutes' GROUP BY minute ORDER BY minute;** 

## **JAWABAN POST-LAB** 

**1. Berapa total log yang masuk ke PostgreSQL setelah 5 menit? Tunjukkan distribusi per container dan per level.** 

## **2. Tulis query SQL yang menampilkan log rate per menit selama 10 menit terakhir.** 

## **3. Apa yang terjadi jika container fluent-bit di-stop? Apakah container lain juga stop? Apakah log hilang?** 

**Apakah container lain juga stop? Tidak.** Container 

nginx-web , flask-app , dan log-generator akan tetap berjalan normal. Ini karena di konfigurasi 

docker-compose.yml , Anda menambahkan opsi 

fluentd-async: "true" . Opsi ini menginstruksikan Docker untuk tidak memblokir aplikasi meskipun _log forwarder_ (Fluent Bit) mati atau tidak merespons. 

**Apakah log hilang? Ya.** Karena Fluent Bit mati, log baru yang diproduksi oleh aplikasi selama masa _downtime_ tersebut akan dibuang ( _dropped_ ) oleh Docker setelah _buffer_ internal sementara penuh, dan tidak akan pernah masuk ke PostgreSQL. Inilah kelemahan desain _direct forwarding_ tanpa _buffer disk_ permanen. 

## **4. Jelaskan alur sebuah log entry dari log-generator stdout sampai masuk ke tabel container_logs.** 

1. **Log Creation (Aplikasi):** _Script_ Python di log-generator mengeksekusi print(json.dumps(...)) yang mencetak teks berformat JSON ke stdout (Standard Output). 

2. **Docker Logging Driver:** Karena log-generator dikonfigurasi menggunakan driver: fluentd , Docker daemon menangkap stdout tersebut (tidak menuliskannya ke file JSON di _host_ ) dan langsung mengirimkannya via koneksi TCP ke localhost:24224 . 

3. **Data Collection (Fluent Bit Input):** Fluent Bit yang _listen_ di port 24224 menerima data mentah dari Docker dengan tag . 

docker.generator 

4. **Data Processing (Fluent Bit Filter):** Data melewati konfigurasi [FILTER] . Fluent Bit mengekstrak struktur JSON menggunakan parser docker_json , lalu mengubah nama variabel internal ( log menjadi message , container_name menjadi source_container ) via filter modify . 

5. **Log Storage (Fluent Bit Output):** Melalui konfigurasi [OUTPUT] pgsql , Fluent Bit membuat koneksi ke postgres-db:5432 menggunakan _username/password_ lab, lalu mengeksekusi perintah INSERT INTO 

   - logs.container_logs ... untuk setiap baris log yang sudah diproses. 

## **6. Modifikasi LOG_INTERVAL menjadi 0.5 detik. Berapa log rate per menit yang dihasilkan?** 

Perhitungan: 

Aplikasi akan mencetak 1 log setiap 0,5 detik. 

Dalam 1 detik: 1 / 0.5 = 2 log/detik. 

Dalam 1 menit: 2 * 60 = 120 log/menit. 

Jadi, log rate per menit yang murni dihasilkan dari log-generator adalah sekitar 120 log/menit (ditambah deviasi sangat kecil karena adanya random.uniform(-0.5, 0.5) di kode generatornya). Jika digabung dengan log Nginx atau Flask (jika sedang diakses), rate ini akan lebih tinggi. 