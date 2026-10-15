# Data Model: Fondasi Database dan Data Gempa BMKG

**Feature**: `002-fondasi-database-gempa`  
**Date**: 2026-10-15  
**ORM**: Prisma 7.10.0  
**Database**: PostgreSQL 16

---

## 1. Definisi Skema Prisma (`prisma/schema.prisma`)

```prisma
datasource db {
  provider = "postgresql"
}

generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

enum StatusTsunami {
  AMAN
  BERPOTENSI
  TIDAK_DINYATAKAN
}

enum JenisGempa {
  TERBARU
  TERKINI_M5
  DIRASAKAN
}

model Gempa {
  id            String        @id @default(cuid())
  bmkgKey       String        @unique // Format: "${DateTime}_${Coordinates}" (menggunakan nilai mentah BMKG)
  waktuUtc      DateTime      // Waktu gempa UTC terstandarisasi
  tanggal       String        // Format tanggal BMKG (contoh: "15 Okt 2026")
  jam           String        // Format jam BMKG (contoh: "14:30:15 WIB")
  lat           Float         // Garis lintang numerik (contoh: -6.78)
  lon           Float         // Garis bujur numerik (contoh: 105.12)
  magnitudo     Float         // Kekuatan magnitudo numerik (contoh: 5.4)
  kedalamanKm   Float         // Kedalaman dalam kilometer numerik (contoh: 10.0)
  wilayah       String        // Keterangan wilayah pusat gempa
  potensi       String?       // Keterangan teks potensi dari BMKG (null untuk gempa dirasakan)
  statusTsunami StatusTsunami @default(TIDAK_DINYATAKAN)
  dirasakan     String?       // Informasi wilayah & MMI (null jika tidak dirasakan)
  shakemap      String?       // Nama berkas / tautan gambar shakemap (null untuk gempa dirasakan)
  jenis         JenisGempa    @default(TERBARU)
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt // Waktu pembaruan/sinkronisasi terakhir (untuk banner status)

  @@index([waktuUtc(sort: Desc)])
  @@index([jenis])
  @@index([magnitudo])
  @@map("gempa")
}

model Berita {
  id          String   @id @default(cuid())
  judul       String
  slug        String   @unique
  ringkasan   String
  konten      String   @db.Text
  gambar      String?
  sumber      String?
  publishedAt DateTime @default(now())
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([publishedAt(sort: Desc)])
  @@map("berita")
}
```

---

## 2. Rincian Entitas & Validasi

### Entitas `Gempa`
| Kolom | Tipe | Nullable | Deskripsi & Aturan Validasi |
|-------|------|----------|-----------------------------|
| `id` | String (CUID) | Tidak | Kunci primer entitas database. |
| `bmkgKey` | String | Tidak | Kunci deduplikasi unik: `${DateTime}_${Coordinates}` langsung dari teks mentah BMKG. Contoh: `"2026-10-15T07:30:00+00:00_-6.78,105.12"`. |
| `waktuUtc` | DateTime | Tidak | Waktu kejadian gempa terstandarisasi ISO UTC. Diindeks untuk pengurutan kronologis. |
| `tanggal` | String | Tidak | Label tanggal asli BMKG (contoh: "15 Okt 2026"). |
| `jam` | String | Tidak | Label jam asli BMKG (contoh: "14:30:15 WIB"). |
| `lat` | Float | Tidak | Lintang episentrum gempa (-90.0 s.d. 90.0). |
| `lon` | Float | Tidak | Bujur episentrum gempa (-180.0 s.d. 180.0). |
| `magnitudo` | Float | Tidak | Nilai magnitudo numerik (Float > 0). |
| `kedalamanKm` | Float | Tidak | Kedalaman pusat gempa dalam kilometer numerik (Float >= 0). |
| `wilayah` | String | Tidak | Keterangan wilayah pusat gempa dari BMKG. |
| `potensi` | String | Ya | Keterangan potensi tsunami asli BMKG. Bernilai `null` untuk gempa dirasakan. |
| `statusTsunami`| Enum | Tidak | Klasifikasi potensi: `AMAN`, `BERPOTENSI`, atau `TIDAK_DINYATAKAN`. |
| `dirasakan` | String | Ya | Daftar wilayah yang merasakan beserta skala MMI. |
| `shakemap` | String | Ya | Berkas/URL peta guncangan. Bernilai `null` untuk gempa dirasakan. |
| `jenis` | Enum | Tidak | Kategori data BMKG: `TERBARU`, `TERKINI_M5`, atau `DIRASAKAN`. |
| `createdAt` | DateTime | Tidak | Waktu pertama kali baris dibuat di database. |
| `updatedAt` | DateTime | Tidak | Waktu sinkronisasi/pembaruan terakhir data gempa (digunakan sebagai penanda banner waktu pembaruan). |

### Entitas `Berita` (Fondasi Fitur Mendatang)
| Kolom | Tipe | Nullable | Deskripsi |
|-------|------|----------|-----------|
| `id` | String (CUID) | Tidak | Kunci primer berita. |
| `judul` | String | Tidak | Judul artikel berita kebencanaan. |
| `slug` | String | Tidak | Penanda URL unik ramah SEO. |
| `ringkasan` | String | Tidak | Cuplikan isi berita. |
| `konten` | String (Text) | Tidak | Isi lengkap artikel berita. |
| `gambar` | String | Ya | Tautan gambar sampul berita. |
| `sumber` | String | Ya | Media rilis/sumber artikel. |
| `publishedAt` | DateTime | Tidak | Waktu publikasi berita. |

---

## 3. Aturan Konversi Data (Parsing Pipeline)

1. **Kunci Deduplikasi (`bmkgKey`)**:
   - Nilai mentah `DateTime` dari BMKG (misal: `"2026-10-15T07:30:00+00:00"`) digabungkan langsung dengan nilai mentah `Coordinates` (misal: `"-6.78,105.12"`).
   - Format: `${DateTime}_${Coordinates}` $\rightarrow$ `"2026-10-15T07:30:00+00:00_-6.78,105.12"`.

2. **Koordinat (`lat` & `lon`)**:
   - Sumber BMKG: `"Coordinates": "-6.78,105.12"` (atau `" -6.78, 105.12 "`).
   - Ekstraksi: String dipisah tanda koma `,`, masing-masing elemen dibersihkan (`trim`) dan diubah dengan `parseFloat`.
   - Hasil: `lat = -6.78`, `lon = 105.12`.

3. **Magnitudo (`magnitudo`)**:
   - Sumber BMKG: `"Magnitude": "5.3"`.
   - Ekstraksi: Mengganti koma dengan titik bila ada, lalu `parseFloat("5.3")`.
   - Hasil: `magnitudo = 5.3`.

4. **Kedalaman (`kedalamanKm`)**:
   - Sumber BMKG: `"Kedalaman": "10 km"`.
   - Ekstraksi: String `" km"` dihilangkan via regex `/\s*km/i`, lalu `parseFloat("10")`.
   - Hasil: `kedalamanKm = 10.0`.

5. **Waktu Kejadian (`waktuUtc`)**:
   - Sumber BMKG: `"DateTime": "2026-10-15T07:30:00+00:00"`.
   - Ekstraksi: Dikonversi ke objek `new Date(DateTime)`.
   - Hasil: Kolom PostgreSQL `timestamp with time zone`.

6. **Status Tsunami (`statusTsunami`)**:
   - Jika teks `Potensi` mengandung `"tidak berpotensi tsunami"` $\rightarrow$ `StatusTsunami.AMAN`.
   - Jika teks `Potensi` mengandung `"berpotensi tsunami"` $\rightarrow$ `StatusTsunami.BERPOTENSI`.
   - Jika teks `Potensi` kosong/null atau tidak menyatakan keterangan (misal pada gempa dirasakan) $\rightarrow$ `StatusTsunami.TIDAK_DINYATAKAN`.
