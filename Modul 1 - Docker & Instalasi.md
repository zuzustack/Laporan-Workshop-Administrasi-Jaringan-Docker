
## **Modul 1 — Docker & Instalasi** 

## **HASIL PRAKTIKUM** 

## **1. Instalasi Docker Engine** 

## docker version 

Hasil menunjukkan Docker Client dan Docker Server berhasil berjalan dengan baik. 

## **2. Verifikasi Docker Service** 

## sudo systemctl status docker 

Hal ini menunjukkan service Docker aktif dan siap digunakan. 

## **3. Menjalankan Container Pertama** 

## docker run hello-world 

Output menampilkan pesan: 

Hello from Docker! 

This message shows that your installation appears to be working correctly. 

Pesan tersebut menunjukkan instalasi Docker berhasil. 

## **4. Download dan Manajemen Image** 

## docker images 

Docker menyimpan image secara lokal sehingga dapat digunakan kembali tanpa download ulang. 

## **5. Monitoring dan Logging Container** 

Monitoring container dilakukan menggunakan: 

docker ps 

container nginx berjalan dengan port mapping 

## **6. Menjalankan Container Nginx** 

Container nginx dijalankan menggunakan mode detached dan port mapping: 

Container berhasil berjalan dan dapat diakses melalui browser menggunakan alamat: 

## http://localhost:8080 

Halaman default Nginx berhasil tampil. 

## **7. Pembuatan Custom Image dengan Dockerfile** 

Custom image dibuat menggunakan Dockerfile berbasis image nginx. 

## Contoh Dockerfile: 

**FROM** nginx:1.26-alpine 

**LABEL** maintainer="admin@pens.ac.id" 

**LABEL** description="Custom Nginx untuk praktikum Docker PENS" 

**RUN** rm -rf /usr/share/nginx/html/* **COPY** index.html /usr/share/nginx/html/index.html 

## **EXPOSE** 80 

## **CMD** ["nginx", "-g", "daemon off;"] 

## Build image dilakukan menggunakan: 

docker build -t pens-web:1.0 . 

## Container dijalankan menggunakan: 

docker run -d --name pens-app -p 9090:80 pens-web:1.0 

Halaman custom berhasil diakses melalui browser pada: 

## http://localhost:9090 

## **JAWABAN POST-LAB** 

## **1. Bandingkan output docker image history nginx dengan docker image history pens-web:1.0. Layer mana saja yang di-share?** 

Image pens-web:1.0 menggunakan nginx:1.26-alpine sebagai base image sehingga layer dasar dari nginx tetap digunakan bersama (shared layer). Layer tambahan pada pens-web:1.0 berasal dari instruksi seperti RUN rm -rf, COPY index.html, dan metadata LABEL. Dengan sistem layered filesystem, Docker tidak perlu menggandakan seluruh image sehingga penggunaan storage menjadi lebih efisien. 

## **2. Apa yang terjadi pada data di dalam container setelah container dihapus dengan docker rm? Bagaimana solusinya?** 

Data yang berada di dalam writable layer container akan ikut terhapus ketika container dihapus menggunakan docker rm. Untuk menyimpan data secara permanen digunakan Docker Volume atau bind mount sehingga data tetap tersimpan walaupun container dihapus. 

Contoh penggunaan volume: 

docker run -v mydata:/data ubuntu 

## **3. Jelaskan perbedaan antara EXPOSE di Dockerfile dan flag -p pada docker run. Apakah EXPOSE cukup untuk membuat port dapat diakses dari host?** 

EXPOSE hanya memberikan informasi bahwa container menggunakan port tertentu. Sementara itu, flag -p digunakan untuk melakukan port mapping antara host dan container. 

Contoh: 

docker run -p 8080:80 nginx 

Tanpa -p, port container tidak dapat diakses langsung dari host. Jadi EXPOSE saja tidak cukup untuk membuka akses port. 

## **4. Mengapa menggunakan tag spesifik (misal nginx:1.26) lebih baik daripada nginx:latest untuk production?** 

Tag spesifik memberikan konsistensi versi aplikasi sehingga environment production menjadi lebih stabil dan dapat diprediksi. Jika menggunakan latest, versi image dapat berubah 

sewaktu-waktu sehingga berpotensi menyebabkan bug atau incompatibility pada aplikasi. 

## **5. Berapa ukuran image alpine:3.20 dibanding ubuntu:22.04? Apa trade-off menggunakan Alpine?** 

Image alpine:3.20 berukuran sekitar 7 MB sedangkan ubuntu:22.04 dapat mencapai ratusan MB. Alpine lebih ringan dan cepat diunduh sehingga cocok untuk container minimalis. 

Namun trade-off Alpine adalah: 

- Package bawaan lebih sedikit 

- Beberapa library tidak kompatibel 

- Debugging lebih sulit 

- Menggunakan musl libc, bukan glibc 

Sedangkan Ubuntu lebih lengkap dan kompatibel untuk berbagai aplikasi. 