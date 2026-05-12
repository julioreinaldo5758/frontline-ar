# CONTEXT.md — Frontline Baby Souvenir WebAR

> Dokumen ini dibuat sebagai memory session. Baca ini di awal session baru untuk langsung paham status project.

---

## Ringkasan Project

WebAR berbasis browser untuk bisnis souvenir bayi **Frontline**.  
Alur customer: scan QR di kartu souvenir → browser terbuka → scan kartu fisik → karakter kuda goyang 3D muncul di atas kartu + suara greeting.

**Keunggulan utama:** Tidak perlu install aplikasi. Pure browser (HTTPS).

---

## Status Deploy (Current)

| Item | Status | URL |
|---|---|---|
| GitHub repo | LIVE | https://github.com/julioreinaldo5758/frontline-ar |
| Railway (AR experience) | LIVE | https://frontline-ar-production.up.railway.app/ |
| Railway (Admin 3D Editor) | LIVE | https://frontline-ar-production.up.railway.app/admin.html |
| QR Code | **Belum dibuat** | Generate dari URL Railway di atas |

---

## Tech Stack

| Layer | Library / Tool | Versi | Catatan |
|---|---|---|---|
| AR Image Tracking | MindAR.js | 1.2.5 | A-Frame edition, via unpkg |
| AR Framework | A-Frame | 1.4.2 | via unpkg |
| 3D Editor (admin) | Three.js | r154 | ES module via unpkg, importmap |
| 3D Editor Loaders | GLTFLoader + DRACOLoader | r154 | via unpkg + gstatic Draco decoder |
| 3D Editor Controls | OrbitControls + TransformControls | r154 | via unpkg |
| Hosting | Railway | — | Dockerfile nginx:alpine |
| Build Tool | — | — | Tidak ada. Pure static HTML/CSS/JS |

**CDN index.html (AR experience):**
```
https://unpkg.com/aframe@1.4.2/dist/aframe.min.js
https://unpkg.com/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js
```

**CDN admin.html (3D Editor) — ES module via importmap:**
```
https://unpkg.com/three@0.154.0/build/three.module.js
https://unpkg.com/three@0.154.0/examples/jsm/  (GLTFLoader, DRACOLoader, OrbitControls, TransformControls)
https://www.gstatic.com/draco/versioned/decoders/1.5.6/  (Draco WASM decoder)
```

---

## Struktur Folder & Status File

```
frontline-ar/
├── index.html          [DONE] AR experience + baca URL params dari admin panel
├── admin.html          [DONE] 3D Editor visual — viewport Three.js + panel setting
├── Dockerfile          [DONE] nginx:alpine, serve static
├── nginx.conf          [DONE] MIME types untuk .glb, .mind, .mp3
├── CONTEXT.md          [DONE] File ini
├── README.md           [DONE] Panduan deploy (Bahasa Indonesia)
├── .gitignore          [DONE]
└── assets/
    ├── models/
    │   └── rockinghorsgdel-v1.glb   [READY] 3.16 MB, Draco compressed, ada tekstur
    ├── targets/
    │   └── marker.mind              [READY] 528 KB — image target MindAR
    └── audio/
        └── greeting.mp3             [READY] 90 KB — audio greeting
```

---

## Arsitektur index.html (AR Experience)

**Tech:** A-Frame 1.4.2 + MindAR 1.2.5 (A-Frame edition), Vanilla JS, no build tool.

**Alur startAR():**
1. Cek library (AFRAME, MINDAR) — tampilkan error spesifik jika gagal
2. Fetch HEAD `marker.mind` (absolute URL — wajib, karena MindAR jalan di Web Worker)
3. `getUserMedia()` eksplisit dalam user-gesture context → stop stream → MindAR buat stream sendiri
4. Buat `<a-scene>` dinamis → insert ke DOM → tunggu event `arReady` (timeout 15 detik)

**URL params** dibaca oleh `getConfig()` saat `startAR()` dipanggil:

| Param | Default | Fungsi |
|---|---|---|
| posX | 0 | Posisi X model di atas kartu |
| posY | 0.15 | Tinggi model dari permukaan kartu |
| posZ | 0 | Posisi Z model |
| rotX | 0 | Rotasi X (derajat) |
| rotY | 0 | Rotasi Y (derajat) |
| rotZ | 0 | Rotasi Z (derajat) |
| scale | 0.85 | Ukuran model (1 unit = lebar kartu) |
| rockOn | true | Toggle animasi goyang |
| rockAmp | 5 | Sudut goyang maksimum (derajat) |
| rockDur | 1200 | Durasi 1 siklus goyang (ms) |
| bobOn | true | Toggle animasi naik-turun |
| bobAmp | 0.06 | Tinggi naik-turun maksimum |
| bobDur | 1800 | Durasi 1 siklus naik-turun (ms) |

**Struktur entity A-Frame di dalam scene:**
```
<a-entity id="ar-target" mindar-image-target="targetIndex: 0">
  <a-entity position="{posX posY posZ}" rotation="{rotX rotY rotZ}">   ← dari URL params
    <a-entity animation__rock animation__bob>                           ← animasi
      <a-gltf-model auto-scale="size: {scale}">                        ← model GLB
```

> ⚠️ **Jangan** tambah wrapper `<a-entity rotation="90 0 0">` di sini.  
> MindAR image target: kartu di bidang XZ, **Y = normal kartu (atas)**. Model berdiri tegak secara default.  
> Wrapper itu justru bikin posY bergerak sepanjang kartu & model miring. Sudah difix 2026-05-12.

**Komponen `auto-scale`:**  
Normalizes model: fit dalam `size` units, center horizontal, bottom geometri di y=0.

---

## Arsitektur admin.html (3D Editor)

**Tech:** Three.js r154 ES modules (importmap), GLTFLoader + DRACOLoader, OrbitControls, TransformControls. Tidak perlu backend/server.

**Cara kerja:**
- Viewport Three.js menampilkan: grid floor + card plane putih (rasio 1:0.631, sesuai kartu fisik) + model GLB
- Struktur Three.js: `configGroup` (position/rotation dari panel) → `animGroup` (animasi preview) → model
- `TransformControls` attach ke `configGroup` → drag gizmo XYZ update input panel secara real-time
- Panel input update `configGroup` → Three.js scene update real-time
- Tombol **Salin URL** / **Buka AR** → generate URL `/?posX=...&posY=...&...` yang kompatibel dengan `index.html`

**Fitur toolbar viewport:**
- `↔ Move` — gizmo translate XYZ
- `↻ Rotate` — gizmo rotate XYZ
- `⏸ Anim` — pause/play animasi preview
- `⌖ View` — reset kamera ke posisi default

**Multi-project:** Untuk project baru (misal naga), ganti field "Model 3D" ke path GLB baru → klik Load. Admin panel-nya sama, tidak perlu dibuat ulang.

---

## Cara Pakai Admin Panel (Workflow)

1. Buka `https://frontline-ar-production.up.railway.app/admin.html`
2. Lihat kuda di viewport — drag gizmo atau geser slider untuk atur posisi
3. Pause animasi (⏸) kalau mau lihat posisi statis
4. Klik **Salin URL** → dapat URL seperti:  
   `https://frontline-ar-production.up.railway.app/?posY=0.3&scale=1.1&rockAmp=8&...`
5. **URL itu yang dijadikan QR code** — semua customer scan QR → lihat kuda di posisi yang sudah disetting

---

## Cara Lokal Preview (tanpa deploy)

```
cd C:\Users\ELVINA\ClaudeProjects\frontline-ar
npx serve .
```
Buka `http://localhost:3000` di Chrome.  
Admin panel: `http://localhost:3000/admin.html`

> ⚠️ AR experience (index.html) butuh HTTPS untuk kamera — lokal hanya bisa test UI-nya.  
> Untuk test AR penuh, harus deploy ke Railway dulu.

---

## Next Steps (Urutan Prioritas)

- [ ] **Test admin panel** — buka `/admin.html`, atur posisi kuda sampai pas, salin URL
- [ ] **Generate QR code** — dari URL hasil export admin panel (pakai qr-code-generator.com atau canva)
- [ ] **Test end-to-end di HP** — scan QR → AR muncul → suara → popup greeting
- [ ] **Fine-tune posisi** — kalau kuda masih aneh posisinya, adjust di admin panel lagi
- [ ] (Opsional) **Hapus file .txt helper** dari folder assets (sudah di .gitignore, tidak urgent)
- [ ] (Opsional) **Rekam ulang audio** greeting.mp3 kalau suaranya kurang cocok

---

## Design System UI (harus match hi.frontline.id)

**Dark luxury gold** — bukan pink. Referensi: https://hi.frontline.id/

| Token | Nilai | Fungsi |
|---|---|---|
| `--bg` | `#080807` | Background utama |
| `--gold` | `#C8980A` | Aksen utama, tombol CTA |
| `--gold2` | `#E2B830` | Gold lebih terang (highlight) |
| `--gold-dim` | `#8A6A08` | Gold redup (border, divider) |
| `--gold-bg` | `#110D02` | Dark gold tint (background chip) |
| `--surface` | `#0F0E0B` | Card / overlay background |
| `--border2` | `#2C2918` | Border subtle |
| `--text` | `#EDE8DC` | Teks utama (cream) |
| `--muted` | `#7A7260` | Teks sekunder |

**Font:** `Gilda Display` (serif, heading/logo) + `DM Sans` (sans-serif, body) — Google Fonts.

---

## Hal Penting yang Sudah Resolved (Jangan Diutak-atik)

| Masalah | Solusi |
|---|---|
| MindAR "no internet" di Web Worker | Wajib pakai **absolute URL** untuk `imageTargetSrc` (`window.location.origin + '/assets/...'`) |
| Kamera tidak minta izin | `getUserMedia()` eksplisit di dalam click handler sebelum buat `<a-scene>` |
| `arReady` tidak pernah fire | Hapus `<a-assets>` preloading — model load langsung di entity |
| CDN jsDelivr tidak bisa diakses | Ganti ke **unpkg.com** untuk semua library AR |
| Model Draco compressed | Tambah `DRACOLoader` dengan decoder dari gstatic |
| Model amblas ke kartu | `auto-scale` taruh bottom model di y=0; URL param `posY` angkat ke atas |
| Kuda muncul di depan kartu bukan di atas | Hapus wrapper `rotation="90 0 0"` — MindAR Y = normal kartu, model berdiri tegak by default |
| UI warna pink tidak sesuai brand | Redesign ke dark gold theme (lihat Design System di atas) |

---

*Last updated: 2026-05-12 | Working dir: C:\Users\ELVINA\ClaudeProjects\frontline-ar*
