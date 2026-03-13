# Thermal Socket Server 🚀

**Thermal Socket Server** adalah aplikasi Desktop berbasis **Rust + Tauri** yang berfungsi sebagai jembatan (bridge) antara Server/VPS Anda dengan printer thermal lokal. Aplikasi ini memungkinkan Anda melakukan pencetakan struk secara otomatis dan *silent* (tanpa dialog print) menggunakan protokol **Socket.IO**.

## ✨ Fitur Utama
- **Silent Printing**: Cetak struk langsung ke printer thermal tanpa perlu interaksi user.
- **Socket.IO Support**: Integrasi mudah dengan Laravel, Node.js, Python, dsb.
- **Font Selection**: Pilih ukuran font dasar (Standard, Small, atau Small Bold) langsung dari GUI.
- **System Tray**: Berjalan di latar belakang, meminimalkan penggunaan layar.
- **Auto-Discovery**: Mencari printer yang terhubung ke sistem secara otomatis.

---

## 🛠️ Instalasi & Persiapan
1. Download installer dari [GitHub Releases](https://github.com/pondokalgoritma/thermal-socket-server/releases/latest).
2. Jalankan aplikasi pada komputer yang terhubung langsung ke printer thermal.
3. Buka tab **Settings** untuk mengatur:
   - **Printer Target**: Pilih printer thermal yang aktif.
   - **Socket Port**: Port untuk mendengarkan koneksi (Default: `3006`).
   - **Font Size**: Pilih ukuran font yang sesuai dengan kertas Anda.

> [!IMPORTANT]
> Pastikan Port yang Anda gunakan tidak diblokir oleh Firewall agar Server/VPS bisa melakukan koneksi.

### 📂 Lokasi File Konfigurasi
Aplikasi menyimpan pengaturan Anda secara permanen di:
`%APPDATA%\dev.denox.thermal-socket-server\config.## 📡 Cara Kerja: Browser sebagai Jembatan (Bridge)

Jika aplikasi Anda (misal: Laravel) berada di **VPS/Cloud**, server Anda tidak bisa mengakses `localhost` printer secara langsung. Solusi paling efisien adalah menggunakan browser user sebagai perantara.

### Alur Kerja:
1. **Trigger**: User mengklik tombol "Cetak" di aplikasi Cloud.
2. **Bridge**: Aplikasi membuka halaman/tab khusus (atau iframe) yang berisi data struk.
3. **Socket**: JavaScript di halaman tersebut menghubungkan browser ke `localhost:3006` milik Thermal Socket Server.
4. **Print**: Data dikirim, printer mencetak, dan halaman ditutup otomatis.

**Keuntungan**: Tidak perlu melakukan *Port Forwarding* atau pengaturan IP Publik pada router lokal.

---

## 🚀 Integrasi (Frontend/Client Side)

Ini adalah cara paling umum digunakan untuk aplikasi berbasis web (Laravel, React, dsb).

### 1. Buat Halaman Pemicu (Contoh: `cetak.blade.php`)
Buat halaman khusus yang akan dipicu saat proses cetak.

```html
<!-- Data Struk (Tersembunyi) -->
<pre id="struk-data" style="display:none;">
TOKO MAKMUR
--------------------------
Kopi Susu ..... Rp 15.000
--------------------------
Total ......... Rp 15.000
</pre>

<script src="https://cdn.socket.io/4.7.2/socket.io.min.js"></script>
<script>
    const socket = io("http://localhost:3006");
    
    socket.on("connect", () => {
        const content = document.getElementById('struk-data').innerText;
        socket.emit("print_job", { content: content });
    });

    socket.on("print_status", (res) => {
        if (res.success) {
            console.log("Cetak Berhasil");
            window.close(); // Tutup tab otomatis
        }
    });
</script>
```

### 2. Cara Trigger dari Aplikasi
Anda bisa memicu cetak dengan membuka jendela baru atau menggunakan **Hidden Iframe** (lebih elegan/silent).

**A. Menggunakan Tab Baru (Simple):**
```html
<a href="/cetak/123" target="_blank">Cetak Struk</a>
```

**B. Menggunakan Hidden Iframe (Silent Printing):**
```html
<!-- Letakkan ini di layout utama -->
<iframe id="silent-printer" style="display:none;"></iframe>

<!-- Tombol Cetak -->
<button onclick="document.getElementById('silent-printer').src='/cetak/123'">
    Cetak (Silent)
</button>
```

---

## 📄 Format ESC/POS (Tips)
Anda bisa mengirimkan perintah hardware ESC/POS langsung melalui string `content`:
- **Rata Tengah**: `\x1b\x61\x01`
- **Rata Kiri**: `\x1b\x61\x00`
- **Tebal (Bold)**: `\x1b\x45\x01`
- **Garis Bawah**: `\x1b\x2d\x01`

> [!IMPORTANT]
> **Penting**: Pengaturan ukuran font dasar (Standard/Small) diatur melalui **Settings** di aplikasi Thermal Socket Server. VPS cukup mengirimkan konten teks mentah.

---

## 🔒 Jaringan & Keamanan
Jika Anda menggunakan metode **Browser Bridge** di atas:
*   **Tidak perlu** Port Forwarding.
*   **Tidak perlu** mengizinkan Firewall untuk koneksi Inbound dari Internet.
*   Cukup pastikan browser dapat mengakses `http://localhost:3006`.

Jika Anda ingin server (VPS) menembak **langsung** ke IP komputer tanpa browser:
1. **Izin Firewall Windows (Wajib)**: Izinkan Port `3006` pada Inbound Rules.
2. **Port Forwarding**: Arahkan port `3006` di Router ke IP lokal komputer printer.
3. **Static IP**: Pastikan komputer printer memiliki IP lokal yang tidak berubah.
 `3006` (IP Lokal komputer printer).

---

**DENOX.DEV**
