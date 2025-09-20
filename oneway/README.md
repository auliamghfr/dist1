# UDP One-Way

## Pengertian

**UDP (User Datagram Protocol)** adalah protokol komunikasi connectionless yang bekerja dengan mengirimkan datagram tanpa perlu membangun koneksi terlebih dahulu.
UDP lebih ringan dibanding TCP karena tidak memiliki mekanisme handshaking, sehingga cocok digunakan pada aplikasi real-time seperti streaming, VoIP, atau sensor IoT.

---

## Langkah Praktik

### 1. Menjalankan Container UDP

```bash
docker compose -f compose/udp.yml up -d
```

Output akan menampilkan proses build dan menjalankan container:

* `udp-server`
* `udp-client`

---

### 2. Mengecek Interface dan IP

```bash
ip a
```

Output menampilkan daftar interface jaringan Docker beserta alamat IP.
Bridge network (misalnya `br-862fa...`) digunakan untuk komunikasi antar container.

---

### 3. Monitoring Lalu Lintas dengan tcpdump

Gunakan `tcpdump` untuk melakukan capture paket UDP:

```bash
sudo tcpdump -nvi <bridge_name> -w udp.pcap
```

Output akan menunjukkan jumlah paket yang berhasil ditangkap.
File `.pcap` ini bisa dianalisis dengan **Wireshark** atau **VSC-Webshark**.

---

### 4. Menjalankan Server

Jalankan server UDP:

```bash
docker compose -f compose/udp.yml exec udp-server python serverUDP.py
```

Contoh output:

```
UDP server up and listening on ('0.0.0.0', 12345)
```

Di dalam `serverUDP.py`, terdapat baris:

```python
server_socket.bind(("0.0.0.0", 12345))
```

* `0.0.0.0` artinya server mendengarkan pada semua interface.
* `12345` adalah port yang ditentukan.

Sehingga saat server dijalankan, ia menampilkan bahwa socket UDP siap menerima data pada alamat `('0.0.0.0', 12345)`.

---

### 5. Menjalankan Client

Client mengirimkan pesan ke server dan menerima balasan.

```bash
docker compose -f compose/udp.yml exec udp-client python clientUDP.py
```

Contoh output:

```
Received from server: Hello, ('172.19.0.3', 50944). You said: Hello, UDP server!
```

Di dalam `clientUDP.py` biasanya ada:

```python
client_socket.sendto(message.encode(), (server_ip, 12345))
data, addr = client_socket.recvfrom(1024)
print(f"Received from server: {data.decode()} from {addr}")
```

* Client mengirim string `"Hello, UDP server!"` ke IP server (`172.19.0.x`) dan port `12345`.
* Server menerima pesan, lalu membalas dengan format:

```python
"Hello, {addr}. You said: {message}"
```

Output menunjukkan IP client (`172.19.0.3`) dan port sumber (`50944`) yang dipilih otomatis oleh OS.
Itulah kenapa setiap run port client bisa berbeda, tapi pesan balasannya konsisten.

---

### 6. Menghentikan dan Membersihkan Container

```bash
docker compose -f compose/udp.yml down
```

Output akan menampilkan bahwa container `udp-server`, `udp-client`, serta network Docker telah dihentikan dan dihapus.

---

## Struktur Folder Terkait

* `compose/udp.yml` → konfigurasi docker compose untuk server dan client.
* `UDP/serverUDP.py` → script server UDP.
* `UDP/clientUDP.py` → script client UDP.

---

## Troubleshooting

* **Address already in use** → pastikan port `12345` tidak digunakan oleh proses lain.
* **Client tidak menerima balasan** → periksa apakah server sudah berjalan dan menggunakan IP yang benar.
* **Paket tidak tertangkap di tcpdump** → gunakan nama bridge network yang sesuai untuk capture.
