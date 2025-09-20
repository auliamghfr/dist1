# REQRESP

## Pengertian

**Request-Response (REQRESP)** adalah pola komunikasi client-server klasik, di mana **client** mengirim permintaan (request) dan **server** merespons (response).
Model ini umum digunakan pada sistem terdistribusi untuk memastikan adanya interaksi dua arah yang sederhana dan jelas.

---

## Langkah Praktik

### 1. Menjalankan Server dan Client

<p align="center">
  <img src="https://imgur.com/jicV8LK.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
docker compose -f compose/reqresp.yml up -d
```

Output menunjukkan container berhasil dibangun dan dijalankan:

* `reqresp-server`
* `reqresp-client`

---

### 2. Menjalankan Server

Server mendengarkan koneksi dari client.

<p align="center">
  <img src="https://imgur.com/wh78ZN9.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
docker compose -f compose/reqresp.yml exec reqresp-server python server.py
```

Contoh output:

```
Server listening on 0.0.0.0:2222
Connection from: ('172.19.0.3', 51778)
Received from client: apa kabar?
```

---

### 3. Menjalankan Client

Client mengirimkan pesan ke server dan menerima respons.

<p align="center">
  <img src="https://imgur.com/0kLzdcc.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
docker compose -f compose/reqresp.yml exec reqresp-client python client.py
```

<p align="center">
  <img src="https://imgur.com/ZCrOyzq.png" alt="SOAP Docker Compose" width="300">
</p>

Contoh output:

```
Enter message: halo
<class 'bytes'>
Received from server: Echo: halo

Enter another message: apa kabar?
<class 'bytes'>
Received from server: Echo: apa kabar?
```

---

### 4. Melihat Lalu Lintas Jaringan REQRESP

Gunakan `tcpdump` untuk memverifikasi paket request-response di jaringan:

<p align="center">
  <img src="https://imgur.com/BpyQ3BI.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
sudo tcpdump -nvi <bridge_name> -w reqresp.pcap
```

File `.pcap` kemudian dapat dianalisis menggunakan **Wireshark** atau **VSC-Webshark**.
Output menunjukkan lalu lintas paket antara client dan server.

---

<p align="center">
  <img src="https://imgur.com/1XQNRmd.png" alt="SOAP Docker Compose" width="700">
</p>

### 5. Menghentikan dan Membersihkan Container

<p align="center">
  <img src="https://imgur.com/3BEbNXv.png" alt="SOAP Docker Compose" width="300">
</p>

```bash
docker compose -f compose/reqresp.yml down
```

Output menampilkan bahwa container server, client, dan network Docker telah dihapus.

---

## Struktur Folder Terkait

* `compose/reqresp.yml` → konfigurasi docker compose untuk server dan client.
* `Reqresp/server.py` → script server request-response.
* `Reqresp/client.py` → script client request-response.
* `Reqresp/requirements.txt` → dependensi Python untuk REQRESP.

---

## Troubleshooting

* **Client tidak terhubung ke server** → pastikan server berjalan di port yang benar (2222).
* **Pesan tidak sampai** → cek apakah client dan server ada dalam jaringan Docker bridge yang sama.
* **Paket tidak terlihat di tcpdump** → gunakan nama bridge network yang sesuai saat capture.
