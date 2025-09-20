# ZMQ

## Pengertian

**ZeroMQ (ZMQ)** adalah library messaging yang menyediakan komunikasi asynchronous dengan berbagai pola, seperti:

* **Request-Reply** → komunikasi antara client (req) dan server (rep).
* **Publish-Subscribe** → komunikasi broadcast dari publisher ke banyak subscriber.
* **Pipeline (Push-Pull)** → distribusi pekerjaan antar worker.

ZeroMQ fleksibel dan sering digunakan untuk sistem terdistribusi dengan performa tinggi.

---

## Langkah Praktik

### 1. Menjalankan Container ZMQ

<p align="center">
  <img src="https://imgur.com/i01NZVu.png" alt="ZMQ Docker Compose" width="300">
</p>

```bash
docker compose -f compose/zmq.yml up -d
```

Output akan menampilkan proses build dan menjalankan container:

* `zmq-req`
* `zmq-sub`
* `zmq-pull`
* `zmq-push`

---

### 2. Mengecek Interface dan IP

<p align="center">
  <img src="https://imgur.com/PI4N6wf.png" alt="ZMQ Docker Compose" width="300">
</p>

```bash
ip a
```

Output menampilkan daftar interface jaringan Docker beserta alamat IP.
Bridge network (misalnya `br-34ede5b1fbb`) digunakan untuk komunikasi antar container.

<p align="center">
  <img src="https://imgur.com/UBHOud2.png" alt="ZMQ Docker Compose" width="300">
</p>

---


### 3. Menjalankan Subscriber

Subscriber digunakan untuk menerima pesan dari publisher.

<p align="center">
  <img src="https://imgur.com/Wl4nEkU.png" alt="ZMQ Docker Compose" width="300">
</p>

```bash
docker compose -f compose/zmq.yml exec zmq-sub python subzmq.py
```

Contoh output:

```
Received: WAKTU Fri Sep 19 12:51:11 2025
Received: WAKTU Fri Sep 19 12:51:12 2025
Received: WAKTU Fri Sep 19 12:51:13 2025
...
```
untuk visualisasinya sebagai berikut :
<p align="center">
  <img src="https://imgur.com/vciaWGP.png" alt="ZMQ Docker Compose" width="700">
</p>

---

### 4. Menjalankan Pull Worker

Worker akan menerima pekerjaan dari push task.

<p align="center">
  <img src="https://imgur.com/bG02jju.png" alt="ZMQ Docker Compose" width="300">
</p>

```bash
docker compose -f compose/zmq.yml exec zmq-pull python pullzmq.py
```

Contoh output:

```
Worker 1 received work: 50
```

---

### 5. Monitoring Lalu Lintas Jaringan

Gunakan `tcpdump` untuk melihat komunikasi ZMQ.

<p align="center">
  <img src="https://imgur.com/SbmiucD.png" alt="ZMQ Docker Compose" width="300">
</p>

```bash
sudo tcpdump -nvi <bridge_name> -w zmq.pcap
```

File `zmq.pcap` dapat dianalisis menggunakan **Wireshark** atau **VSC-Webshark** untuk melihat paket-paket yang ditransfer.

<p align="center">
  <img src="https://imgur.com/IVFoj8Z.png" alt="ZMQ Docker Compose" width="700">
</p>

---

### 6. Menghentikan dan Membersihkan Container

<p align="center">
  <img src="https://imgur.com/hm4WQ3h.png" alt="ZMQ Docker Compose" width="300">
</p>

```bash
docker compose -f compose/zmq.yml down
```

Output akan menampilkan container `zmq-req`, `zmq-sub`, `zmq-pull`, dan `zmq-push` telah dihentikan serta network Docker dihapus.

---

## Struktur Folder Terkait

* `compose/zmq.yml` → konfigurasi docker compose untuk ZMQ.
* `ZMQ/subzmq.py` → script subscriber.
* `ZMQ/pullzmq.py` → script worker pull.
* `ZMQ/requirements.txt` → dependensi Python untuk ZeroMQ.

---

## Troubleshooting

* **Subscriber tidak menerima pesan** → pastikan publisher sudah dijalankan sebelum subscriber.
* **Worker tidak menerima task** → pastikan script push berjalan dan worker aktif.
* **Tcpdump tidak menampilkan paket** → gunakan nama bridge network yang sesuai saat capture.
