## **Modul 4 — Database Service di Docker — PostgreSQL** 

## **JAWABAN PRE-LAB** 

## **1. Apa fungsi file/folder /docker-entrypoint-initdb.d/ di image PostgreSQL?** 

Folder ini adalah fitur bawaan dari official image PostgreSQL di Docker yang berfungsi untuk inisialisasi otomatis. 

## **2. Mengapa POSTGRES_PASSWORD wajib diset? Apa risikonya jika tidak ada password?** 

Image resmi PostgreSQL mengharuskan Anda menetapkan variabel lingkungan (environment variable) 

POSTGRES_PASSWORD sebagai langkah pengamanan dasar. Variabel ini digunakan untuk mengatur password akun _superuser_ bawaan, yaitu postgres . 

## **3. Jelaskan perbedaan antara pg_dump format custom (-Fc) dan format SQL plain text.** 

SQL Plain Text: Ukuran file besar dan proses _restore_ lebih lambat. Anda juga tidak bisa merestore hanya satu tabel tertentu dari file backup ini. 

Format Custom: Ukuran file jauh lebih kecil. Format ini juga sangat fleksibel; Anda bisa merestore _schema_ -nya saja, datanya saja, atau bahkan memilih untuk merestore satu tabel/objek tertentu saja dari keseluruhan backup. Selain itu, pg_restore bisa menggunakan banyak CPU _core_ secara paralel ( -j ) agar proses _restore_ jauh lebih cepat. 

## **4. Apa itu shared_buffers dan mengapa perlu disesuaikan untuk container?** 

shared_buffers adalah jumlah memori (RAM) yang dialokasikan oleh PostgreSQL secara khusus untuk melakukan _caching_ data. Ketika database perlu membaca atau menulis data, ia akan melakukannya di memori ini terlebih dahulu sebelum menyimpannya ke disk, yang mana jauh lebih cepat. 

## **5. Mengapa data PostgreSQL harus disimpan di Docker Volume, bukan di container layer?** 

jika Anda menyimpan data di _container layer_ (tanpa volume) dan container tersebut dihapus (misalnya untuk di-update ke versi Postgres terbaru, atau Anda menjalankan docker compose down ), **semua data database Anda akan hilang secara permanen** . 

## **HASIL PRAKTIKUM** 

## **1. docker compose ps — db dan pgadmin running + healthy** 

## **2. psql connect + \dt app.* — list tabel** 

## **3. SELECT * FROM app.mahasiswa — data sample** 

## **4. Query JOIN nilai — output tabel gabungan** 

## **5. pgAdmin4 login + server connection** 

## **6. pgAdmin4 — tabel view/edit data** 

## **7. pg_dump output — backup file terbuat** 

## **8. pg_restore + SELECT — data berhasil di-restore** 

## **9. pg_stat_activity — koneksi aktif** 

## **10. PostgreSQL log — isi log file** 

## **JAWABAN POST-LAB** 

## **1. Jalankan docker compose down lalu docker compose up -d. Apakah data mahasiswa masih ada? Buktikan.** 

## **2. Jalankan docker compose down -v lalu docker compose up -d. Apa yang terjadi? Apakah init script dijalankan ulang?** 

Apa yang terjadi? Apakah init script dijalankan ulang? Ketika Anda menambahkan flag -v (volume), Docker akan menghapus semua named volume yang terhubung dengan container tersebut (termasuk pg-data). Artinya, seluruh data database fisik Anda musnah. 

Saat Anda menjalankan up -d kembali, Docker akan membuat volume pg-data yang baru dan kosong. Karena direktori data kosong, PostgreSQL mendeteksinya sebagai instalasi baru, sehingga Ya, proses inisialisasi awal dan init script di . /docker-entrypoint-initdb.d/ akan dijalankan ulang dari awal 

## **3. Bandingkan ukuran file backup format custom vs SQL. Mana yang lebih kecil dan mengapa?** 

Mana yang lebih kecil dan mengapa? 

Format Custom (-Fc) akan jauh lebih kecil dibandingkan format plain text SQL (.sql). 

Mengapa? Karena format custom milik PostgreSQL secara bawaan (default) menerapkan kompresi (zlib) pada datanya dan menyimpannya dalam format biner yang padat. Sebaliknya, format SQL menyimpan perintah mentah (INSERT INTO ...) secara berulang dalam format teks, sehingga memakan banyak ruang (terutama jika datanya jutaan baris). 

## **4. Buat query yang menampilkan mahasiswa yang belum memiliki nilai di semester apapun.** 

**5. Jelaskan peran user app_reader yang dibuat di init script. Apa bedanya dengan labuser?** 