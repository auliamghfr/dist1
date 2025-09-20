# SOAP

## Pengertian

**SOAP (Simple Object Access Protocol)** adalah protokol komunikasi berbasis **XML** yang digunakan untuk pertukaran data antar aplikasi melalui jaringan.
Komunikasi dilakukan dengan format **SOAP Request** dan **SOAP Response** yang dibungkus dalam XML, sehingga terstandarisasi, aman, dan lintas platform.

---

## Langkah Praktik

### 1. Menjalankan SOAP Server dan Client
<p align="center">
  <img src="https://imgur.com/SP7Z9OT.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
docker compose -f compose/soap.yml up -d
```

Output akan menampilkan proses build image dan menjalankan container:

* `soap-server`
* `soap-client`

---

### 2. Menjalankan Server

SOAP server dijalankan menggunakan perintah berikut:

<p align="center">
  <img src="https://imgur.com/nOkc4wz.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
docker compose -f compose/soap.yml exec soap-server python server.py
```

Contoh output:

```
SOAP server listening on http://0.0.0.0:8000
```

---

### 3. Menjalankan Client

SOAP client digunakan untuk mengirim request ke server.

<p align="center">
  <img src="https://imgur.com/95qStMr.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
docker compose -f compose/soap.yml exec soap-client python client.py
```

Contoh output:

```
Hasil penjumlahan dari server SOAP: 15
```

---

### 4. Melihat Lalu Lintas Jaringan SOAP

Gunakan `tcpdump` untuk memverifikasi lalu lintas SOAP di jaringan:

<p align="center">
  <img src="https://imgur.com/tkfmuLI.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
sudo tcpdump -nvi <bridge_name> -w soap.pcap
```

Output akan menampilkan paket-paket yang ditransfer melalui protokol HTTP/XML SOAP.

---

### 5. Mengecek Interface dan IP

```bash
ip a
```

Perintah ini menampilkan daftar interface jaringan beserta alamat IP yang digunakan container.
Bridge network (misalnya `br-xxxxxxxx`) digunakan untuk komunikasi antar container.

---

### 6. Monitoring Realtime

Monitoring trafik dapat dilakukan dengan tools wireshark

<p align="center">
  <img src="https://imgur.com/rccdwU4.png" alt="SOAP Docker Compose" width="700">
</p>

Untuk memvisualisasikan paket SOAP yang melewati jaringan.

---

### 7. Menghentikan dan Membersihkan Container

<p align="center">
  <img src="https://imgur.com/jzMHqsx.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
docker compose -f compose/soap.yml down
```

Output akan menunjukkan bahwa container server, client, dan network Docker telah dihapus.

---

## Struktur Folder Terkait

* `compose/soap.yml` → konfigurasi docker compose untuk server dan client.
* `SOAP/server.py` → script server SOAP.
* `SOAP/client.py` → script client SOAP.
* `SOAP/requirements.txt` → dependensi Python untuk SOAP.
* `SOAP/Dockerfile` → environment untuk SOAP server & client.

---

## Troubleshooting

* **Server tidak merespons** → pastikan container server berjalan dengan benar.
* **Client gagal koneksi** → periksa apakah alamat dan port server sesuai (`http://soap-server:8000`).
* **Paket tidak terlihat di tcpdump** → pastikan menggunakan interface bridge network yang benar.
