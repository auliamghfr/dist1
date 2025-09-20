# REST

## Pengertian

**REST (Representational State Transfer)** adalah arsitektur komunikasi berbasis HTTP yang menggunakan resource sebagai objek utama.
REST memanfaatkan metode HTTP (GET, POST, PUT, DELETE) untuk operasi terhadap resource.
Biasanya digunakan dalam pengembangan **Web API** karena sederhana, ringan, dan mudah diintegrasikan.

---

## Langkah Praktik

### 1. Menjalankan Container REST
<p align="center">
  <img src="https://imgur.com/dTfCCMZ.png" alt="Rest Docker Compose" width="300">
</p>

```bash
docker compose -f compose/rest.yml up -d
```

Output akan menampilkan container:

* `rest-server`
* `rest-client`

---

### 2. Mengecek Interface dan IP

```bash
ip a
```
<p align="center">
  <img src="https://imgur.com/7SfhP7b.png" alt="Rest Docker Compose" width="300">
</p>

Perintah ini digunakan untuk melihat interface dan IP address dari container.
Bridge network (misalnya `br-d44fc16962b6`) digunakan untuk komunikasi antar container.

---

### 3. Monitoring dengan tcpdump

Gunakan `tcpdump` untuk menangkap paket REST:

```bash
sudo tcpdump -nvi <bridge_name> -w rest.pcap
```
<p align="center">
  <img src="https://imgur.com/2M3R8as.png" alt="Rest Docker Compose" width="300">
</p>

File `.pcap` ini bisa dianalisis di **Wireshark** atau **VSC-Webshark** untuk melihat request/response HTTP.


---

### 4. Menjalankan Server

Server dijalankan dengan Flask:

```bash
docker compose -f compose/rest.yml exec rest-server python server.py
```
<p align="center">
  <img src="https://imgur.com/fTfZvPV.png" alt="Rest Docker Compose" width="300">
</p>

Contoh output:

```
* Serving Flask app 'server'
* Debug mode: on
* Running on all addresses (0.0.0.0)
* Running on http://172.20.0.2:5151
```

Di dalam `server.py` terdapat baris:

```python
app.run(host="0.0.0.0", port=5151, debug=True)
```

* `0.0.0.0` → Flask server mendengarkan semua interface.
* `port=5151` → server dijalankan pada port 5151.
* Debug mode aktif karena `debug=True`.

Log `GET /add?a=2&b=3` menunjukkan bahwa client memanggil endpoint `/add` dengan parameter query.

---

### 5. Menjalankan Client

Client melakukan request ke server REST:
<p align="center">
  <img src="https://imgur.com/WZYKhbu.png" alt="Rest Docker Compose" width="300">
</p>

```bash
docker compose -f compose/rest.yml exec rest-client python client.py --op both -a 2 -b 3
```

Contoh output:

```
add(2,3) = 5
mul(2,3) = 6
```

Isi `client.py` berisi:

```python
resp_add = requests.get(f"http://{server_ip}:5151/add", params={"a": a, "b": b})
resp_mul = requests.get(f"http://{server_ip}:5151/mul", params={"a": a, "b": b})
```

Client melakukan GET request ke `/add` dan `/mul`.

Server (`server.py`) punya route:

```python
@app.route('/add')
def add():
    a = int(request.args['a'])
    b = int(request.args['b'])
    return str(a + b)
```

Jadi `2 + 3 = 5`.
Endpoint `/mul` mengembalikan hasil `2 * 3 = 6`.

Karena itu output di client sama persis dengan perhitungan dari server.

---

### 6. Monitoring dengan Wireshark

Pada Wireshark, terlihat paket HTTP GET dengan query string (`/add?a=2&b=3` dan `/mul?a=2&b=3`).
Response `200 OK` berisi hasil operasi dari server (`5` dan `6`).

<p align="center">
  <img src="https://imgur.com/JYWhTXe.png" alt="Rest Docker Compose" width="700">
</p>

Ini membuktikan bahwa komunikasi berbasis REST berjalan sesuai desain: **client → request HTTP → server → response hasil**.

---

### 7. Menghentikan dan Membersihkan Container

```bash
docker compose -f compose/rest.yml down
```
<p align="center">
  <img src="https://imgur.com/Cu5BBct.png" alt="Rest Docker Compose" width="300">
</p>

Output menunjukkan container `rest-server`, `rest-client`, serta network Docker telah dihentikan.

---

## Struktur Folder Terkait

* `compose/rest.yml` → konfigurasi docker compose untuk REST server dan client.
* `REST/server.py` → script server Flask dengan endpoint `/add` dan `/mul`.
* `REST/client.py` → script client yang melakukan HTTP request.

---

## Troubleshooting

* **Port sudah dipakai** → pastikan tidak ada service lain di port `5151`.
* **Client tidak dapat mengakses server** → cek apakah IP server sesuai (`172.20.0.x`) dan bridge Docker benar.
* **Wireshark tidak ada data** → pastikan capture dilakukan di bridge network yang tepat.
