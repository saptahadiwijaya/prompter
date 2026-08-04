# Prompter Cihuy — v0.9.3-alpha

Teleprompter PWA untuk tablet Android dengan **live sync Google Docs**.

## Fitur

- **Live sync Google Docs** via Google Apps Script (polling 2,5 detik), posisi baca tidak lompat saat naskah berubah (anchor per paragraf)
- **Formatter otomatis**: KAPITAL semua, `,` → `/`, `.` → `//`, dengan proteksi angka Indonesia (`Rp2.500.000`, `2,5 juta`) dan singkatan umum
- **Fullscreen**, **mirror horizontal/vertikal**, **kecepatan px/detik**, **ukuran huruf**, **jarak baris**
- **Garis baca** dengan posisi bisa diatur (default 30% dari atas, ideal untuk kamera di atas kaca) + highlight baris aktif
- **Preset** (termasuk link naskah) tersimpan di localStorage
- **Dark / light mode**
- **Countdown 3-2-1**, ketuk layar untuk jeda/lanjut, jeda otomatis di akhir naskah
- **Wake Lock** — layar tablet tidak mati selama scroll berjalan
- **Offline fallback** — naskah terakhir tetap tampil kalau WiFi putus, dengan indikator status sync (lampu tally)
- **Estimasi durasi baca** (± 140 kata/menit)
- **Keyboard / remote Bluetooth**: `Spasi` jeda-lanjut, `↑↓` kecepatan, `R` reset, `F` fullscreen, `M` mirror
- **Mode teks manual** sebagai fallback tanpa Google Docs
- **Password koneksi**: URL Apps Script + token ditanam di kode (`lib/config.ts`); pengguna cukup memasukkan password (6 karakter terakhir token) sekali per perangkat
- **Aturan tanda baca**: koma/titik dua/titik koma → `/`, titik → `//`, dengan **satu spasi sebelum & sesudah** setiap `/` dan `//` (naskah yang sudah memakai `/` `//` pun ikut dirapikan). Tanda tanya `?` dan seru `!` dipertahankan apa adanya. Angka (`Rp2.500.000`), domain, dan `[cue]` tetap aman.
- **Warna penanda jeda `/` `//`**: bisa diubah dari tablet maupun remote (Auto = ikut warna teks, atau 7 pilihan warna).
- **Catatan sutradara `[Teks]`**: teks dalam bracket tampil dengan **warna dan ukuran sendiri** — default merah 20px, bisa diatur dari tablet maupun remote (slider *Ukuran catatan* 12–48px + 7 pilihan warna), tidak terpengaruh slider ukuran huruf utama, tidak di-kapital, tanda baca di dalamnya tidak dikonversi, dan tidak dihitung dalam estimasi durasi baca. Di panel naskah remote juga ditandai merah.
- **Proteksi URL/domain/email** di formatter: `accurate.id`, `https://accurate.id/promo`, `halo@accurate.id` tampil utuh
- **Remote control** dari HP di halaman `/remote` — play/pause/reset, kecepatan, ukuran huruf, jarak baris, posisi garis baca, mirror, tema — via relay Apps Script (CacheService), tanpa server tambahan. Pairing dengan kode ruang 4 digit.

## Setup

### 1. Apps Script (jembatan Google Docs)

1. Buka [script.google.com](https://script.google.com) → **New project** → paste isi `apps-script/Code.gs`.
2. Jalankan fungsi `setupToken()` sekali → izinkan akses → salin token dari **Logs**.
   (Token tersimpan di **Script Properties**, jadi aman saat Code.gs di-paste ulang.)
3. **Deploy → New deployment → Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Salin URL `/exec`.

> Dokumen naskah harus bisa diakses akun pemilik script (milik sendiri atau di-share ke akun itu). Tidak perlu "Publish to web".

### 2. Isi konfigurasi bawaan

**Cara direkomendasikan (kebal update kode):** di dashboard Vercel → Settings → Environment Variables, tambahkan `NEXT_PUBLIC_GAS_URL` (URL `/exec`) dan `NEXT_PUBLIC_GAS_TOKEN` (token dari Script Properties), lalu redeploy. Nilai ini tidak akan tertimpa walau kode diganti zip baru.

Alternatif: buka `lib/config.ts` → isi `gasUrl` dengan URL `/exec` dan pastikan `gasToken` sesuai token di Script Properties (catatan: cara ini harus diulang setiap kali mengganti kode dengan zip rilis baru). Password aplikasi otomatis = **6 karakter terakhir token**.

> Kalau `gasUrl` dibiarkan placeholder, aplikasi otomatis menampilkan input manual URL + token (mode lama) — berguna untuk deployment tanpa konfigurasi bawaan.

### 3. Deploy aplikasi ke Vercel

```bash
npm install
npm run build   # verifikasi
```

Push ke GitHub → import di Vercel. **Tidak ada environment variable yang dibutuhkan** — URL Apps Script + token diisi dari UI (Pengaturan koneksi) dan tersimpan di localStorage perangkat.

### 4. Di tablet

1. Buka URL Vercel di Chrome → menu → **Add to Home screen** (terpasang sebagai aplikasi, fullscreen + landscape).
2. Buka **Pengaturan koneksi Apps Script** → masukkan **password** (6 karakter terakhir token) → Simpan koneksi. Cukup sekali per perangkat.
3. Tempel link Google Docs → **HUBUNGKAN**.

## Update Code.gs di kemudian hari

Paste kode baru → **Deploy → Manage deployments → Edit → New version**. Token tidak perlu disentuh (Script Properties).

## Struktur

```
app/            layout, page, globals.css
components/     PrompterApp.tsx (engine scroll, polling, UI)
lib/            formatter.ts (aturan konversi), store.ts (preset & koneksi)
public/         manifest.json, sw.js, ikon PWA
apps-script/    Code.gs
```

## Remote control

1. **Tablet**: panel kiri → bagian **REMOTE** → *Aktifkan remote* → catat kode 4 digit.
2. **HP**: buka `https://<url-vercel>/remote` → masukkan password (sama, 6 karakter terakhir token) → masukkan kode ruang → **Sambungkan**.
3. HP menampilkan status live (progress %, berjalan/jeda, jumlah kata) dan semua kontrol dengan tombol besar (+/− per slider, cocok untuk jempol).
4. **Panel NASKAH**: remote menampilkan naskah yang sama persis dengan tablet (dikirim tablet lewat relay, jadi mode Google Docs maupun teks manual sama-sama tampil). Baris yang sedang di garis baca di-highlight dan otomatis diikuti (toggle *Ikuti baris*). **Ketuk/klik baris mana pun → tablet melompat ke baris itu** tepat di garis baca.
5. **Tampilan penuh** (tombol *▢ Penuh* di header): layout dua kolom untuk laptop — kontrol di kiri, naskah besar di kanan. Default tetap tampilan remote kompak.
6. **Kelola sumber naskah**: toggle *Tablet / Remote* di bagian SUMBER NASKAH (ada di kedua sisi, tersinkron). Saat **Remote**: menu ganti naskah (link Google Docs / teks manual) pindah ke halaman remote dan hilang dari tablet — praktis untuk pergantian naskah antar-take tanpa menyentuh tablet. Saat **Tablet** (default): sebaliknya. Sisi yang tidak memegang kendali tetap melihat toggle-nya agar kendali bisa diambil kembali kapan saja.

### Jalur cepat (opsional, disarankan) — Supabase Realtime

Tanpa ini, perintah remote lewat relay Apps Script dengan latensi ± 1,5–3 detik. Dengan Supabase Realtime, latensi turun ke ± 0,1–0,3 detik. Hitung mundur 3-2-1 bisa dinyalakan/dimatikan lewat centang "Hitung mundur" (di tablet & remote, default nyala). Overlay 3-2-1 juga muncul di remote, tersinkron dengan tablet.

Setup sekali:
1. Buat project gratis di [supabase.com](https://supabase.com) (tidak perlu bikin tabel apa pun — fitur Broadcast bersifat ephemeral).
2. Project Settings → API → salin **Project URL** dan **anon public key**.
3. Di Vercel → Environment Variables, tambahkan `NEXT_PUBLIC_SUPABASE_URL` dan `NEXT_PUBLIC_SUPABASE_ANON_KEY`, lalu redeploy.

anon key memang dirancang aman untuk dipasang di sisi client (dilindungi kode ruang 4 digit + password aplikasi). Kalau env var ini kosong, aplikasi otomatis kembali ke mode relay Apps Script — tidak ada yang rusak.

Indikator: panel REMOTE di tablet menunjukkan "Jalur cepat aktif", dan header remote menampilkan badge ⚡ CEPAT saat realtime tersambung.

**Update Apps Script diperlukan**: paste `Code.gs` v0.5.0 → Deploy → Manage deployments → Edit → **New version**. Token & URL tidak berubah.
