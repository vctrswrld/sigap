# Research: Fondasi Database dan Data Gempa BMKG

**Feature**: `002-fondasi-database-gempa`  
**Date**: 2026-10-15  
**Context**: Next.js 16.3.8 App Router + React 19.2.8 + TypeScript + Prisma 7.10.0 + PostgreSQL 16 (VPS Docker) + Tailwind CSS 4

---

## 1. Driver Adapter pada Prisma 7.10.0

### Decision
Wajib menginstal dan menggunakan `@prisma/adapter-pg` bersama paket driver `pg` (serta `@types/pg` pada devDependencies).

### Rationale
- Pada Prisma ORM versi 7 (`prisma-client` generator), driver adapter **bersifat wajib (required)** untuk seluruh SQL database provider (PostgreSQL, MySQL, SQLite).
- Klien Prisma diinstansiasi dengan menyuntikkan driver adapter PostgreSQL:
  ```typescript
  import { PrismaClient } from '@/generated/prisma/client';
  import { PrismaPg } from '@prisma/adapter-pg';
  import { Pool } from 'pg';

  const connectionString = process.env.DATABASE_URL;
  const pool = new Pool({ connectionString });
  const adapter = new PrismaPg(pool);

  export const prisma = new PrismaClient({ adapter });
  ```
- Klien hasil generate berada di `src/generated/prisma` (sesuai konfigurasi generator di `prisma/schema.prisma` dan telah terdaftar di `.gitignore`).

---

## 2. Konfigurasi Prisma CLI, Penghapusan `prisma7.config.ts`, & Environment

### Decision
Menghapus file `prisma7.config.ts` dan membuat `prisma.config.ts` resmi di root repositori dengan memuat variabel lingkungan secara eksplisit via `dotenv/config` dan `defineConfig` dari `prisma/config`.

### Rationale
- Prisma CLI versi 7 hanya mencari `prisma.config.ts` (bukan `prisma7.config.ts`) dan tidak lagi membaca `.env.local` secara implisit.
- `prisma.config.ts` adalah standar konfigurasi resmi Prisma 7:
  ```typescript
  import 'dotenv/config';
  import { defineConfig, env } from 'prisma/config';

  export default defineConfig({
    schema: 'prisma/schema.prisma',
    migrations: {
      path: 'prisma/migrations',
    },
    datasource: {
      url: env('DATABASE_URL'),
    },
  });
  ```
- File `.env` memuat `DATABASE_URL="postgresql://user:password@localhost:5433/sigap?schema=public"` (melalui SSH tunnel lokal port 5433 ke PostgreSQL VPS).

---

## 3. Struktur Respon API BMKG & Strategi Parsing

### Decision
Mendukung 3 endpoint resmi BMKG via variabel lingkungan `BMKG_BASE_URL` (default: `https://data.bmkg.go.id/DataMKG/TEWS`):
1. `autogempa.json`: Gempa terkini terbaru. Objek tunggal pada path `Infogempa.gempa`.
2. `gempaterkini.json`: Daftar gempa dengan magnitudo M ≥ 5.0. Array objek pada path `Infogempa.gempa`.
3. `gempadirasakan.json`: Daftar gempa yang dilaporkan dirasakan masyarakat. Array objek pada path `Infogempa.gempa`.

### Karakteristik & Parsing Payload BMKG:
- **Kunci Unik Deduplikasi**:
  - `bmkgKey = `${DateTime}_${Coordinates}`` menggunakan nilai teks mentah dari BMKG.
  - Contoh konkret: `"2026-10-15T07:30:00+00:00_-6.78,105.12"`.
  - Menjamin idempotensi: pembukaan halaman berulang kali menjalankan operasi *upsert* tanpa menduplikasi baris di database.
- **Transformasi Tipe Data Numerik & Nama Kolom Standar**:
  - `Coordinates`: Format `"lat,lon"` dipisah menjadi `lat: Float` dan `lon: Float`.
  - `Magnitude`: Format teks dikonversi ke `magnitudo: Float`.
  - `Kedalaman`: Format teks berakhiran `" km"` dibersihkan lalu dikonversi ke `kedalamanKm: Float`.
  - `DateTime`: Teks waktu mentah dikonversi menjadi objek `Date` UTC untuk kolom `waktuUtc: DateTime`.
  - `updatedAt`: Diperbarui otomatis oleh Prisma `@updatedAt` setiap kali baris di-upsert, digunakan sebagai waktu sinkronisasi terakhir untuk banner status.
- **Kondisi Khusus Kategori `gempadirasakan.json`**:
  - Tidak memiliki atribut `Potensi` dan `Shakemap`.
  - Field `potensi` dan `shakemap` pada database diset `null`, dan `statusTsunami` diset `TIDAK_DINYATAKAN`.

---

## 4. Alur Sinkronisasi Ber-cache (60s) & Halaman Dinamis

### Decision
Sinkronisasi BMKG (fetch + upsert) dibungkus dalam **satu fungsi ber-cache** (`syncBMKGWithCache`) menggunakan Next.js `unstable_cache` dengan `revalidate: 60`. Halaman `/gempa` dan `/gempa/[id]` disetel `export const dynamic = 'force-dynamic'`.

### Rationale
- **Pembatasan Frekuensi Eksekusi**:
  - Jika tidak dibungkus cache, setiap request render halaman akan memicu 3 fetch ke BMKG dan serangkaian query upsert ke PostgreSQL.
  - Dengan cache 60 detik, sinkronisasi hanya dieksekusi maksimal 1 kali per menit.
- **Pencegahan Error Saat Build**:
  - Secara default Next.js mencoba melakukan static prerendering pada rute. Jika halaman tidak disetel dinamis, `npm run build` akan mencoba mengeksekusi koneksi ke database.
  - Dengan `export const dynamic = 'force-dynamic'`, `next build` tidak menyentuh database sama sekali.
- **Pengujian Fallback yang Aman**:
  - Pengujian kegagalan BMKG dilakukan dengan mengubah `BMKG_BASE_URL` di `.env` ke domain salah.
  - Opsi memutus koneksi internet dilarang karena akan memutus SSH tunnel lokal ke PostgreSQL VPS pada port 5433.
