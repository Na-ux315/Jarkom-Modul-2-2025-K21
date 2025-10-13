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
