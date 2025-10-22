# Jarkom-Modul-2-2025-K21

Member :
1. Danuja Prasasta Bastu (5027241037)
2. Naila Cahyarani Idelia (5027241063)

<div align=center>

# Soal Modul 2

</div>

- [Question 1](#question-1)
- [Question 2](#question-2)
- [Question 3](#question-3)
- [Question 4](#question-4)
- [Question 5](#question-5)
- [Question 6](#question-6)
- [Question 7](#question-7)
- [Question 8](#question-8)
- [Question 9](#question-9)
- [Question 10](#question-10)

## Question 1

> Di tepi Beleriand yang porak-poranda, Eonwe merentangkan tiga jalur: Barat untuk Earendil dan Elwing, Timur untuk Círdan, Elrond, Maglor, serta pelabuhan DMZ bagi Sirion, Tirion, Valmar, Lindon, Vingilot. Tetapkan alamat dan default gateway tiap tokoh sesuai glosarium yang sudah diberikan.

<img width="1430" height="793" alt="Screenshot 2025-10-12 130334" src="https://github.com/user-attachments/assets/c9912191-9233-4e0a-804e-da40864193d8" />

Dan lakukan Edit Network Configuration.

## Question 2

> Angin dari luar mulai berhembus ketika Eonwe membuka jalan ke awan NAT. Pastikan jalur WAN di router aktif dan NAT meneruskan trafik keluar bagi seluruh alamat internal sehingga host di dalam dapat mencapai layanan di luar menggunakan IP address.

Agar host internal dapat akses internet maka lakukan

```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s [IP Prefix].0.0/16
```
Dan lakukan

```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

## Question 3

> Kabar dari Barat menyapa Timur. Pastikan kelima klien dapat saling berkomunikasi lintas jalur (routing internal via Eonwe berfungsi), lalu pastikan setiap host non-router menambahkan resolver 192.168.122.1 saat interfacenya aktif agar akses paket dari internet tersedia sejak awal.

Tambahkan configuration `echo "nameserver 192.168.122.1" > /etc/resolv.conf`.

<img width="744" height="748" alt="Screenshot 2025-10-13 175550" src="https://github.com/user-attachments/assets/c330013d-1b3a-4de9-8b52-4e611661c3e1" />

Lalu tes untuk memastikan agar semua klien bisa saling berkomunikasi melalui Eonwe menggunakan ping.

<img width="854" height="209" alt="Screenshot 2025-10-13 175422" src="https://github.com/user-attachments/assets/7a392a09-94b2-421a-a5f5-4349522deb8f" />
<img width="851" height="203" alt="Screenshot 2025-10-13 175511" src="https://github.com/user-attachments/assets/7e2b6b48-6d10-4581-8c75-d38741f9907c" />

## Question 4

> Para penjaga nama naik ke menara, di Tirion (ns1/master) bangun zona <xxxx>.com sebagai authoritative dengan SOA yang menunjuk ke ns1.<xxxx>.com dan catatan NS untuk ns1.<xxxx>.com dan ns2.<xxxx>.com. Buat A record untuk ns1.<xxxx>.com dan ns2.<xxxx>.com yang mengarah ke alamat Tirion dan Valmar sesuai glosarium, serta A record apex <xxxx>.com yang mengarah ke alamat Sirion (front door), aktifkan notify dan allow-transfer ke Valmar, set forwarders ke 192.168.122.1. Di Valmar (ns2/slave) tarik zona <xxxx>.com dari Tirion dan pastikan menjawab authoritative. pada seluruh host non-router ubah urutan resolver menjadi IP dari ns1.<xxxx>.com → ns2.<xxxx>.com → 192.168.122.1. Verifikasi query ke apex dan hostname layanan dalam zona dijawab melalui ns1/ns2.

- Node Tirion

```bash
apt-get update && apt-get install bind9 -y
```

```bash
nano /etc/bind/named.conf.options

options {
        directory "/var/cache/bind";
        forwarders { 192.168.122.1; };
        allow-query { any; };
};  

```

```bash
nano /etc/bind/named.conf.local

zone "K21.com" { type master; file "/etc/bind/db.K21.com"; allow-transfer { 10.74.3.3; }; };
```

```bash
nano /etc/bind/db.K21.com

$TTL    604800
@   IN  SOA ns1.K21.com. admin.K21.com. (
            1           ; Serial
            604800      ; Refresh
            86400       ; Retry
            2419200     ; Expire
            604800 )    ; Negative Cache TTL
;
@   IN  NS  ns1.K21.com.
@   IN  NS  ns2.K21.com.

@   IN  A   10.74.3.6    ; Sirion
ns1 IN  A   10.74.3.2    ; Tirion
ns2 IN  A   10.74.3.3    ; Valmar

```
lalu jalankan command ini 

```bash
named-checkconf

named-checkzone K21.com /etc/bind/db.K21.com

named

dig @localhost K21.com
```

- Node Valmar

```bash
apt-get update && apt-get install bind9 -y
```

```bash
nano /etc/bind/named.conf.options

options {
    directory "/var/cache/bind";
    
    forwarders {
        192.168.122.1;
    };
};
```

```bash
nano /etc/bind/named.conf.local

zone "K21.com" {
    type slave;
    file "db.K21.com";
    masters { 10.74.3.2; }; // IP Tirion
};
```
```bash 
named-checkconf

service named restart
```
jalankan ini di node client lainnya selain eonwe
```bash
echo "nameserver 10.74.3.2" > /etc/resolv.conf
echo "nameserver 10.74.3.3" >> /etc/resolv.conf
echo "nameserver 192.168.122.1" >> /etc/resolv.conf
```
Untuk memverivikasi kita menggunakan `dig K21.com` begitu juga dengan `dig ns1.K21.com`

<img width="896" height="586" alt="image" src="https://github.com/user-attachments/assets/ca1cd828-0263-43cf-af7e-6c8d76a52683" />

## Question 5

> “Nama memberi arah,” kata Eonwe. Namai semua tokoh (hostname) sesuai glosarium, eonwe, earendil, elwing, cirdan, elrond, maglor, sirion, tirion, valmar, lindon, vingilot, dan verifikasi bahwa setiap host mengenali dan menggunakan hostname tersebut secara system-wide. Buat setiap domain untuk masing masing node sesuai dengan namanya (contoh: eru.<xxxx>.com) dan assign IP masing-masing juga. Lakukan pengecualian untuk node yang bertanggung jawab atas ns1 dan ns2

Lakukan dengan semua client, ubah namanya menjadi nama client

```bash
echo "eonwe" > /etc/hostname
hostname -F /etc/hostname
```
lalu kembali ke nodes tirion dan tambahkan di file db.K21.com
```bash
nano /etc/bind/db.K21.com

// Tambahkan ini

eonwe.K21.com.      IN      A       10.74.1.1
earendil.K21.com.   IN      A       10.74.1.2
elwing.K21.com.     IN      A       10.74.1.3
cirdan.K21.com.     IN      A       10.74.2.2
elrond.K21.com.     IN      A       10.74.2.3
maglor.K21.com.     IN      A       10.74.2.4
sirion.K21.com.     IN      A       10.74.3.6
lindon.K21.com.     IN      A       10.74.3.5
vingilot.K21.com.   IN      A       10.74.3.4
```

```bash
named-checkzone K21.com /etc/bind/db.K21.com
```

```bash
service named restart
```

Lalu cek dengan hostname `dig earendil.K21.com` & `dig sirion.K21.com`

<img width="895" height="363" alt="image" src="https://github.com/user-attachments/assets/2525d91d-50f5-4dc2-b0ee-5005e42e6d77" />

## Question 6

> Lonceng Valmar berdentang mengikuti irama Tirion. Pastikan zone transfer berjalan, Pastikan Valmar (ns2) telah menerima salinan zona terbaru dari Tirion (ns1). Nilai serial SOA di keduanya harus sama

Untuk Tirion

```bash
dig @10.74.3.3 K21.com SOA
```
<img width="895" height="386" alt="image" src="https://github.com/user-attachments/assets/bcbf7379-efbe-4992-8e72-6f442939fb8c" />

Untuk Valmar

```bash
dig @10.74.3.2 K21.com SOA
```
<img width="896" height="383" alt="image" src="https://github.com/user-attachments/assets/b05c13f0-29ec-4fd3-877c-241b0c27a73f" />

Perhatikan bagian **ANSWER SECTION**, bagian pojok kanan ada serial number, dan disini serial numbernya sama, membuktikan bahwa zone transfer berjalan dengan benar. Valmar telah menerima salinan zona terbaru dari Tirion.

## Question 7

> Peta kota dan pelabuhan dilukis. Sirion sebagai gerbang, Lindon sebagai web statis, Vingilot sebagai web dinamis. Tambahkan pada zona <xxxx>.com A record untuk sirion.<xxxx>.com (IP Sirion), lindon.<xxxx>.com (IP Lindon), dan vingilot.<xxxx>.com (IP Vingilot). Tetapkan CNAME :
 - www.<xxxx>.com → sirion.<xxxx>.com, 
 - static.<xxxx>.com → lindon.<xxxx>.com, dan 
 - app.<xxxx>.com → vingilot.<xxxx>.com.
Verifikasi dari dua klien berbeda bahwa seluruh hostname tersebut ter-resolve ke tujuan yang benar dan konsisten.

Node Tirion

```bash
nano /etc/bind/db.K21.com

// Lalu tambahkan bagian nomor serialnya +1 (awalnya 1 → 2)
2           ; Serial

// Tambahkan juga
;
; CNAME records - NOMOR 7
;
www     IN      CNAME   sirion.K21.com.
static  IN      CNAME   lindon.K21.com.
app     IN      CNAME   vingilot.K21.com.
```

Cara cek nya buat cadangan lakuin `apt-get update && apt-get install bind9 -y`

```bash
named-checkconf

named-checkzone K21.com /etc/bind/db.K21.com

named
```

Test dari client lain:

Edit server nya dulu pakai

```bash
nano /etc/resolv.conf

nameserver 10.74.3.2    # Tirion (ns1) -> ini yang ditambah
nameserver 10.74.3.3    # Valmar (ns2) -> ini yang ditambah
nameserver 192.168.122.1
```
Dari Elrond:
```bash
dig www.K21.com
dig static.K21.com
dig app.K21.com
```

Dari Earendil:
```bash
dig www.K21.com
dig static.K21.com
dig app.K21.com
```

<img width="1919" height="154" alt="Screenshot 2025-10-20 140235" src="https://github.com/user-attachments/assets/237b3c1a-d3e5-48a8-af94-db9c3fcc7fb5" />

## Question 8

> Setiap jejak harus bisa diikuti. Di Tirion (ns1) deklarasikan satu reverse zone untuk segmen DMZ tempat Sirion, Lindon, Vingilot berada. Di Valmar (ns2) tarik reverse zone tersebut sebagai slave, isi PTR untuk ketiga hostname itu agar pencarian balik IP address mengembalikan hostname yang benar, lalu pastikan query reverse untuk alamat Sirion, Lindon, Vingilot dijawab authoritative.

Buat reverse zone untuk DMZ pada Tirion dan tarik sebagai slave di Valmar.

DI TIRION (ns1/master):

Buat reverse zone file:

```bash
nano /etc/bind/db.10.74.3
```

Isi dengan:

```bash
;
; BIND reverse data file for DMZ 10.74.3.0/24
;
$TTL    604800
@       IN      SOA     ns1.K21.com. admin.K21.com. (
                        1         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL

;
; Name servers
;
@       IN      NS      ns1.K21.com.
@       IN      NS      ns2.K21.com.

;
; PTR records - IP to hostname mapping
;
6       IN      PTR     sirion.K21.com.
5       IN      PTR     lindon.K21.com.
4       IN      PTR     vingilot.K21.com.
2       IN      PTR     tirion.K21.com.
3       IN      PTR     valmar.K21.com.
```

Edit /etc/bind/named.conf.local:

```bash
nano /etc/bind/named.conf.local
```

Tambahkan:

```bash
// Reverse zone untuk DMZ
zone "3.74.10.in-addr.arpa" {
    type master;
    file "/etc/bind/db.10.74.3";
    allow-transfer { 10.74.3.3; }; // Valmar
    notify yes;
};
```

DI VALMAR (ns2/slave):

Edit /etc/bind/named.conf.local:

```bash
nano /etc/bind/named.conf.local
```

Tambahkan:

```bash
// Reverse zone slave
zone "3.74.10.in-addr.arpa" {
    type slave;
    file "db.10.74.3";
    masters { 10.74.3.2; }; // Tirion
};
```

Kita coba test di tirion:

```bash
named-checkzone 3.74.10.in-addr.arpa /etc/bind/db.10.74.3

named-checkconf

named
```

Kita coba test di valmar:

```bash
named-checkconf

named
```

Untuk cek  PTR menggunakan 

```bash
dig -x 10.74.3.6 @10.74.3.2    # Dari Tirion
dig -x 10.74.3.6 @10.74.3.3    # Dari Valmar
```

<img width="1919" height="616" alt="image" src="https://github.com/user-attachments/assets/620b3a9d-72fb-41da-aef5-7dfd05144aab" />


## Question 9

> Lampion Lindon dinyalakan. Jalankan web statis pada hostname static.<xxxx>.com dan buka folder arsip /annals/ dengan autoindex (directory listing) sehingga isinya dapat ditelusuri. Akses harus dilakukan melalui hostname, bukan IP.

Node Lindon

```bash
apt-get update && apt-get install nginx -y
```

```bash
mkdir -p /var/www/lindon/annals
```

```bash
cat > /var/www/lindon/annals/index.html
```

```bash
systemctl enable --now nginx
```
Di client: `curl -I http://static.K21.com/annals/`

## Question 10

> Vingilot mengisahkan cerita dinamis. Jalankan web dinamis (PHP-FPM) pada hostname app.<xxxx>.com dengan beranda dan halaman about, serta terapkan rewrite sehingga /about berfungsi tanpa akhiran .php. Akses harus dilakukan melalui hostname.

Node Vingilot

```bash
apt-get update && apt-get install nginx -y php8.4-fpm php8.4-cli
```

```bash
systemctl enable --now nginx php8.4-fpm
```

Untuk memverivikasi kita menggunakan `curl -I http://app.K21.com/` & `curl -I http://app.K21.com/about`
