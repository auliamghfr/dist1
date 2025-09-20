# UPCALL

## Pengertian

**Upcall** adalah mekanisme komunikasi di mana server dapat memanggil kembali fungsi pada client (callback).
Dengan pola ini, server tidak hanya menerima request dari client, tetapi juga dapat langsung mengirim notifikasi atau eksekusi fungsi ke client.

---

## Langkah Praktik

### 1. Menjalankan Container Upcall

<p align="center">
  <img src="https://imgur.com/UUz4XhK.png" alt="Upcall Docker Compose" width="300">
</p>

```bash
docker compose -f compose/upcall.yml up -d
```

Output akan menampilkan proses build dan menjalankan container:

* `upcall-server`
* `upcall-client`

---

### 2. Menjalankan Server

Jalankan server dengan perintah:

```bash
docker compose -f compose/upcall.yml exec upcall-server python servercall.py
```
<p align="center">
  <img src="https://imgur.com/uOPAiah.png" alt="Upcall Docker Compose" width="300">
</p>

Pastikan tidak ada server lain berjalan pada port yang sama sebelum menjalankan ulang.

---

### 3. Menjalankan Client

Client akan mengirim pesan ke server dan menerima upcall dari server.

<p align="center">
  <img src="https://imgur.com/G9DFUUk.png" alt="Upcall Docker Compose" width="300">
</p>

```bash
docker compose -f compose/upcall.yml exec upcall-client python clientcall.py
```

Contoh output:

```
Enter message: haii
Received upcall from server: Upcall event: Processing haii

Enter another message: halo
Received upcall from server: Upcall event: Processing halo
```

Server akan menampilkan log setiap kali melakukan upcall ke client.

---

### 4. Mengecek Interface dan IP

<p align="center">
  <img src="https://imgur.com/NVQcQXC.png" alt="Upcall Docker Compose" width="300">
</p>

```bash
ip a
```

Output akan menampilkan daftar interface jaringan Docker beserta alamat IP.
Bridge network (misalnya `br-34edb51ffbb`) digunakan untuk komunikasi antar container.

---

### 5. Monitoring Lalu Lintas Jaringan

Gunakan `tcpdump` untuk memverifikasi komunikasi antara server dan client:

<p align="center">
  <img src="https://imgur.com/mzO4Van.png" alt="Upcall Docker Compose" width="300">
</p>

```bash
sudo tcpdump -nvi <bridge_name> -w upcall.pcap
```
<p align="center">
  <img src="https://imgur.com/k4U39PV.png" alt="Upcall Docker Compose" width="700">
</p>

File `upcall.pcap` dapat dianalisis menggunakan **Wireshark** atau **VSC-Webshark** untuk melihat detail paket komunikasi.


---

### 6. Menghentikan dan Membersihkan Container

<p align="center">
  <img src="https://imgur.com/ZfxH6U7.png" alt="Upcall Docker Compose" width="300">
</p>

```bash
docker compose -f compose/upcall.yml down
```

Output akan menampilkan bahwa container server, client, dan network Docker telah dihapus.

---

## Struktur Folder Terkait

* `compose/upcall.yml` → konfigurasi docker compose untuk server dan client.
* `Upcall/servercall.py` → script server dengan upcall.
* `Upcall/clientcall.py` → script client yang menerima upcall.

---

## Troubleshooting

* **Address already in use (Errno 98)** → hentikan proses lain yang memakai port sama sebelum menjalankan server.
* **Client tidak menerima upcall** → pastikan client aktif dan berada dalam jaringan Docker bridge yang sama.
* **Paket tidak muncul di tcpdump** → gunakan nama bridge network yang sesuai saat capture.
