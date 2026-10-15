# Quickstart & Manual Testing Guide: Fondasi Database dan Data Gempa BMKG

**Feature**: `002-fondasi-database-gempa`  
**Date**: 2026-10-15

Panduan ini berisi langkah-langkah penyiapan lingkungan dan skenario pengujian manual untuk memverifikasi fitur secara menyeluruh (*end-to-end*).

---

## 1. Prasyarat & Penyiapan Lingkungan

### A. Membuka SSH Tunnel ke PostgreSQL VPS
Pastikan PostgreSQL 16 pada VPS (Docker) dapat diakses melalui SSH tunnel lokal pada port 5433:
```bash
ssh -N -L 5433:localhost:5432 <user>@<vps_ip>
```
> **PERINGATAN**: Jangan memutus koneksi internet selama sesi pengembangan dan pengujian, karena pemutusan internet akan memutus koneksi SSH tunnel ke database PostgreSQL.

### B. Konfigurasi Variabel Lingkungan (`.env`)
Pastikan file `.env` (bukan `.env.local`) memuat kredensial koneksi database dan basis URL BMKG:
```env
DATABASE_URL="postgresql://<db_user>:<db_password>@localhost:5433/<db_name>?schema=public"
BMKG_BASE_URL="https://data.bmkg.go.id/DataMKG/TEWS"
```

### C. Hapus `prisma7.config.ts` dan Siapkan `prisma.config.ts`
Prisma CLI versi 7 hanya membaca konfigurasi dari `prisma.config.ts`. Hapus file `prisma7.config.ts` lama:
```bash
# Hapus file prisma7.config.ts lama
rm prisma7.config.ts
```
Pastikan `prisma.config.ts` sudah ada dengan isi konfigurasi resmi Prisma 7:
```typescript
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  datasource: {
    url: env("DATABASE_URL"),
  },
});
```

### D. Instalasi Dependensi Wajib (Driver Adapter Prisma 7)
```bash
npm install @prisma/adapter-pg pg dotenv
npm install -D @types/pg
```

### E. Menjalankan Migrasi & Menghasilkan Prisma Client
```bash
npx prisma migrate dev --name init_gempa_berita
npx prisma generate
```

---

## 2. Skenario Pengujian Manual

### Skenario 1: Sinkronisasi Awal Data BMKG ke Database & Verifikasi Caching (Maks. 1x per 60 Detik)
1. Jalankan aplikasi: `npm run dev`.
2. Buka peramban ke `http://localhost:3000/gempa`.
3. Buka terminal baru dan jalankan `npx prisma studio`.
4. **Hasil yang Diharapkan**:
   - Halaman `/gempa` menampilkan gempa terbaru, daftar M5.0+, dan daftar gempa dirasakan.
   - Pada Prisma Studio, tabel `gempa` terisi data dari ketiga endpoint BMKG.
   - Kolom `lat`, `lon`, `magnitudo`, dan `kedalamanKm` bernilai numerik murni (misal: `kedalamanKm = 10` tanpa satuan teks `"km"`).
   - Kolom `bmkgKey` terisi format mentah BMKG, contoh: `"2026-10-15T07:30:00+00:00_-6.78,105.12"`.
   - Kolom `statusTsunami` bernilai `AMAN`, `BERPOTENSI`, atau `TIDAK_DINYATAKAN`.
   - Sinkronisasi dibungkus fungsi cache revalidasi 60 detik sehingga pembukaan halaman dalam rentang 60 detik tidak memicu query sinkronisasi ulang ke BMKG.

### Skenario 2: Uji Idempotensi / Pencegahan Baris Duplikat
1. Segarkan (*refresh*) halaman `http://localhost:3000/gempa` sebanyak 3–5 kali secara berturut-turut.
2. Periksa kembali tabel `gempa` di Prisma Studio.
3. **Hasil yang Diharapkan**:
   - Jumlah baris data di database tidak bertambah.
   - Kolom `bmkgKey` (`${DateTime}_${Coordinates}`) berhasil mencegah data duplikat melalui operasi *upsert*.

### Skenario 3: Uji Ketahanan & Fallback Saat BMKG Tidak Tersedia (Ubah BMKG_BASE_URL)
1. Buka file `.env`, lalu ubah nilai `BMKG_BASE_URL` ke domain yang sengaja disalahkan (misalnya untuk menyimulasikan BMKG down):
   ```env
   BMKG_BASE_URL="https://invalid-bmkg-domain-test.local/DataMKG/TEWS"
   ```
   *(Catatan: Jangan mematikan koneksi internet laptop/PC karena SSH tunnel ke port 5433 akan ikut terputus).*
2. Segarkan halaman `http://localhost:3000/gempa`.
3. **Hasil yang Diharapkan**:
   - Halaman tidak mengalami *crash* atau galat unhandled 500.
   - Data gempa terakhir dari database PostgreSQL tetap tampil secara utuh.
   - Muncul banner informatif berbahasa Indonesia: *"Menampilkan data tersimpan terakhir — BMKG sedang tidak dapat dijangkau"*, disertai keterangan waktu pembaruan terakhir yang diambil dari kolom `updatedAt`.
4. Kembalikan `BMKG_BASE_URL` di file `.env` ke URL asli setelah pengujian selesai.

### Skenario 4: Memeriksa Halaman Detail Gempa Dinamis (`/gempa/[id]`)
1. Dari halaman daftar, klik salah satu kartu gempa berkekuatan M5.0+.
2. Periksa kelengkapan informasi:
   - Koordinat presisi (`lat` dan `lon`)
   - Waktu kejadian dalam WIB (dihitung dari `waktuUtc`)
   - Status potensi tsunami (`AMAN`, `BERPOTENSI`, atau `TIDAK_DINYATAKAN`)
   - Gambar peta guncangan (*shakemap*)
3. Kembali ke daftar, lalu klik salah satu kartu **Gempa Dirasakan**.
4. **Hasil yang Diharapkan**:
   - Halaman detail menampilkan daftar wilayah dan skala intensitas MMI.
   - Ketiadaan peta guncangan ditangani secara elegan (tidak ada gambar pecah atau broken image).
5. Pastikan `export const dynamic = 'force-dynamic'` terpasang di `src/app/gempa/page.tsx` dan `src/app/gempa/[id]/page.tsx` sehingga perintah `npm run build` berjalan mulus tanpa mencoba menyambung ke database saat proses build.
