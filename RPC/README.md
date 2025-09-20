# RPC

## Pengertian

**Remote Procedure Call (RPC)** adalah mekanisme komunikasi di mana sebuah program (client) dapat memanggil prosedur yang dijalankan di komputer lain (server) seolah-olah prosedur tersebut dijalankan secara lokal.
RPC memungkinkan abstraksi pemanggilan fungsi jarak jauh sehingga developer tidak perlu mengelola detail komunikasi jaringan.

---

## Langkah Praktik

### 1. Menjalankan Container RPC

<p align="center">
  <img src="https://imgur.com/UsV4eXh.png" alt="RPC Docker Compose" width="300">
</p>

```bash
docker compose -f compose/rpc.yml up -d
```

Output menampilkan bahwa container berhasil dijalankan:

* `rpc-server`
* `rpc-client`

---

### 2. Mengecek Interface dan IP

```bash
ip a
```
<p align="center">
  <img src="https://imgur.com/8Wld5XN.png" alt="RPC Docker Compose" width="300">
</p>

Perintah ini digunakan untuk melihat interface jaringan dan IP yang digunakan oleh container RPC.
Bridge network (misalnya `br-c7bca5cb5271`) digunakan untuk komunikasi antar container.

---

### 3. Menjalankan Server

Server RPC dijalankan dengan perintah:

```bash
docker compose -f compose/rpc.yml exec rpc-server python rpcserver.py
```
<p align="center">
  <img src="https://imgur.com/x6ikgrv.png" alt="RPC Docker Compose" width="300">
</p>

Contoh output:

```
Starting JSON-RPC server on port 4000...
172.19.0.3 - - [19/Sep/2025 14:21:36] "POST / HTTP/1.1" 200 -
172.19.0.3 - - [19/Sep/2025 14:21:39] "POST / HTTP/1.1" 200 -
```

---

### 4. Menjalankan Client

Client RPC memanggil fungsi yang disediakan oleh server.

<p align="center">
  <img src="https://imgur.com/7iVWBoM.png" alt="RPC Docker Compose" width="300">
</p>

```bash
docker compose -f compose/rpc.yml exec rpc-client python rpcclient.py
```

Contoh output:

```
Result of add: 5.0
Result of multiply: 50
```

---

### 5. Monitoring Lalu Lintas Jaringan

Gunakan `tcpdump` untuk melihat komunikasi RPC:

```bash
sudo tcpdump -nvi <bridge_name> -w rpc.pcap
```
<p align="center">
  <img src="https://imgur.com/6hLitEK.png" alt="RPC Docker Compose" width="300">
</p>

Output menunjukkan jumlah paket yang ditangkap. File `rpc.pcap` dapat dianalisis menggunakan **Wireshark** atau **VSC-Webshark**.

<p align="center">
  <img src="https://imgur.com/VIybC3y.png" alt="RPC Docker Compose" width="700">
</p>
---

### 6. Menghentikan dan Membersihkan Container

```bash
docker compose -f compose/rpc.yml down
```

Output akan menunjukkan container `rpc-server`, `rpc-client`, serta network Docker telah dihapus.
<p align="center">
  <img src="https://imgur.com/5kcXBfV.png" alt="RPC Docker Compose" width="300">
</p>

---

## Struktur Folder Terkait

* `compose/rpc.yml` → konfigurasi docker compose untuk server dan client.
* `RPC/rpcserver.py` → script server RPC.
* `RPC/rpcclient.py` → script client RPC.
* `RPC/requirements.txt` → dependensi Python untuk RPC.

---

## Troubleshooting

* **Client tidak dapat konek ke server** → pastikan server sudah berjalan di port yang benar (4000).
* **Error JSON decode** → periksa apakah request dari client sesuai format JSON-RPC.
* **Paket tidak terlihat di tcpdump** → pastikan menggunakan interface bridge yang sesuai un
