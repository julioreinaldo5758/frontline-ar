# Frontline Baby Souvenir — WebAR Experience

WebAR berbasis browser untuk souvenir bayi **Frontline**.  
Customer scan QR → buka browser → scan kartu fisik → kuda goyang 3D muncul + suara greeting.

**Teknologi:** MindAR.js v1.2.5 + Three.js r154 — pure browser, tanpa install aplikasi.

---

## Struktur Folder

```
frontline-ar/
├── index.html                          ← Halaman AR utama (siap pakai)
├── README.md                           ← Panduan ini
└── assets/
    ├── models/
    │   └── rockinghorsgdel-v1.glb     ← Taruh file GLB di sini
    ├── targets/
    │   └── marker.mind                ← Generate dari MindAR tools (lihat Step 2)
    └── audio/
        └── greeting.mp3              ← Rekam dan taruh file audio di sini
```

---

## Setup Step-by-Step

### Step 1 — Taruh File 3D

Salin file `rockinghorsgdel-v1.glb` ke:

```
assets/models/rockinghorsgdel-v1.glb
```

Nama file harus persis sama. Jika berbeda, edit baris ini di `index.html`:

```javascript
loader.load('./assets/models/rockinghorsgdel-v1.glb', ...
```

---

### Step 2 — Generate Marker (.mind)

File `.mind` dibuat dari foto kartu fisik souvenir yang akan di-scan customer.

**Langkah:**

1. Buka **MindAR Image Target Compiler**:
   `https://hiukim.github.io/mind-ar-js-doc/tools/compile`

2. Upload foto kartu souvenir fisik kamu
   - Gunakan foto yang terang, tidak buram, tidak miring
   - Gambar dengan detail kaya (logo, ornamen, teks) lebih mudah di-track
   - Resolusi minimal 300×300px, idealnya 600×800px
   - Format: JPG atau PNG

3. Klik **"Start"** → tunggu proses compile selesai

4. Download file hasil compile → rename menjadi `marker.mind`

5. Taruh di: `assets/targets/marker.mind`

**Tips desain kartu yang mudah di-track:**
- Hindari background polos tanpa detail
- Logo + ornamen + teks = kombinasi terbaik
- Kontras warna yang tinggi membantu tracking
- Hindari bidang reflektif/mengkilap saat foto

---

### Step 3 — Rekam Audio Greeting

Rekam suara greeting dalam format MP3 dan simpan sebagai `assets/audio/greeting.mp3`.

**Contoh script:**
> "Halo! Terima kasih sudah memilih Frontline Baby Souvenir.
> Semoga menjadi kenangan manis yang selalu dikenang.
> Selamat merayakan momen spesial bersama orang-orang tersayang!"

**Tools rekam gratis:**
- **Audacity** (PC/Mac) → Record → Export as MP3
- **Voice Recorder** bawaan HP → convert ke MP3 via convertio.co
- **Online recorder:** vocaroo.com → download → save as greeting.mp3

**Catatan iOS/Safari:** iPhone membutuhkan interaksi user (tap layar) sebelum bisa memutar audio otomatis. Greeting card yang muncul setelah scan sudah menangani ini — audio akan play saat target pertama kali ditemukan.

---

### Step 4 — Test Lokal

Browser membutuhkan **HTTPS atau localhost** untuk mengakses kamera — tidak bisa via `file://`.

**Opsi A — VS Code Live Server (paling mudah):**
1. Install extension "Live Server" di VS Code
2. Klik kanan `index.html` → "Open with Live Server"
3. Buka di browser, izinkan kamera

**Opsi B — Node.js:**
```bash
npx serve .
# Buka http://localhost:3000
```

**Opsi C — Python:**
```bash
python -m http.server 8000
# Buka http://localhost:8000
```

**Test di HP:**
1. Pastikan HP dan laptop terhubung ke WiFi yang sama
2. Cari IP laptop: `ipconfig` (Windows) → IPv4 Address
3. Buka `http://[IP-laptop]:PORT` di browser HP

---

### Step 5 — Deploy (Go Live)

Semua platform ini gratis dan otomatis HTTPS:

| Platform | Cara Deploy | Catatan |
|---|---|---|
| **Netlify** (Rekomendasi) | Drag & drop folder ke netlify.com/drop | Paling mudah, langsung jadi |
| **GitHub Pages** | Push ke repo → Settings → Pages | Butuh akun GitHub |
| **Vercel** | `npx vercel` di folder project | Butuh Node.js |

Setelah deploy, catat URL production-nya (contoh: `https://frontline-ar.netlify.app`).

---

### Step 6 — Buat QR Code

1. Buka **qrcode-monkey.com** atau **qr.io**
2. Masukkan URL AR experience
3. Sesuaikan warna dengan brand Frontline (pink: `#ff7aa8`)
4. Download resolusi tinggi (PNG, minimal 500×500px)
5. Sisipkan ke desain kartu souvenir dan cetak

---

## Kustomisasi

### Teks Greeting Popup

Cari dan edit di `index.html`:

```html
<h2>Selamat Datang!</h2>
<p>
  Terima kasih sudah memilih <strong>Frontline</strong>
  untuk momen spesial si kecil.<br><br>
  Semoga menjadi kenangan manis yang selalu dikenang. 💕
</p>
```

### Ukuran dan Posisi Model

```javascript
const targetSize = 0.85;       // Perbesar/perkecil (0.5 = kecil, 1.2 = besar)
horseModel.position.y += 0.12; // Naikkan/turunkan (angka lebih besar = lebih tinggi)
```

### Kecepatan dan Besar Ayunan

```javascript
tick += 0.022;                                // Kecepatan animasi (lebih besar = lebih cepat)
horseModel.rotation.z = Math.sin(tick) * 0.09; // 0.09 = sudut ayunan (lebih besar = lebih dramatis)
```

### Warna Pink Model

```javascript
mat.color.lerp(new THREE.Color(0xffb6c1), 0.08); // 0.08 = intensitas tint (0=tidak ada, 0.3=kuat)
```

### Warna Brand

Ganti nilai warna di bagian `<style>`:

```css
/* Warna utama button dan aksen */
background: linear-gradient(135deg, #ff7aaa, #d92060);
```

---

## Troubleshooting

| Masalah | Kemungkinan Penyebab | Solusi |
|---|---|---|
| Kamera tidak muncul | Buka via `file://` atau HTTP | Wajib HTTPS atau localhost |
| AR tidak deteksi kartu | Marker salah / pencahayaan kurang | Cek marker.mind, perbaiki pencahayaan |
| Model tidak muncul | Path atau nama file GLB salah | Cek nama file persis di `index.html` |
| Model terlalu besar/kecil | Skala default tidak cocok | Ubah nilai `targetSize` |
| Audio tidak berbunyi di iPhone | iOS blokir autoplay | Normal — akan play setelah target pertama ditemukan |
| Loading sangat lama | File GLB besar + koneksi lambat | Normal untuk file 3MB, tunggu sebentar |
| Marker cepat hilang | Pencahayaan tidak merata | Pindah ke tempat terang, hindari bayangan |

---

## Checklist Sebelum Cetak QR

- [ ] `rockinghorsgdel-v1.glb` sudah ada di `assets/models/`
- [ ] `marker.mind` sudah digenerate dari foto kartu yang akan dicetak
- [ ] `greeting.mp3` sudah direkam dan ada di `assets/audio/`
- [ ] Test di localhost berjalan normal (AR muncul, audio play, popup tampil)
- [ ] Deploy ke hosting HTTPS
- [ ] Test di HP via URL production
- [ ] QR code sudah mengarah ke URL production

---

## Tech Stack

| Library | Versi | Fungsi |
|---|---|---|
| MindAR.js | 1.2.5 | Image tracking & AR camera |
| Three.js | r154 | 3D rendering & lighting |
| GLTFLoader | r154 | Load file .glb |

Pure HTML/CSS/JS — tidak ada framework, tidak ada build tool, tidak ada server-side.

---

*Made with love for Frontline Baby Souvenir*
