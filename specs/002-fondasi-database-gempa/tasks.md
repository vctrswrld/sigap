# Tasks: Fondasi Database dan Data Gempa BMKG

**Feature**: `002-fondasi-database-gempa`  
**Input**: Implementation plan from `specs/002-fondasi-database-gempa/plan.md`  
**Spec**: `specs/002-fondasi-database-gempa/spec.md`  
**Data Model**: `specs/002-fondasi-database-gempa/data-model.md`

---

## Phase 1: Setup (Konfigurasi & Dependensi)

**Purpose**: Penyiapan dependensi dan konfigurasi awal proyek

- [ ] T001 Hapus berkas `prisma7.config.ts` dan buat berkas konfigurasi resmi Prisma 7 di `prisma.config.ts` menggunakan `dotenv/config` dan `defineConfig` dari `prisma/config`
- [ ] T002 Instal paket dependensi driver adapter PostgreSQL Prisma 7 (`@prisma/adapter-pg`, `pg`, `dotenv` di dependencies; `@types/pg` di devDependencies) di `package.json`
- [ ] T003 [P] Konfigurasi variabel lingkungan koneksi database dan sumber BMKG (`DATABASE_URL="postgresql://...:5433/sigap?schema=public"` dan `BMKG_BASE_URL="https://data.bmkg.go.id/DataMKG/TEWS"`) di `.env`

---

## Phase 2: Foundational (Skema Database, Migrasi & Klien Prisma)

**Purpose**: Fondasi persistensi data yang menjadi prasyarat sebelum implementasi antarmuka dan layanan

**⚠️ CRITICAL**: Seluruh tugas fondasi ini wajib selesai sebelum pengerjaan user story dimulai.

- [ ] T004 Definisikan skema model `Gempa` (kolom: `id` CUID, `bmkgKey` unik, `waktuUtc` DateTime, `tanggal` String, `jam` String, `lat` Float, `lon` Float, `magnitudo` Float, `kedalamanKm` Float, `wilayah` String, `potensi` String?, `statusTsunami` Enum `AMAN` / `BERPOTENSI` / `TIDAK_DINYATAKAN`, `dirasakan` String?, `shakemap` String?, `jenis` Enum `TERBARU` / `TERKINI_M5` / `DIRASAKAN`, `createdAt` DateTime, `updatedAt` DateTime; index pada `waktuUtc`, `jenis`, `magnitudo`) dan model `Berita` (kolom: `id` CUID, `judul` String, `slug` String unik, `ringkasan` String, `konten` Text, `gambar` String?, `sumber` String?, `publishedAt` DateTime, `createdAt` DateTime, `updatedAt` DateTime) di `prisma/schema.prisma`
- [ ] T005 Jalankan migrasi basis data lokal melalui SSH tunnel dengan perintah `npx prisma migrate dev --name init_gempa_berita` untuk menghasilkan berkas migrasi di `prisma/migrations/`
- [ ] T006 Hasilkan klien Prisma Client ke target direktori `src/generated/prisma` menggunakan perintah `npx prisma generate`
- [ ] T007 [P] Implementasikan singleton Prisma Client dengan driver adapter `@prisma/adapter-pg` dan connection pool `pg.Pool` di `src/lib/prisma.ts`
- [ ] T008 [P] Definisikan interface TypeScript untuk tipe data mentah respons BMKG (`BMKGRawGempa`, `BMKGAutogempaResponse`, `BMKGListResponse`), tipe entitas domain `GempaModel`, dan enum `StatusTsunami` / `JenisGempa` di `src/types/gempa.ts`

**Checkpoint**: Skema database dan Prisma Client singleton siap digunakan — pengerjaan User Story dapat dimulai.

---

## Phase 3: User Story 1 - Menjelajahi Daftar Gempa Terkini & Riwayat Tersimpan (Priority: P1) 🎯 MVP

**Goal**: Pengunjung dapat melihat daftar gempa terbaru, gempa M5.0+, dan gempa dirasakan yang tersinkronisasi otomatis dari BMKG ke PostgreSQL tanpa duplikasi data.

**Independent Test**: Buka `http://localhost:3000/gempa`, verifikasi daftar gempa dari ketiga kategori muncul dengan benar, dan periksa Prisma Studio (`npx prisma studio`) untuk memastikan data tersimpan dengan tipe numerik (`lat`, `lon`, `magnitudo`, `kedalamanKm`) serta baris duplikat tidak bertambah saat halaman di-refresh.

### Implementation for User Story 1

- [ ] T009 [US1] Implementasikan fungsi parser data mentah BMKG yang membersihkan teks menjadi angka numerik (`lat` Float, `lon` Float dari `Coordinates`, `magnitudo` Float dari `Magnitude`, `kedalamanKm` Float dari `Kedalaman` tanpa satuan `" km"`), membentuk kunci deduplikasi mentah `bmkgKey = "${DateTime}_${Coordinates}"` (contoh: `"2026-10-15T07:30:00+00:00_-6.78,105.12"`), serta memetakan status tsunami (`AMAN`, `BERPOTENSI`, `TIDAK_DINYATAKAN`) di `src/lib/bmkg.ts`
- [ ] T010 [US1] Implementasikan fungsi sinkronisasi `executeBMKGSyncToDatabase` untuk mengambil data dari endpoint `autogempa.json`, `gempaterkini.json`, dan `gempadirasakan.json` lalu melakukan operasi `prisma.gempa.upsert` berbasis `bmkgKey` ke PostgreSQL di `src/lib/bmkg.ts`
- [ ] T011 [US1] Bungkus fungsi sinkronisasi BMKG dalam cache revalidasi 60 detik (`syncBMKGWithCache` menggunakan `unstable_cache` dengan `revalidate: 60`) di `src/lib/bmkg.ts` agar sinkronisasi fetch dan upsert hanya berjalan maksimal 1 kali per menit
- [ ] T012 [P] [US1] Buat komponen kartu gempa di `src/components/gempa/GempaCard.tsx` menggunakan komponen reusable `Card` dan `Badge` untuk menampilkan waktu (WIB), magnitudo, kedalaman, wilayah, dan badge potensi tsunami / MMI
- [ ] T013 [P] [US1] Buat komponen pemilih kategori gempa di `src/components/gempa/FilterBar.tsx` menggunakan komponen reusable `Tabs` untuk berpindah antara kategori Terbaru, M5.0+, dan Dirasakan
- [ ] T014 [P] [US1] Buat komponen tabel daftar gempa di `src/components/gempa/GempaTable.tsx` untuk menampilkan daftar riwayat gempa secara kronologis
- [ ] T015 [US1] Implementasikan Route Handler `GET /api/gempa` dengan filter query `kategori` (`terbaru`, `terkini_m5`, `dirasakan`, `semua`) yang mengambil data dari database via Prisma di `src/app/api/gempa/route.ts`
- [ ] T016 [US1] Implementasikan React Server Component halaman daftar gempa di `src/app/gempa/page.tsx` dengan konfigurasi `export const dynamic = 'force-dynamic'`, memanggil `syncBMKGWithCache()` dan merender daftar gempa menggunakan `GempaCard`, `FilterBar`, dan `GempaTable`

**Checkpoint**: User Story 1 selesai sebagai MVP — daftar gempa terkini tersinkronisasi ke basis data dan tampil di antarmuka.

---

## Phase 4: User Story 2 - Memeriksa Informasi Rinci Satu Kejadian Gempa (Priority: P2)

**Goal**: Pengunjung dapat membuka halaman detail suatu kejadian gempa untuk melihat seluruh atribut teknis secara lengkap, termasuk peta guncangan (jika ada) dan daftar wilayah yang merasakan beserta skala MMI.

**Independent Test**: Buka salah satu gempa M5.0+ pada `/gempa/[id]` dan pastikan peta guncangan serta status potensi tsunami muncul; buka salah satu gempa dirasakan dan pastikan daftar wilayah MMI tampil dengan ketiadaan peta guncangan ditangani tanpa error gambar rusak.

### Implementation for User Story 2

- [ ] T017 [P] [US2] Buat komponen penampil peta guncangan di `src/components/gempa/ShakemapImage.tsx` yang menampilkan gambar shakemap dari BMKG dan menangani kondisi nilai `shakemap: null` (pada gempa dirasakan) secara elegan tanpa menampilkan broken image
- [ ] T018 [P] [US2] Buat komponen kartu ringkasan parameter teknis di `src/components/gempa/StatCard.tsx` untuk menyajikan magnitudo, kedalaman dalam km, dan koordinat lintang/bujur
- [ ] T019 [US2] Implementasikan fungsi kueri data detail gempa tunggal berdasarkan identifier `id` atau `bmkgKey` di `src/lib/bmkg.ts`
- [ ] T020 [US2] Implementasikan React Server Component halaman detail gempa di `src/app/gempa/[id]/page.tsx` dengan konfigurasi `export const dynamic = 'force-dynamic'`, mengekstrak `params` (bertipe `Promise<{ id: string }>`), memuat data gempa via Prisma, dan merender parameter teknis, daftar skala MMI, status potensi tsunami, serta `ShakemapImage`

**Checkpoint**: User Story 1 dan User Story 2 berfungsi secara utuh dan independen.

---

## Phase 5: User Story 3 - Akses Informasi Gempa Saat BMKG Tidak Tersedia (Priority: P3)

**Goal**: Pengunjung tetap dapat melihat data riwayat gempa terakhir yang tersimpan di PostgreSQL ketika API BMKG mengalami gangguan atau tidak dapat dijangkau, disertai banner waktu pembaruan terakhir.

**Independent Test**: Ubah `BMKG_BASE_URL` di `.env` ke domain salah (tanpa mematikan internet), buka `http://localhost:3000/gempa`, dan pastikan halaman tetap menyajikan data tersimpan dari PostgreSQL disertai banner informasi fallback dan waktu `updatedAt` terakhir.

### Implementation for User Story 3

- [ ] T021 [US3] Implementasikan penanganan galat dan fallback di `src/lib/bmkg.ts`: tangkap error saat fetch BMKG gagal/timeout, ambil data tersimpan terakhir dari database (`prisma.gempa.findMany({ orderBy: { waktuUtc: 'desc' } })`), dan kembalikan flag `isFallback: true` beserta waktu `updatedAt` dari catatan terbaru
- [ ] T022 [US3] Buat komponen banner fallback di `src/components/gempa/FallbackBanner.tsx` yang menampilkan pemberitahuan berbahasa Indonesia *"Menampilkan data tersimpan terakhir — BMKG sedang tidak dapat dijangkau"* beserta indikator waktu sinkronisasi terakhir
- [ ] T023 [US3] Integrasikan `FallbackBanner` ke dalam halaman `src/app/gempa/page.tsx` saat `isFallback === true`, serta sediakan tampilan *empty state* ramah pengguna berbahasa Indonesia jika basis data masih dalam kondisi kosong

**Checkpoint**: Seluruh User Story (US1, US2, US3) telah lengkap dan tangguh terhadap kegagalan jaringan eksternal.

---

## Phase 6: Polish & Validasi Lintas Fitur

**Purpose**: Verifikasi menyeluruh terhadap standar kualitas kode, tipe data, dan skenario pengujian

- [ ] T024 Jalankan seluruh skenario pengujian manual sesuai panduan di `specs/002-fondasi-database-gempa/quickstart.md` (uji sinkronisasi awal, uji pencegahan duplikasi data, dan uji fallback via `BMKG_BASE_URL` salah)
- [ ] T025 [P] Jalankan pengecekan tipe statis TypeScript (`npx tsc --noEmit`) dan ESLint (`npm run lint`) untuk memastikan tidak ada penggunaan tipe `any` dan semua fungsi memiliki explicit return type
- [ ] T026 Jalankan perintah kompilasi produksi `npm run build` untuk memverifikasi proses build Next.js 16 berhasil tanpa mencoba membuka koneksi ke database saat build

---

## Dependencies & Execution Order

### Phase Dependencies
- **Setup (Phase 1)**: Tidak memiliki dependensi — dapat segera dimulai.
- **Foundational (Phase 2)**: Bergantung pada Phase 1 — **MENGUNCI (BLOCKING)** seluruh user story.
- **User Story 1 (Phase 3 - P1)**: Bergantung pada Phase 2 — merupakan MVP sistem.
- **User Story 2 (Phase 4 - P2)**: Bergantung pada Phase 2 & model gempa dari Phase 3.
- **User Story 3 (Phase 5 - P3)**: Bergantung pada Phase 2 & alur sinkronisasi dari Phase 3.
- **Polish (Phase 6)**: Bergantung pada penyelesaian seluruh user story.

### Parallel Opportunities
- **Phase 1**: `T003` (.env setup) dapat dikerjakan paralel dengan `T001` dan `T002`.
- **Phase 2**: `T007` (`prisma.ts`) dan `T008` (`types/gempa.ts`) dapat dikerjakan paralel setelah `T006` selesai.
- **Phase 3 (US1)**: Komponen antarmuka `T012` (`GempaCard.tsx`), `T013` (`FilterBar.tsx`), dan `T014` (`GempaTable.tsx`) dapat dikerjakan secara paralel karena berada pada file yang berbeda.
- **Phase 4 (US2)**: Komponen `T017` (`ShakemapImage.tsx`) dan `T018` (`StatCard.tsx`) dapat dikerjakan secara paralel.
- **Phase 6**: Pengecekan tipe dan lint `T025` dapat dijalankan paralel dengan pengujian manual `T024`.

---

## Implementation Strategy (MVP First)

1. **Langkah 1**: Selesaikan Phase 1 (Setup) dan Phase 2 (Foundational). Skema dan database siap.
2. **Langkah 2**: Implementasikan Phase 3 (User Story 1). Buka `/gempa` dan lakukan verifikasi data BMKG tersimpan tanpa duplikasi $\rightarrow$ **Tahap MVP Selesai**.
3. **Langkah 3**: Lanjutkan ke Phase 4 (User Story 2) untuk mengaktifkan halaman detail `/gempa/[id]`.
4. **Langkah 4**: Lanjutkan ke Phase 5 (User Story 3) untuk menguji ketahanan sistem saat BMKG gagal dijangkau via perubahan `BMKG_BASE_URL`.
5. **Langkah 5**: Jalankan Phase 6 (Polish & Build test) untuk verifikasi akhir sebelum deployment.

