## **Modul 1 - Docker & Instalasi** 

## **HASIL PRAKTIKUM** 

## **1. Instalasi Docker Engine** 

student@student:~$ docker version Client: Docker Engine - Community Version:           29.4.1 API version:       1.54 (minimum version 1.40) Go version:        go1.26.2 Git commit:        055a478 Built:             Mon Apr 20 16:32:37 2026 OS/Arch:           linux/amd64 Context:           default Server: Docker Engine - Community Engine: Version:          29.4.1 API version:      1.54 (minimum version 1.40) Go version:       go1.26.2 Git commit:       6c91b92 Built:            Mon Apr 20 16:32:37 2026 OS/Arch:          linux/amd64 Experimental:     false containerd: Version:          v2.2.3 GitCommit:        77c84241c7cbdd9b4eca2591793e3d4f4317c590 runc: Version:          1.3.5 GitCommit:        v1.3.5-0-g488fc13e docker-init: Version:          0.19.0 GitCommit:        de40ad0 

_Hasil menunjukkan Docker Client dan Docker Server berhasil berjalan dengan baik._ 

## **2. Verifikasi Docker Service** 

student@student:~$ sudo systemctl status docker 

● docker.service - Docker Application Container Engine 

Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled) Active: active (running) since Thu 2026-04-30 01:57:17 UTC; 10min ago 

TriggeredBy: ● docker.socket Docs: [https://docs.docker.com](https://docs.docker.com) Main PID: 1867 (dockerd) Tasks: 10 Memory: 27.9M (peak: 31.1M) CPU: 334ms CGroup: /system.slice/docker.service └─ 1867 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock 

_Hal ini menunjukkan service Docker aktif dan siap digunakan._ 

## **3. Menjalankan Container Pertama** 

student@student:~$ docker run hello-world 

Hello from Docker! 

This message shows that your installation appears to be working correctly. 

_Pesan tersebut menunjukkan instalasi Docker berhasil._ 

## **4. Download dan Manajemen Image** 

student@student:~$ docker images REPOSITORY                 TAG       IMAGE ID       CREATED         SIZE alpine                     3.20      d9e853e87e55   2 weeks ago     7.33MB hello-world                latest    f9078146db2e   3 months ago    9.14kB nginx                      1.26      41b194461e4b   2 weeks ago     188MB nginx                      latest    6e23479198b9   2 weeks ago     188MB ubuntu                     22.04     962f6cadeae    3 weeks ago     77.8MB 

_Docker menyimpan image secara lokal sehingga dapat digunakan kembali tanpa download ulang._ 

## **5. Monitoring dan Logging Container** 

Monitoring container dilakukan menggunakan: student@student:~$ docker ps CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS NAMES 

4a5dcd77bcef   nginx:alpine   "/docker-entrypoint.…"   9 seconds ago   Up 8 seconds 0.0.0.0:8080->80/tcp, :::8080->80/tcp   web-public 

_Container nginx berjalan dengan port mapping._ 

## **6. Menjalankan Container Nginx** 

Container nginx dijalankan menggunakan mode detached dan port mapping. Container 

berhasil berjalan dan dapat diakses melalui browser menggunakan alamat: http://localhost:8080 

## **Welcome to nginx!** 

If you see this page, the nginx web server is successfully installed and working. Further configuration is required. 

For online documentation and support please refer to nginx.org. To engage with the community please visit community.nginx.org. _Thank you for using nginx. Halaman default Nginx berhasil tampil._ 

## **7. Pembuatan Custom Image dengan Dockerfile** 

Custom image dibuat menggunakan Dockerfile berbasis image nginx. Contoh Dockerfile: FROM nginx:1.26-alpine LABEL maintainer="admin@pens.ac.id" LABEL description="Custom Nginx untuk praktikum Docker PENS" RUN rm -rf /usr/share/nginx/html/* COPY index.html /usr/share/nginx/html/index.html EXPOSE 80 CMD ["nginx", "-g", "daemon off;"] 

Build image dilakukan menggunakan: 

student@student:~/docker-lab/custom-web$ docker build -t pens-web:1.0 . 

Container dijalankan menggunakan: 

docker run -d --name pens-app -p 9090:80 pens-web:1.0 

Halaman custom berhasil diakses melalui browser pada: http://localhost:9090 

## **Docker Lab PENS** 

Container berhasil berjalan! 

- Hostname: 10.252.108.77 

- Server: Nginx on Docker 

- Praktikum: Modul 1 - Instalasi Docker 

## **JAWABAN POST-LAB** 

## **1. Bandingkan output docker image history nginx dengan docker image history pens-web:1.0. Layer mana saja yang di-share?** 

Image pens-web:1.0 menggunakan nginx:1.26-alpine sebagai base image sehingga layer dasar dari nginx tetap digunakan bersama (shared layer). Layer tambahan pada pens-web:1.0 berasal dari instruksi seperti RUN rm -rf, COPY index.html, dan metadata LABEL. Dengan sistem layered filesystem, Docker tidak perlu menggandakan seluruh image sehingga penggunaan storage menjadi lebih efisien. 

## **2. Apa yang terjadi pada data di dalam container setelah container dihapus dengan** 

## **docker rm? Bagaimana solusinya?** 

Data yang berada di dalam writable layer container akan ikut terhapus ketika container dihapus menggunakan docker rm. Untuk menyimpan data secara permanen digunakan Docker Volume atau bind mount sehingga data tetap tersimpan walaupun container dihapus. Contoh penggunaan volume: docker run -v mydata:/data ubuntu 

## **3. Jelaskan perbedaan antara EXPOSE di Dockerfile dan flag -p pada docker run. Apakah EXPOSE cukup untuk membuat port dapat diakses dari host?** 

EXPOSE hanya memberikan informasi bahwa container menggunakan port tertentu. Sementara itu, flag -p digunakan untuk melakukan port mapping antara host dan container. Contoh: docker run -p 8080:80 nginx 

Tanpa -p, port container tidak dapat diakses langsung dari host. Jadi EXPOSE saja tidak cukup untuk membuka akses port. 

## **4. Mengapa menggunakan tag spesifik (misal nginx:1.26) lebih baik daripada nginx:latest untuk production?** 

Tag spesifik memberikan konsistensi versi aplikasi sehingga environment production menjadi lebih stabil dan dapat diprediksi. Jika menggunakan latest, versi image dapat berubah sewaktu-waktu sehingga berpotensi menyebabkan bug atau incompatibility pada aplikasi. 

## **5. Berapa ukuran image alpine:3.20 dibanding ubuntu:22.04? Apa trade-off** 

## **menggunakan Alpine?** 

Image alpine:3.20 berukuran sekitar 7 MB sedangkan ubuntu:22.04 dapat mencapai ratusan MB. Alpine lebih ringan dan cepat diunduh sehingga cocok untuk container minimalis. Namun trade-off Alpine adalah: 

- Package bawaan lebih sedikit 

- Beberapa library tidak kompatibel 

- Debugging lebih sulit 

- Menggunakan musl libc, bukan glibc 

Sedangkan Ubuntu lebih lengkap dan kompatibel untuk berbagai aplikasi. 
