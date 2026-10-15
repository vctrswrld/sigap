# Feature Specification: Halaman Data Gempa BMKG

**Feature Branch**: `001-halaman-data-gempa`

**Created**: 2026-10-13

**Status**: Draft

**Input**: User description: "Pengguna dapat melihat gempa terbaru, gempa M5.0+ (terkini), dan gempa
dirasakan dari BMKG. Data diperbarui saat halaman diakses dan disimpan ke PostgreSQL tanpa
duplikat. Tiap gempa menampilkan waktu, magnitudo, kedalaman, lokasi, dan wilayah yang
merasakan (skala MMI). Jika BMKG tidak bisa diakses, tampilkan data terakhir dari database."

---

## User Scenarios & Testing *(mandatory)*

<!--
  User stories diprioritaskan sebagai user journeys berurutan dari yang paling kritis.
  Setiap story harus INDEPENDENTLY TESTABLE dan menghasilkan nilai tersendiri.
-->

### User Story 1 — Melihat Daftar Gempa Terbaru (Priority: P1)

Sebagai warga Indonesia yang ingin tahu situasi gempa terkini, saya membuka halaman Data Gempa
SIGAP dan langsung melihat daftar gempa terbaru yang direkam BMKG — mencakup 15 gempa paling
baru tanpa batasan magnitudo — sehingga saya bisa menilai apakah ada aktivitas seismik di
sekitar saya.

**Why this priority**: Ini adalah kebutuhan dasar halaman: pengguna membuka halaman untuk
melihat ada-tidaknya gempa. Tanpa daftar gempa, halaman tidak memiliki nilai sama sekali.

**Independent Test**: Dapat diuji sepenuhnya dengan membuka `/gempa` dan memverifikasi
bahwa daftar gempa muncul dengan data yang dapat dibaca. Tidak bergantung pada story lain.

**Acceptance Scenarios**:

1. **Given** pengguna membuka `/gempa`, **When** BMKG dapat diakses, **Then** halaman
   menampilkan daftar minimal 1 gempa terbaru dengan waktu (WIB/WITA/WIT), magnitudo (SR),
   kedalaman (km), dan lokasi yang terbaca jelas.
2. **Given** pengguna melihat daftar gempa, **When** ia membaca satu entri gempa,
   **Then** ia dapat melihat: waktu kejadian, magnitudo dalam satuan SR, kedalaman dalam km,
   dan nama lokasi gempa (koordinat + keterangan wilayah dari BMKG).
3. **Given** pengguna membuka `/gempa`, **When** BMKG **tidak** dapat diakses,
   **Then** halaman tetap menampilkan daftar gempa terakhir yang tersimpan di database
   disertai notifikasi "Menampilkan data terakhir — BMKG tidak dapat dijangkau."
4. **Given** BMKG tidak dapat diakses **dan** database kosong, **When** pengguna membuka
   `/gempa`, **Then** halaman menampilkan pesan "Data gempa belum tersedia."

---

### User Story 2 — Melihat Gempa Signifikan M5.0+ (Priority: P2)

Sebagai pengguna yang ingin fokus pada gempa berpotensi merusak, saya dapat beralih ke
tab/filter "Gempa M5.0+" dan melihat gempa terkini dengan magnitudo ≥ 5,0 SR — lengkap
dengan label potensi tsunami jika relevan — sehingga saya dapat mengambil keputusan
kesiapsiagaan yang lebih tepat.

**Why this priority**: Gempa M5.0+ adalah informasi paling kritis untuk keselamatan.
Namun story ini bergantung pada story 1 (daftar gempa sudah ditampilkan) dan hanya menambahkan
filter/tampilan, bukan infrastruktur baru.

**Independent Test**: Dapat diuji dengan memilih tab "M5.0+" dan memverifikasi bahwa
hanya gempa ≥ 5,0 SR yang muncul, tanpa perlu fitur story lain berjalan.

**Acceptance Scenarios**:

1. **Given** pengguna berada di halaman gempa, **When** ia memilih tab/filter "M5.0+",
   **Then** hanya gempa dengan magnitudo ≥ 5,0 SR yang ditampilkan dari sumber BMKG
   (endpoint gempa-m50.json BMKG).
2. **Given** pengguna melihat daftar gempa M5.0+, **When** suatu gempa memiliki potensi
   tsunami, **Then** gempa tersebut ditandai dengan label/badge "Berpotensi Tsunami"
   yang mencolok.
3. **Given** pengguna melihat daftar gempa M5.0+, **When** tidak ada gempa M5.0+ dalam
   periode terkini dari BMKG, **Then** halaman menampilkan pesan informatif
   "Tidak ada gempa M5.0+ dalam periode terkini."

---

### User Story 3 — Melihat Gempa yang Dirasakan (Priority: P3)

Sebagai pengguna yang ingin tahu apakah guncangan yang ia rasakan merupakan gempa,
saya dapat melihat tab "Gempa Dirasakan" yang menampilkan gempa-gempa yang dilaporkan
dirasakan oleh masyarakat, lengkap dengan informasi wilayah yang merasakan beserta
skala intensitas MMI, sehingga saya dapat mengidentifikasi gempa yang sesuai dengan
pengalaman saya.

**Why this priority**: Fitur ini menjawab pertanyaan spesifik "apakah gempa yang saya rasakan
tercatat?". Nilainya tinggi namun lebih sempit dibanding story 1 dan 2. Bergantung pada
infrastruktur yang dibangun story 1.

**Independent Test**: Dapat diuji dengan memilih tab "Dirasakan" dan memverifikasi bahwa
setiap entri menampilkan wilayah yang merasakan dan skala MMI, tanpa perlu story lain.

**Acceptance Scenarios**:

1. **Given** pengguna memilih tab "Dirasakan", **When** data tersedia dari BMKG,
   **Then** daftar gempa yang dirasakan muncul, masing-masing dengan wilayah yang merasakan
   dan skala intensitas MMI (I–XII).
2. **Given** pengguna melihat entri gempa dirasakan, **When** ia membaca detail,
   **Then** ia dapat melihat: nama kota/provinsi yang merasakan, skala MMI yang berlaku
   di wilayah tersebut, dan keterangan deskriptif skala MMI (misal: "IV — Terasa oleh banyak
   orang, getaran seperti truk lewat").
3. **Given** pengguna melihat gempa dirasakan, **When** satu gempa dirasakan di beberapa
   wilayah dengan skala berbeda, **Then** semua wilayah dengan skala MMI masing-masing
   ditampilkan (bukan hanya satu wilayah).

---

### Edge Cases

- Apa yang terjadi jika BMKG mengembalikan response yang malformed/parseable tetapi struktur
  datanya berbeda dari yang diharapkan?
- Apa yang terjadi jika database PostgreSQL tidak tersedia bersamaan dengan BMKG tidak
  dapat diakses?
- Apa yang terjadi jika magnitudo atau kedalaman bernilai 0 atau null dari BMKG?
- Bagaimana sistem menentukan "duplikat" saat menyimpan ke PostgreSQL — apakah berdasarkan
  ID unik BMKG, atau kombinasi atribut (waktu + koordinat + magnitudo)?
- Apa yang terjadi jika BMKG mengembalikan data yang sama persis dengan yang sudah ada di
  database (upsert vs. skip)?
- Bagaimana zona waktu gempa ditampilkan — apakah selalu WIB, atau disesuaikan dengan
  zona waktu lokasi gempa (WIB/WITA/WIT)?

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Sistem HARUS menampilkan tiga kategori data gempa dari BMKG: gempa terbaru,
  gempa M5.0+, dan gempa yang dirasakan, yang dapat diakses melalui navigasi tab atau
  filter di halaman `/gempa`.
- **FR-002**: Sistem HARUS mengambil data terbaru dari API BMKG setiap kali halaman diakses
  (on-demand refresh saat page request).
- **FR-003**: Sistem HARUS menyimpan setiap gempa yang berhasil diambil dari BMKG ke
  PostgreSQL tanpa menciptakan duplikat (idempotent upsert).
- **FR-004**: Sistem HARUS menampilkan fallback dari data database jika BMKG tidak dapat
  diakses, dengan notifikasi yang jelas kepada pengguna.
- **FR-005**: Setiap entri gempa HARUS menampilkan: waktu kejadian, magnitudo (SR),
  kedalaman (km), dan lokasi (nama wilayah dari BMKG).
- **FR-006**: Untuk gempa yang dirasakan, sistem HARUS menampilkan wilayah yang merasakan
  beserta skala MMI (I–XII) dan keterangan deskriptifnya.
- **FR-007**: Untuk gempa M5.0+, sistem HARUS menampilkan label khusus jika gempa
  berpotensi tsunami sesuai informasi dari BMKG.
- **FR-008**: Sistem HARUS menampilkan pesan yang informatif dalam Bahasa Indonesia ketika:
  data tidak tersedia, BMKG tidak terjangkau, atau tidak ada gempa dalam kategori tertentu.

### Key Entities *(feature involves data)*

- **Gempa**: Satu kejadian gempa bumi yang dicatat BMKG. Atribut utama: ID unik (dari BMKG),
  waktu kejadian, magnitudo, kedalaman, koordinat (lat/lon), lokasi (nama wilayah), potensi
  tsunami, dan kategori (terbaru / M5.0+ / dirasakan). Satu gempa dapat masuk ke lebih dari
  satu kategori.
- **WilayahDirasakan**: Relasi antara satu Gempa dengan wilayah yang merasakannya.
  Atribut: nama wilayah (kota/provinsi), skala MMI. Satu gempa dapat dirasakan di banyak
  wilayah dengan skala MMI yang berbeda.
- **Sumber Data**: Tiga endpoint BMKG yang berbeda menghasilkan tiga tipe respons:
  `autogempa.json` (terbaru), `gempaterkini.json` (daftar terbaru), `gempadirasakan.json`
  (dirasakan), `gempa-m50.json` (M5.0+).

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Halaman `/gempa` memuat dan menampilkan data gempa dari BMKG dalam waktu
  kurang dari 3 detik pada koneksi normal (bukan offline).
- **SC-002**: Pengguna dapat memahami kategori gempa yang sedang dilihat (terbaru / M5.0+ /
  dirasakan) tanpa penjelasan tambahan — navigasi antar kategori selesai dalam 1 klik.
- **SC-003**: Data yang disimpan ke PostgreSQL tidak mengandung duplikat — upsert yang sama
  dijalankan dua kali menghasilkan tepat satu record.
- **SC-004**: Ketika BMKG tidak dapat diakses, pengguna melihat data fallback dari database
  dalam waktu kurang dari 1 detik (tanpa menunggu timeout yang panjang).
- **SC-005**: Informasi skala MMI pada gempa yang dirasakan dapat dibaca dan dipahami oleh
  pengguna awam — ditampilkan dalam Bahasa Indonesia dengan keterangan deskriptif.

---

## Assumptions

- Endpoint BMKG yang digunakan adalah endpoint publik tanpa autentikasi:
  `https://data.bmkg.go.id/DataMKG/TEWS/` dengan format JSON.
- ID unik gempa dari BMKG (field yang mengidentifikasi gempa secara unik) tersedia di
  respons BMKG dan konsisten antar endpoint — digunakan sebagai kunci deduplication.
- Zona waktu yang ditampilkan mengikuti zona waktu yang disediakan BMKG dalam responnya
  (biasanya WIB/UTC+7) dan tidak dikonversi ke zona waktu lokal pengguna.
- Data skala MMI (`src/data/skala-mmi.ts`) sudah tersedia dan berisi keterangan deskriptif
  untuk tiap skala I–XII — tidak perlu dibuat baru.
- Halaman `/gempa` sudah ada di routing Next.js (`src/app/gempa/page.tsx`) namun masih
  kosong — fitur ini mengisi implementasinya.
- Prisma schema belum memiliki model Gempa — perlu ditambahkan.
- Mobile support adalah in-scope untuk versi ini — halaman harus responsif.
- Halaman detail gempa individual (`/gempa/[id]`) adalah **out of scope** untuk spec ini;
  hanya halaman daftar yang menjadi target.

