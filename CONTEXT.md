# CONTEXT.md — Frontline Baby Souvenir WebAR

> Dokumen ini dibuat sebagai memory session. Baca ini di awal session baru untuk langsung paham status project.

---

## Ringkasan Project

WebAR berbasis browser untuk bisnis souvenir bayi **Frontline**.  
Alur customer: scan QR di kartu souvenir → browser terbuka → scan kartu fisik → karakter kuda goyang 3D pink muncul + suara greeting.

**Keunggulan utama:** Tidak perlu install aplikasi. Pure browser (HTTPS).

---

## Tech Stack

| Layer | Library / Tool | Versi | Catatan |
|---|---|---|---|
| AR Image Tracking | MindAR.js | 1.2.5 | CDN, bundled via jsDelivr |
| 3D Rendering | Three.js | r154 | CDN, UMD build |
| 3D Model Loader | GLTFLoader | r154 | CDN, dari examples/js Three.js |
| Hosting | Railway | — | Deployment target (bukan Netlify) |
| Build Tool | — | — | Tidak ada. Pure static HTML/CSS/JS |
| Framework | — | — | Tidak ada. Vanilla JS |

**CDN yang dipakai di index.html:**
```
https://cdn.jsdelivr.net/npm/three@0.154.0/build/three.min.js
https://cdn.jsdelivr.net/npm/three@0.154.0/examples/js/loaders/GLTFLoader.js
https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-three.prod.js
```

---

## Struktur Folder & Status File

```
frontline-ar/
├── index.html                          [DONE] AR experience lengkap
├── README.md                           [DONE] Panduan deploy lengkap (Bahasa Indonesia)
├── CONTEXT.md                          [DONE] File ini
└── assets/
    ├── models/
    │   ├── rockinghorsgdel-v1.glb     [READY] 3.16 MB, sudah ada tekstur
    │   └── TARUH-FILE-GLB-DI-SINI.txt [helper, bisa dihapus]
    ├── targets/
    │   ├── targets.mind               [READY] 528 KB — ⚠️ lihat catatan di bawah
    │   └── CARA-BUAT-MARKER.txt       [helper, bisa dihapus]
    └── audio/
        ├── greeting.mp3               [MISSING] Belum direkam
        └── CARA-BUAT-AUDIO.txt        [helper, bisa dihapus]
```

---

## ⚠️ Isu Yang Harus Diselesaikan Sebelum Live

### 1. NAMA FILE MARKER TIDAK COCOK (Prioritas Tinggi)

File marker yang ada: `assets/targets/targets.mind`  
Yang diexpect index.html: `assets/targets/marker.mind`

**Fix — pilih salah satu:**

**Opsi A** — Rename file (lebih simpel):
```
Rename: targets.mind → marker.mind
```

**Opsi B** — Update index.html (jika tidak mau rename):
```javascript
// Di index.html, baris MindARThree options, ganti:
imageTargetSrc: './assets/targets/marker.mind',
// Menjadi:
imageTargetSrc: './assets/targets/targets.mind',
```

### 2. AUDIO BELUM ADA

File `assets/audio/greeting.mp3` belum dibuat.  
AR tetap berjalan tanpa audio — hanya tidak ada suara greeting.  
Rekam audio → simpan sebagai `greeting.mp3` di folder `assets/audio/`.

---

## Apa yang Sudah Selesai di index.html

- Custom loading screen branded pink Frontline (menggantikan loader default MindAR)
- Header brand "FRONTLINE" + tagline di atas kamera
- Scan instruction pill dengan animasi bobbing
- MindAR image tracking (1 anchor, target index 0)
- Load GLB via GLTFLoader + auto-centre + auto-scale ke lebar kartu AR
- Lighting: ambient + key light (warm pink) + fill light + rim light
- Animasi rocking: ayunan Z-axis + bobbing Y-axis berulang
- Pink tint halus di atas material asli model (lerp 8% ke #ffb6c1)
- Greeting popup card muncul 750ms setelah target pertama ditemukan
- Audio play saat target ditemukan (dengan `.catch()` untuk iOS silent mode)
- Error handler: tampilkan pesan jika kamera gagal diakses
- scanHint sembunyi saat target ditemukan, muncul lagi saat target hilang

---

## Deploy ke Railway

Railway melayani static files via service. Langkah deploy:

1. **Pastikan semua aset sudah ada** (marker.mind fix + greeting.mp3)
2. Push folder `frontline-ar/` ke GitHub repo
3. Buka Railway → New Project → Deploy from GitHub repo
4. Railway akan detect static files — jika perlu, tambahkan config:

   Buat file `railway.toml` di root folder:
   ```toml
   [build]
   builder = "nixpacks"

   [deploy]
   startCommand = "npx serve . -l $PORT"
   ```

   Atau gunakan **Static Site** template di Railway (drag & drop atau GitHub).

5. Railway otomatis assign domain HTTPS (misal: `frontline-ar.up.railway.app`)
6. Gunakan URL tersebut untuk generate QR code

**Penting:** Railway wajib HTTPS — sudah otomatis terpenuhi.

---

## Next Steps (Urutan Prioritas)

- [ ] **Fix nama file marker** — rename `targets.mind` → `marker.mind` (atau update index.html)
- [ ] **Rekam audio greeting** — simpan sebagai `assets/audio/greeting.mp3`
- [ ] **Test lokal** — via Live Server / `npx serve .` sebelum deploy
- [ ] **Deploy ke Railway** — push ke GitHub → connect di Railway
- [ ] **Generate QR code** dari URL Railway
- [ ] **Test end-to-end** di HP: scan QR → AR muncul → suara → popup
- [ ] (Opsional) Hapus file `.txt` helper dari folder assets setelah tidak diperlukan

---

## Parameter Kustomisasi Cepat

Semua ada di `index.html` bagian `<script>`:

```javascript
const targetSize = 0.85;                        // Skala model (0.5 kecil — 1.5 besar)
horseModel.position.y += 0.12;                  // Tinggi model di atas kartu
tick += 0.022;                                  // Kecepatan animasi rocking
horseModel.rotation.z = Math.sin(tick) * 0.09; // Sudut ayunan (0.09 = ~5 derajat)
mat.color.lerp(new THREE.Color(0xffb6c1), 0.08); // Intensitas pink tint
```

Greeting card text ada di bagian HTML:
```html
<h2>Selamat Datang!</h2>
<p>Terima kasih sudah memilih <strong>Frontline</strong> ...</p>
```

---

*Last updated: 2026-05-11 | Working dir: C:\Users\ELVINA\ClaudeProjects\frontline-ar*
