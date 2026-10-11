# DESIGN: SIGAP Gempa

Panduan visual dan komponen. Semua teks antarmuka berbahasa Indonesia.

---

## 1. Arah desain

**Konsep: lembar rekaman seismograf.** Web ini terasa seperti kertas grafik tempat jarum seismograf menulis. Netral, tenang, terbaca di mana saja, dan satu-satunya warna jenuh adalah skala intensitas gempa, sama seperti peta guncangan resmi.

**Bagian yang paling diingat: jejak seismogram di hero.** Garis yang digambar sekali saat halaman dibuka, memperlihatkan gelombang P kecil yang datang lebih dulu, lalu gelombang S yang besar. Tingginya mengikuti magnitudo gempa terbaru. Semua yang lain sengaja dibuat tenang.

**Prinsip**

1. **Data lebih dulu.** Angka penting (magnitudo, waktu, lokasi) selalu terbaca dalam sekali lihat, termasuk saat pengguna panik atau sinyal lemah.
2. **Warna membawa arti, bukan hiasan.** Warna jenuh hanya muncul untuk tingkat magnitudo dan peringatan. Selain itu, tampilan netral dingin.
3. **Warna tidak pernah sendirian.** Setiap warna tingkat selalu disertai kata ("Sedang", "Kuat").
4. **Tenang saat darurat.** Tidak ada animasi berkedip, tidak ada merah di tempat yang tidak berbahaya.
5. **Struktur adalah informasi.** Garis, penomoran, dan pembatas hanya dipakai kalau memang mewakili urutan atau pengelompokan.

## 2. Warna

### 2.1 Palet dasar

| Nama | Hex | Peran |
|---|---|---|
| Kertas | `#EAF0F2` | Latar halaman (biru-abu dingin) |
| Garis kertas | `#D3DEE3` | Garis kisi seismograf |
| Permukaan | `#F8FBFC` | Panel, baris data, kartu |
| Tinta | `#12303F` | Teks utama, jejak seismogram |
| Tinta redup | `#4A6573` | Teks sekunder |
| Jejak | `#0A6C85` | Tautan, tombol, fokus, elemen interaktif |
| Jejak gelap | `#07495B` | Hover dan state aktif |

### 2.2 Skala magnitudo (satu-satunya warna jenuh)

Terinspirasi skala warna peta guncangan: dari dingin ke hangat.

| Tingkat | Rentang | Warna | Teks di atasnya |
|---|---|---|---|
| Kecil | M di bawah 4 | `#8FD0E0` | Tinta |
| Ringan | M 4,0 sampai 4,9 | `#A6D96A` | Tinta |
| Sedang | M 5,0 sampai 5,9 | `#F5C84B` | Tinta |
| Kuat | M 6,0 sampai 6,9 | `#EE7B30` | Tinta |
| Besar | M 7,0 ke atas | `#C4262E` | Putih |

> Nama dan batas tingkat mengikuti klasifikasi umum magnitudo. Cocokkan dengan sumber resmi (BMKG) sebelum dipakai di laporan.

### 2.3 Warna status

| Status | Tampilan |
|---|---|
| Tidak berpotensi tsunami | Teks tinta di atas Permukaan, ikon centang, garis kiri `#2F7D5B` |
| Berpotensi tsunami | Latar `#C4262E`, teks putih, ikon peringatan, selalu disertai kalimat lengkap |
| Tidak dinyatakan | Tanpa label status; tampilkan teks asli BMKG |

### 2.4 Token CSS

```css
:root {
  --kertas: #EAF0F2;
  --kertas-garis: #D3DEE3;
  --permukaan: #F8FBFC;
  --tinta: #12303F;
  --tinta-redup: #4A6573;
  --jejak: #0A6C85;
  --jejak-gelap: #07495B;

  --mag-kecil: #8FD0E0;
  --mag-ringan: #A6D96A;
  --mag-sedang: #F5C84B;
  --mag-kuat: #EE7B30;
  --mag-besar: #C4262E;

  --aman: #2F7D5B;
  --garis-halus: rgba(18, 48, 63, 0.14);
  --fokus: 0 0 0 3px var(--kertas), 0 0 0 6px var(--jejak);
}
```

Mode gelap tidak termasuk versi ini.

### 2.5 Latar kisi seismograf

Dipakai hanya di hero dan pita pembuka tiap bagian, tidak di belakang teks panjang.

```css
.kisi {
  background-color: var(--kertas);
  background-image:
    linear-gradient(var(--kertas-garis) 1px, transparent 1px),
    linear-gradient(90deg, var(--kertas-garis) 1px, transparent 1px);
  background-size: 24px 24px;
}
```

## 3. Tipografi

| Peran | Font | Bobot |
|---|---|---|
| Judul dan angka besar | Bricolage Grotesque | 600 dan 800 |
| Isi dan antarmuka | Atkinson Hyperlegible | 400 dan 700 |

Alasan: Bricolage punya karakter yang jelas untuk judul dan angka magnitudo. Atkinson dirancang agar sangat mudah dibaca, cocok untuk informasi darurat dan pembaca dengan penglihatan terbatas.

Pasang lewat `next/font/google` dan cek nama impor serta opsi bobot di dokumentasi Next.js. Pakai `font-variant-numeric: tabular-nums` pada angka data bila fontnya mendukung.

| Tingkat | Ukuran | Tinggi baris | Pemakaian |
|---|---|---|---|
| Magnitudo besar | `clamp(3.5rem, 9vw, 6.5rem)` | 1 | Angka di hero |
| Judul 1 | `clamp(2rem, 4.5vw, 3rem)` | 1,1 | Judul halaman |
| Judul 2 | `1.75rem` | 1,2 | Judul bagian |
| Judul 3 | `1.25rem` | 1,3 | Sub-bagian, judul kartu |
| Isi | `1.0625rem` (17 px) | 1,6 | Teks umum |
| Kecil | `0.875rem` | 1,5 | Keterangan dan metadata |

Aturan teks:
- Panjang baris isi maksimal 68 karakter (`max-width: 68ch`).
- Teks rata kiri di semua tempat.
- Kalimat huruf biasa (sentence case). Tidak memakai huruf kapital semua untuk label, dan tidak ada label kecil di atas setiap judul.
- Angka desimal memakai koma (4,6), waktu ditulis "04.38 WIB".

## 4. Jarak, bentuk, dan elevasi

- Satuan dasar 4 px. Skala: 4, 8, 12, 16, 24, 32, 48, 72.
- Lebar konten maksimum 1200 px, gutter 16 px (HP) dan 32 px (desktop). Bacaan panjang di kolom 68ch.
- Sudut membulat dibedakan menurut peran: baris data 0, panel 10 px, chip dan tombol kecil pil penuh.
- Tanpa bayangan di kartu. Pemisah memakai garis halus (`--garis-halus`). Satu-satunya bayangan ada di panel chatbot: `0 12px 32px rgba(18, 48, 63, 0.18)`.

## 5. Elemen khas: jejak seismogram

Komponen `SeismogramTrace` (SVG, client component) untuk hero beranda dan halaman proses gempa.

**Bentuk**
- Garis tunggal berwarna Tinta, tebal 2 px, melintang selebar hero, berjalan dari kiri ke kanan.
- Segmen awal hampir datar dengan noise kecil (±2 px).
- Di sekitar 28% lebar muncul **gelombang P**: amplitudo kecil (sekitar 25% dari puncak), pendek.
- Di sekitar 36% lebar muncul **gelombang S**: amplitudo puncak, lalu meluruh bertahap sampai ujung kanan.
- Dua penanda kecil pada garis: "P" dan "S" (teks Tinta redup, ukuran kecil).

**Amplitudo.** Puncak dipetakan dari magnitudo: M 3 menjadi 6 px, M 8 menjadi 60 px, linear dan dibatasi di antara keduanya. Pola noise harus deterministik (benih dari `id` gempa) agar tidak berubah tiap render.

**Gerak.** Garis tergambar sekali saat halaman dibuka (`stroke-dashoffset`, sekitar 1,6 detik). Ini satu-satunya animasi otomatis di seluruh web. Dengan `prefers-reduced-motion: reduce`, garis langsung tampil penuh. Bila data tidak ada, tampilkan garis datar dengan noise kecil dan teks "Belum ada data gempa terbaru".

**Aksesibilitas.** SVG diberi `role="img"` dan label, misalnya: "Ilustrasi jejak seismogram untuk gempa magnitudo 4,6".

## 6. Tata letak dan navigasi

**Header.** Nama "SIGAP Gempa" di kiri, menu horizontal di desktop (Beranda, Data gempa, Berita, Edukasi, Mitigasi, Penanggulangan, Tentang). Di HP menu menjadi tombol "Menu" yang membuka panel penuh. Menu aktif ditandai garis bawah 3 px berwarna Jejak.

**Footer.** Sumber data (BMKG), tautan Tentang, disclaimer singkat, atribusi peta (OpenStreetMap).

**Tautan lewati.** Tautan "Lewati ke konten" muncul saat fokus keyboard.

### 6.1 Beranda

```
┌────────────────────────────────────────────────────────────────┐
│ SIGAP Gempa   Beranda  Data gempa  Berita  Edukasi  …  [Menu] │
├────────────────────────────────────────────────────────────────┤
│ (latar kisi)                                                   │
│ Gempa terbaru, 6 Okt 2026, 04.38 WIB                           │
│                                                                │
│ 4,6   Gempa di laut selatan Kodi, Sumba Barat Daya             │
│                                                                │
│ ─────────────────P~~~~~~S/\/\/\_____________   (jejak)        │
│                                                                │
│ Kedalaman 19 km     Dirasakan II Kab. Sumba Barat Daya         │
│ [Tingkat: Ringan]   [Status tsunami]        [Lihat detail]    │
├────────────────────────────────────────────────────────────────┤
│ Gempa M 5,0 ke atas, 15 kejadian terakhir                      │
│ ┌──────────────────────────┐ ┌───────────────────────────────┐ │
│ │ Ringkasan: jumlah,       │ │            PETA               │ │
│ │ magnitudo tertinggi,     │ │                               │ │
│ │ wilayah terbanyak        │ │                               │ │
│ └──────────────────────────┘ └───────────────────────────────┘ │
├────────────────────────────────────────────────────────────────┤
│ Pelajari   Pengertian | Proses | Mitigasi | Penanggulangan     │
└────────────────────────────────────────────────────────────────┘
```

Catatan: angka magnitudo berada di kiri dan sejajar dengan baris pertama judul lokasi, bukan di tengah. Latar kisi hanya melingkupi blok hero.

### 6.2 Data gempa (`/gempa`)

Desktop: dua kolom (daftar di kiri, peta di kanan, menempel saat scroll). HP: dua tab "Daftar" dan "Peta".

```
[Tab: Terkini M 5,0+ | Dirasakan]
Filter: [Magnitudo ▾] [Tanggal ▾] [Cari wilayah…]   Urutkan: [Terbaru ▾]

▌6,2  Kuat    42 km timur laut Pulau Doi, Malut      14 Sep, 17.58 WIB
▌5,9  Sedang  68 km barat daya Calang, Aceh Jaya      4 Okt, 06.37 WIB
▌…
```

Pita warna 6 px di kiri tiap baris memakai warna tingkat. Satu baris memuat: magnitudo, nama tingkat, wilayah, waktu. Baris berupa tautan penuh ke halaman detail.

### 6.3 Detail gempa

Judul memuat lokasi, lalu angka magnitudo dan jejak seismogram ringkas. Di bawahnya: tabel field (waktu, koordinat, kedalaman, potensi, dirasakan), peta lokasi, dan gambar shakemap bila ada. Teks alternatif shakemap: "Peta guncangan gempa M … di …".

### 6.4 Halaman edukasi

- Kolom bacaan 68ch, daftar isi menempel di kiri pada desktop dan menjadi ringkasan yang bisa dilipat pada HP.
- Ilustrasi proses gempa memakai SVG sendiri. Gelombang P dan S memakai ulang `SeismogramTrace`.
- Tabel skala intensitas memakai pita warna dari skala magnitudo agar konsisten.
- Sumber ditaruh di akhir tiap halaman dalam daftar bertautan.

### 6.5 Mitigasi dan penanggulangan

- **Mitigasi:** tiga bagian *Sebelum*, *Saat*, *Sesudah*, masing-masing berupa daftar centang besar (target sentuh minimal 44 px). Centang tersimpan di browser. Urutan di sini adalah fase waktu, jadi penandaan fase boleh dipakai.
- **Tas siaga:** daftar barang dikelompokkan (dokumen, air dan makanan, obat, alat bantu).
- **Penanggulangan:** garis waktu vertikal dari tanggap darurat sampai pemulihan. Penomoran dipakai karena isinya memang urutan tahap.
- **Kontak darurat:** blok tetap dengan nomor yang bisa diketuk di HP (`tel:`). Verifikasi nomor sebelum rilis.

## 7. Komponen

| Komponen | Isi dan perilaku |
|---|---|
| `MagnitudeBadge` | Angka dengan koma desimal + nama tingkat; warna sesuai skala |
| `TsunamiStatus` | Menerjemahkan `statusTsunami`; hanya tampil bila statusnya jelas |
| `SeismogramTrace` | Lihat bagian 5 |
| `GempaRow` | Baris data dengan pita warna; seluruh baris dapat diklik |
| `FilterBar` | Kontrol filter; nilai tersimpan di URL |
| `GempaMap` | Leaflet; penanda berupa lingkaran berukuran dan berwarna sesuai tingkat; popup berisi ringkasan dan tautan detail; atribusi OSM terlihat |
| `StatTile` | Satu angka dengan keterangan; tidak lebih dari tiga per baris |
| `ChecklistFase` | Daftar centang dengan penyimpanan lokal |
| `KontakDarurat` | Nama lembaga, nomor, tombol telepon |
| `SumberKonten` | Daftar sumber bertautan di akhir halaman |
| `ChatWidget` | Lihat bagian 8 |
| `StateKosong`, `StateError`, `Skeleton` | Lihat bagian 9 |

Penanda peta: ukuran lingkaran naik dengan magnitudo, garis tepi Tinta 1,5 px, dan isi sesuai warna tingkat. Sediakan daftar teks setara di samping peta untuk pengguna yang tidak dapat memakai peta.

## 8. Chatbot

**Tombol pembuka.** Tombol bundar di kanan bawah, berlabel teks "Tanya gempa" di desktop dan ikon bergaris di HP. Warna Jejak, teks putih.

**Panel.**
- Desktop: lebar 360 px, tinggi hingga 560 px, menempel di kanan bawah.
- HP: lembar penuh dari bawah dengan tombol tutup yang jelas.
- Muncul dengan transisi singkat (200 ms) sebagai tanggapan atas aksi pengguna. Dengan reduced motion, langsung tampil.

**Isi.**
- Judul "Tanya seputar gempa" dan satu baris disclaimer: "Jawaban bukan pengganti informasi resmi BMKG atau BNPB."
- Gelembung pengguna: latar Jejak, teks putih, rata kanan. Gelembung asisten: latar Permukaan dengan garis halus, rata kiri.
- Saat menunggu jawaban, tampil jejak seismogram mini yang bergerak. Dengan reduced motion, tampil teks "Menyiapkan jawaban".
- Jawaban dari cadangan diberi keterangan kecil: "Jawaban dari panduan offline".
- Tiga saran pertanyaan bisa diklik: "Apa yang harus dilakukan saat gempa?", "Apa bedanya magnitudo dan intensitas?", "Apa isi tas siaga bencana?"
- Kolom input dengan batas 500 karakter dan penghitung yang muncul mendekati batas. Tombol "Kirim" dinonaktifkan saat kosong.

**Aksesibilitas.** Daftar pesan memakai `aria-live="polite"`. Fokus pindah ke input saat panel dibuka dan kembali ke tombol pembuka saat ditutup. Tombol Esc menutup panel.

## 9. State dan teks antarmuka

Gaya tulisan: kalimat biasa, kata kerja aktif, tanpa basa-basi. Nama aksi konsisten di seluruh alur.

| Situasi | Tampilan |
|---|---|
| Memuat data | Skeleton berbentuk baris dan kartu dengan garis datar; tanpa denyut bila reduced motion |
| Data kosong (hasil filter) | "Tidak ada gempa yang cocok dengan filter ini." + tombol "Hapus filter" |
| BMKG gagal dimuat | "Data BMKG belum bisa dimuat. Periksa koneksi, lalu coba lagi." + tombol "Muat ulang" |
| Chatbot memakai cadangan | "Layanan jawab otomatis sedang terbatas. Kamu tetap bisa bertanya, jawabannya dari panduan offline." |
| Chatbot gagal total | "Pertanyaan belum bisa dijawab sekarang. Coba lihat halaman Edukasi atau Mitigasi." |
| Pesan terlalu panjang | "Pesan maksimal 500 karakter. Persingkat pertanyaanmu." |
| Halaman tidak ada | "Halaman tidak ditemukan. Kembali ke Beranda atau lihat Data gempa." |
| Gempa tidak ada di daftar | "Gempa ini sudah tidak ada di daftar BMKG terbaru. Lihat daftar gempa terkini." |

Pesan error tidak meminta maaf berlebihan dan selalu menyebut apa yang bisa dilakukan berikutnya.

## 10. Gerak

- Hanya satu animasi otomatis: jejak seismogram di hero saat halaman dibuka.
- Selain itu gerak hanya sebagai respons aksi: panel chat terbuka, menu HP terbuka, daftar dilipat, hover pada tautan.
- Tidak ada animasi masuk (fade atau slide) per bagian dan tidak ada efek hover di setiap kartu.
- Semua gerak dimatikan atau disederhanakan dengan `prefers-reduced-motion: reduce`.

## 11. Aksesibilitas

- Kontras teks minimal 4,5 banding 1 (AA). Pasangan warna di bagian 2 sudah dihitung (2026-10-07) dan semuanya lolos; yang terendah Tinta di atas Kuat `#EE7B30` (4,94) dan Aman `#2F7D5B` di atas Permukaan (4,80). Hitung ulang bila ada warna yang diubah.
- Fokus keyboard selalu terlihat memakai `--fokus`.
- Target sentuh minimal 44 x 44 px.
- `<html lang="id">`, urutan judul berurutan, landmark `header`, `nav`, `main`, `footer`.
- Warna tidak pernah satu-satunya penanda: tingkat selalu bertuliskan nama, dan status tsunami selalu berupa kalimat yang disertai ikon.
- Semua gambar punya teks alternatif; SVG dekoratif memakai `aria-hidden`.
- Peta punya padanan berupa daftar teks.

## 12. Responsif

| Lebar | Perilaku |
|---|---|
| 360 sampai 639 px | Satu kolom; menu menjadi panel; peta dan daftar berupa tab; chat layar penuh |
| 640 sampai 1023 px | Satu atau dua kolom sesuai halaman; menu masih ringkas |
| 1024 px ke atas | Menu horizontal; daftar dan peta berdampingan; daftar isi edukasi menempel |
| 1280 px ke atas | Lebar konten dikunci 1200 px |

Tidak boleh ada scroll horizontal pada lebar 360 px.

## 13. Daftar periksa sebelum rilis

- [ ] Kontras diperiksa ulang jika ada warna yang diubah
- [ ] Seluruh tingkat magnitudo tampil dengan teks dan warna
- [ ] Hero menampilkan jejak yang benar untuk M kecil dan M besar
- [ ] Reduced motion diuji
- [ ] Navigasi keyboard dan pembaca layar diuji pada beranda dan chatbot
- [ ] Atribusi OpenStreetMap terlihat di semua peta
- [ ] Semua halaman konten mencantumkan sumber
- [ ] Nomor kontak darurat sudah diverifikasi