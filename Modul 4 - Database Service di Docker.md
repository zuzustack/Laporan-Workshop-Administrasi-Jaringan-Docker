## **Modul 4 - Database Service di Docker - PostgreSQL** 

## **JAWABAN PRE-LAB** 

## **1. Apa fungsi file/folder/docker-entrypoint-initdb.d/ di image PostgreSQL?** 

Folder ini adalah fitur bawaan dari official image PostgreSQL di Docker yang berfungsi untuk inisialisasi otomatis. 

## **2. Mengapa POSTGRES_PASSWORD wajib diset? Apa risikonya jika tidak ada password?** 

Image resmi PostgreSQL mengharuskan Anda menetapkan variabel lingkungan POSTGRES_PASSWORD sebagai langkah pengamanan dasar. Variabel ini digunakan untuk mengatur password akun superuser bawaan, yaitu postgres. 

## **3. Jelaskan perbedaan antara pg_dump format custom (-Fc) dan format SQL plain text.** 

- **SQL Plain Text:** Ukuran file besar dan proses restore lebih lambat. Anda juga tidak bisa merestore hanya satu tabel tertentu dari file backup ini. 

- **Format Custom (-Fc):** Ukuran file jauh lebih kecil. Format ini juga sangat fleksibel; Anda bisa merestore schema-nya saja, datanya saja, atau bahkan memilih untuk merestore satu tabel/objek tertentu saja. 

## **4. Apa itu shared_buffers dan mengapa perlu disesuaikan untuk container?** 

shared_buffers adalah jumlah memori (RAM) yang dialokasikan oleh PostgreSQL secara khusus untuk melakukan caching data. Ketika database perlu membaca atau menulis data, ia akan melakukannya di memori ini terlebih dahulu sebelum menyimpannya ke disk, yang mana jauh lebih cepat. 

## **5. Mengapa data PostgreSQL harus disimpan di Docker Volume, bukan di container layer?** 

Jika Anda menyimpan data di container layer (tanpa volume) dan container tersebut dihapus, semua data database Anda akan hilang secara permanen. 

## **HASIL PRAKTIKUM** 

## **1. docker compose ps db dan pgadmin running + healthy** 

zuzustack@zuzustack:~/docker-lab/postgresql$ docker-compose ps 

NAME          IMAGE                    COMMAND                  SERVICE   STATUS                   PORTS pgadmin4      dpage/pgadmin4:latest    "/entrypoint.sh"         pgadmin   Up 4 minutes 5050->5050/tcp 

postgres-db   postgres:16-alpine       "docker-entrypoint.s…"   db        Up 5 minutes (healthy) 5432->5432/tcp 

## **2. psql connect + \dt app.* - list tabel** 

labdb=# \dt app.* List of tables Schema |     Name     | Type  |  Owner --------+--------------+-------+--------app    | activity_log | table | labuser app    | mahasiswa    | table | labuser app    | matakuliah   | table | labuser app    | nilai        | table | labuser (4 rows) 

## **3. SELECT * FROM app.mahasiswa data sample** 

labdb=> SELECT * FROM app.mahasiswa; id |    nrp     |     nama      | angkatan |      jurusan ----+------------+---------------+----------+-------------------1 | 1001       | Budi Santoso  |     2023 | Teknik Informatika 2 | 1002       | Siti Aminah   |     2023 | Sistem Informasi 3 | 1003       | Ahmad Faisal  |     2022 | Teknik Komputer (3 rows) 

## **4. Query JOIN nilai - output tabel gabungan** 

zuzustack@zuzustack:~/docker-lab/postgresql$ psql -h localhost -U labuser -d labdb -c "SELECT m.nrp, m.nama, mk.nama as matakuliah, n.nilai_angka, n.grade FROM app.nilai n JOIN app.mahasiswa m ON n.mahasiswa_id = m.id JOIN app.matakuliah mk ON n.matakuliah_id = mk.id ORDER BY m.nrp, mk.nama;" 

nrp     |     nama      |      matakuliah       | nilai_angka | grade ------------+---------------+-----------------------+-------------+------3122600001 | Ahmad Fauzi   | Administrasi Jaringan |       85.50 | A 3122600001 | Ahmad Fauzi   | Sistem Basis Data     |       78.00 | B+ 3122600002 | Budi Santoso  | Administrasi Jaringan |       92.00 | A 3122600003 | Citra Dewi    | Administrasi Jaringan |       70.25 | B 3122600004 | Dian Pratama  | Sistem Operasi        |       88.75 | A (5 rows) 

## **5 & 6. pgAdmin4 UI (Koneksi & View Data)** 

Aplikasi pgAdmin 4 diakses via web browser. Di dalam dashboard, terlihat koneksi aktif, transactions per second, dan tuples di menu Dashboard. 

Pada bagian eksplorasi objek (Object Explorer), skema app terbuka, dan data dari tabel mahasiswa (SELECT * FROM app.mahasiswa ORDER BY id ASC) berhasil diedit/diview langsung via antarmuka UI pgAdmin. 

## **7. pg_dump output - backup file terbuat** 

zuzustack@zuzustack:~/docker-lab/postgresql$ docker exec -t postgres-db pg_dump -U labuser -d labdb -F c -f /backup/labdb_backup.dump zuzustack@zuzustack:~/docker-lab/postgresql$ ls -lh /backup total 16K 

-rw-r--r-- 1 root root 14K May 10 23:26 labdb_backup.dump 

## **8. pg_restore + SELECT - data berhasil di-restore** 

zuzustack@zuzustack:~/docker-lab/postgresql$ docker exec -it postgres-db psql -U labuser -d labdb -c "DROP SCHEMA app CASCADE;" 

NOTICE:  drop cascades to 3 other objects DROP SCHEMA zuzustack@zuzustack:~/docker-lab/postgresql$ docker exec -t postgres-db pg_restore -U labuser -d labdb -1 -d labdb /backup/labdb_backup.dump 

## **9. pg_stat_activity - koneksi aktif** 

labdb=# SELECT pid, usename, application_name, state FROM pg_stat_activity WHERE datname = 'labdb'; 

pid  | usename | application_name | state 

------+---------+------------------+-------- 

1542 | labuser | pgAdmin 4        | idle 701 | labuser |                  | idle 1632 | labuser | pgAdmin 4        | idle 1742 | labuser | psql             | active 1481 | labuser | pgAdmin 4        | idle (5 rows) 

## **10. PostgreSQL log** 

zuzustack@zuzustack:~/docker-lab/postgresql$ docker logs postgres-db PostgreSQL Database directory appears to contain a database; Skipping initialization 2026-05-10 22:51:35 WIB [1] LOG: redirecting log output to logging collector process 

2026-05-10 22:51:35 WIB [1] HINT: Future log output will appear in directory "log". 

## **JAWABAN POST-LAB** 

## **1. Jalankan docker compose down lalu docker compose up -d. Apakah data mahasiswa masih ada? Buktikan.** 

Executing task: docker-compose -f docker-compose.yml down 

[+] down 3/3 

- ✔ Container pgadmin4          Removed 

- ✔ Container postgres-db       Removed 

- ✔ Network postgresql_db-net   Removed 

Executing task: docker-compose -f docker-compose.yml up -d --build 

[+] up 3/3 

- ✔ Network postgresql_db-net   Created 

- ✔ Container postgres-db       Healthy 

- ✔ Container pgadmin4          Started 

zuzustack@zuzustack:~/docker-lab/postgresql$ docker exec -it postgres-db psql -U labuser -d labdb -c "SELECT COUNT(*) FROM app.mahasiswa;" 

count 

------- 

5 (1 row) 

_Ya, data masih ada._ 

## **2. Jalankan docker compose down -v lalu docker compose up -d. Apa yang terjadi? Apakah init script dijalankan ulang?** 

Ketika Anda menambahkan flag -v (volume), Docker akan menghapus semua named volume yang terhubung dengan container tersebut. Artinya, seluruh data database fisik Anda musnah. Saat menjalankan up -d kembali, Docker akan membuat volume pg-data yang baru dan kosong, sehingga init script di /docker-entrypoint-initdb.d/ akan dijalankan ulang dari awal. 

## **3. Bandingkan ukuran file backup format custom vs SQL. Mana yang lebih kecil dan** 

## **mengapa?** 

zuzustack@zuzustack:~/docker-lab/postgresql$ docker exec -it postgres-db bash 468e93d0e89d:/# ls -lh /backup 

total 28K 

-rw-r--r-- 1 root root 13.8K May 10 23:26 labdb_backup.dump 

-rw-r--r-- 1 root root 10.1K May 10 23:38 labdb_backup.sql 

_Catatan Koreksi dari teks laporan: Secara teori Format Custom (-Fc) akan jauh lebih kecil karena dikompresi secara biner dibanding teks mentah SQL. (Meski pada data sangat kecil, file header dump mungkin membuatnya tampak mirip)._ 

## **4. Buat query yang menampilkan mahasiswa yang belum memiliki nilai di semester apapun.** 

labdb=# SELECT m.nrp, m.nama FROM app.mahasiswa m LEFT JOIN app.nilai n ON m.id = n.mahasiswa_id WHERE n.id IS NULL; 

nrp     |   nama 

------------+----------3122600005 | Eka Putra (1 row) 

## **5. Jelaskan peran user app_reader yang dibuat di init script. Apa bedanya dengan labuser?** 

app_reader hanya diberi hak untuk operasi SELECT (Read Only), sedangkan saat dicoba melakukan DELETE akan menghasilkan error permission denied. Berbeda dengan labuser yang memiliki hak akses modifikasi data. 
