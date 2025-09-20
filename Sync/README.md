# Lamport Clock

## Pengertian

**Lamport Clock** adalah mekanisme penanda waktu logis (logical clock) dalam sistem terdistribusi.
Tujuannya adalah menjaga **urutan kejadian (event ordering)** antar proses, meskipun mereka berjalan di mesin yang berbeda dan tidak memiliki jam fisik yang sinkron.

Aturan dasar Lamport Clock:

1. Setiap proses memiliki counter lokal.
2. Setiap kali terjadi event internal, counter bertambah 1.
3. Saat mengirim pesan, counter dilampirkan.
4. Saat menerima pesan, counter penerima di-update menjadi `max(counter_lokal, counter_pesan) + 1`.

---

## Langkah Praktik

### 1. Menjalankan Container Lamport

```bash
docker compose -f compose/lamport.yml up -d
```

Output menampilkan container:

* `lamport-server`
* `lamport-client`

---

### 2. Menjalankan Server & Client

Server menerima event dari client dan meng-update logical clock.
Client mengirim event ke server.

```bash
docker compose -f compose/lamport.yml exec lamport-server python server.py
docker compose -f compose/lamport.yml exec lamport-client python client.py
```

Contoh output di server:

```
clock: 4
```

📌 **Kenapa outputnya begitu?**
Di dalam `server.py` ada kode seperti:

```python
clock += 1  # setiap event internal atau menerima pesan
print("clock:", clock)
```

Saat server menerima pesan dari client, ia membandingkan nilai clock lokal dengan timestamp yang dikirim.
Jika timestamp dari client lebih besar, server meng-update clock menjadi `max(local, remote) + 1`.
Hasil akhirnya ditampilkan di terminal (misalnya `clock: 4`).

---

### 3. Monitoring dengan tcpdump

```bash
sudo tcpdump -nvi <bridge_name> -w lamport.pcap
```

Output menunjukkan jumlah paket yang tertangkap.
File `.pcap` bisa dianalisis di **Wireshark** untuk melihat komunikasi antar node.

---

### 4. Analisis dengan Wireshark

Pada Wireshark, terlihat paket komunikasi antar container (`172.20.0.x`) menggunakan protokol TCP.
Paket-paket ini membawa data event dengan timestamp Lamport yang digunakan untuk sinkronisasi logis.

---

### 5. Menghentikan dan Membersihkan Container

```bash
docker compose -f compose/lamport.yml down
```

Output akan menampilkan container `lamport-server`, `lamport-client`, serta network Docker telah dihentikan.

---

## Struktur Folder Terkait

* `compose/lamport.yml` → konfigurasi docker compose untuk server dan client.
* `Lamport/server.py` → script server dengan logical clock.
* `Lamport/client.py` → script client pengirim event.

---

## Penjelasan Teori

* **Output clock meningkat** → karena setiap event (internal/send/receive) memicu `clock += 1`.
* **Clock sinkron antar node** → karena pada saat receive, dilakukan `max(local, remote) + 1`.
* Paket di tcpdump/Wireshark hanya menunjukkan payload komunikasi, tapi nilai logis clock ditentukan oleh algoritma di kode, bukan jam sistem.

---

## Troubleshooting

* **Clock tidak naik** → pastikan ada event (internal atau pesan) yang dipicu.
* **Pesan tidak terkirim** → cek apakah container client dan server ada di bridge network yang sama.
* **Tcpdump kosong** → gunakan nama bridge network yang benar saat capture.
