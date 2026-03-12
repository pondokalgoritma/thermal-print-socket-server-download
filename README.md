# Thermal Print Socket Server 🚀

**Thermal Print Socket Server** adalah aplikasi Desktop yang berfungsi sebagai jembatan (bridge) antara Server/VPS Anda dengan printer thermal lokal. Aplikasi ini memungkinkan Anda melakukan pencetakan struk secara otomatis dan *silent* (tanpa dialog print) menggunakan protokol **Socket.IO**.

## ✨ Fitur Utama
- **Silent Printing**: Cetak struk langsung ke printer thermal tanpa perlu interaksi user.
- **Socket.IO Support**: Integrasi mudah dengan Laravel, Node.js, Python, dsb.
- **Font Selection**: Pilih ukuran font dasar (Standard, Small, atau Small Bold) langsung dari GUI.
- **System Tray**: Berjalan di latar belakang, meminimalkan penggunaan layar.
- **Auto-Discovery**: Mencari printer yang terhubung ke sistem secara otomatis.

---

## 🛠️ Instalasi & Persiapan
1. Download installer dari [GitHub Releases](https://github.com/pondokalgoritma/thermal-print-socket-server-download/releases/latest).
2. Jalankan aplikasi pada komputer yang terhubung langsung ke printer thermal.
3. Buka tab **Settings** untuk mengatur:
   - **Printer Target**: Pilih printer thermal yang aktif.
   - **Socket Port**: Port untuk mendengarkan koneksi (Default: `3006`).
   - **Font Size**: Pilih ukuran font yang sesuai dengan kertas Anda.

> [!IMPORTANT]
> Pastikan Port yang Anda gunakan tidak diblokir oleh Firewall agar Server/VPS bisa melakukan koneksi.

---

## 📡 Integrasi Socket (Server Side)

Aplikasi ini mendengarkan event socket dengan detail sebagai berikut:
- **Event Name**: `print_job`
- **Payload**: `{ "content": "Teks yang ingin dicetak..." }`
- **Confirmation**: Server akan mengirim balik `print_status` setelah selesai.

---

### 🐘 Contoh: PHP (Laravel)
Gunakan library `elephant.io` atau `socket.io-client-php`.

```php
use ElephantIO\Client;

$url = "http://[IP-KOMPUTER-LOKAL]:3006";
$client = Client::create($url);

$client->initialize();
$client->emit('print_job', [
    'content' => "LAPAK KOPI\n----------\nKopi Hitam ... 5.000\nTotal ........ 5.000\n\nTerima Kasih!"
]);
$client->close();
```

---

### 🟢 Contoh: Node.js
Gunakan `socket.io-client`.

```javascript
const io = require("socket.io-client");
const socket = io("http://[IP-KOMPUTER-LOKAL]:3006");

socket.on("connect", () => {
    socket.emit("print_job", {
        content: "PESANAN #123\n----------------\nItem: Nasi Goreng\n----------------\n"
    });
});

socket.on("print_status", (data) => {
    console.log("Hasil Print:", data.message);
    socket.disconnect();
});
```

---

### 🐍 Contoh: Python
Gunakan `python-socketio`.

```python
import socketio

sio = socketio.Client()

@sio.event
def connect():
    sio.emit('print_job', {'content': 'TEST PRINT DARI PYTHON\n'})

@sio.on('print_status')
def on_status(data):
    print(f"Status: {data['message']}")
    sio.disconnect()

sio.connect('http://[IP-KOMPUTER-LOKAL]:3006')
sio.wait()
```

---

## 📄 Format ESC/POS (Tips)
Anda bisa mengirimkan perintah hardware ESC/POS langsung melalui string `content`:
- **Rata Tengah**: `\x1b\x61\x01`
- **Rata Kiri**: `\x1b\x61\x00`
- **Tebal (Bold)**: `\x1b\x45\x01`
- **Garis Bawah**: `\x1b\x2d\x01`

> [!IMPORTANT]
> Pengaturan ukuran font dasar (Font A/B) sebaiknya diatur secara lokal melalui **GUI Print Server** ini, bukan melalui logika di VPS. Hal ini untuk memastikan kompatibilitas jenis font dengan kemampuan fisik printer thermal Anda di lokasi. VPS cukup mengirimkan konten teks mentah, dan Print Server yang akan menyesuaikan format font-nya.
---

## 🖼️ Cetak Logo
Jika ingin mencetak logo, simpan logo di memori printer (**NVRAM**).

1. **Upload Logo**: Gunakan tool **"NV Logo Tool"** (biasanya ada di CD Driver atau download dari situsnya) untuk mengunggah logo hitam-putih Anda ke printer. Simpan sebagai logo nomor **1**.
2. **Panggil Logo dari VPS**: Gunakan kode berikut di dalam payload `content`:
   - Code: `\x1c\x70\x01\x00`
   - Rekomendasi (Center): `\x1b\x61\x01\x1c\x70\x01\x00\x1b\x61\x00` (Center logo lalu kembalikan ke rata kiri).


## 🔒 Jaringan & Keamanan
Agar Print Server dapat diakses oleh VPS atau perangkat lain, lakukan langkah berikut:

### 1. Izin Firewall Windows (Wajib)
Secara default Windows memblokir koneksi masuk. Anda harus mengizinkan port aplikasi (Standar: `3006`):
1.  Buka **Windows Defender Firewall with Advanced Security**.
2.  Klik **Inbound Rules** -> **New Rule...**
3.  Pilih **Port** -> Next.
4.  Pilih **TCP** dan isi **Specific local ports**: `3006`.
5.  Pilih **Allow the connection** -> Next -> Next.
6.  Beri nama (misal: `Thermal Socket Server`) dan klik **Finish**.

### 2. Gunakan IP Lokal Statis
Agar alamat Print Server tidak berubah-ubah saat komputer/router restart:
*   Atur **Static IP** pada Windows (Misal: `192.168.1.100`).
*   Atau gunakan fitur **DHCP Reservation** di pengaturan Router Anda.

### 3. Akses dari Luar Jaringan (VPS Cloud)
Jika VPS Anda tidak berada dalam satu jaringan (Wi-Fi/LAN) yang sama dengan printer:
*   Anda perlu melakukan **Port Forwarding** pada Router Anda.
*   Arahkan trafik dari Port `3006` (Publik) ke Port `3006` (IP Lokal komputer printer).


**Created by Pondok Algoritma**
