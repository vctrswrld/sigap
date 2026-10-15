# Feature Specification: Fondasi Database dan Data Gempa BMKG

**Feature Branch**: `002-fondasi-database-gempa`

**Created**: 2026-10-15

**Status**: Draft

**Input**: User description: "Fondasi Database dan Data Gempa BMKG. SIGAP Gempa adalah sistem informasi kebencanaan gempa bumi untuk tugas kuliah. Data gempa berasal dari API resmi BMKG dan perlu disimpan di database agar tetap tersedia saat BMKG tidak dapat diakses, serta agar riwayat gempa dapat ditampilkan. Kebutuhan: melihat gempa terbaru, M5.0+, gempa dirasakan; melihat detail gempa (waktu WIB, koordinat, magnitudo, kedalaman, wilayah, potensi tsunami, wilayah dirasakan MMI, peta guncangan); melihat data terakhir saat BMKG tidak dapat diakses disertai waktu pembaruan terakhir. Aturan: data diperbarui saat halaman diakses (bukan scheduler); penyimpanan tanpa duplikat (waktu + koordinat); penanganan wajar kondisi kosong; konversi teks ke angka untuk magnitudo, kedalaman, dan koordinat; pengelolaan lewat seed dan database tool tanpa admin page. Cakupan: model Gempa dan Berita, sinkronisasi BMKG ke DB, halaman daftar dan detail gempa, penanganan BMKG tidak tersedia. Di luar cakupan: chatbot, konten edukasi, halaman berita, deploy."

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Menjelajahi Daftar Gempa Terkini dan Riwayat Tersimpan (Priority: P1)

Sebagai pengunjung situs SIGAP Gempa, saya ingin melihat ringkasan gempa bumi terbaru di Indonesia yang terbagi dalam tiga kategori (gempa terbaru, daftar gempa M5.0+, dan daftar gempa yang dirasakan) secara langsung dan akurat, sehingga saya dapat segera mengetahui aktivitas kegempaan yang baru saja terjadi.

**Why this priority**: Menampilkan informasi gempa terkini adalah fungsi esensial sistem. Pengunjung membuka situs terutama untuk mengetahui gempa yang sedang atau baru saja terjadi di wilayah Indonesia.

**Independent Test**: Dapat diuji secara independen dengan mengakses halaman utama daftar gempa (`/gempa`). Sistem menyinkronkan data dari BMKG ke basis data lokal dan menampilkan daftar gempa dengan atribut waktu, lokasi, magnitudo, dan kedalaman.

**Acceptance Scenarios**:

1. **Given** pengunjung mengakses halaman daftar gempa dan BMKG dapat dijangkau, **When** halaman selesai dimuat, **Then** sistem menampilkan gempa terbaru secara mencolok, serta menyediakan daftar gempa berkekuatan M5.0+ dan daftar gempa dirasakan dengan informasi waktu (WIB), magnitudo (angka), kedalaman (angka km), dan keterangan wilayah.
2. **Given** halaman daftar gempa dibuka berkali-kali oleh satu atau banyak pengunjung, **When** proses sinkronisasi dari BMKG berlangsung, **Then** sistem menyimpan data gempa ke basis data tanpa ada data ganda (catatan dengan waktu dan koordinat yang sama hanya tersimpan satu kali).
3. **Given** pengunjung beralih antar kategori (gempa terbaru, gempa M5.0+, dan gempa dirasakan), **When** kategori dipilih, **Then** daftar gempa yang relevan ditampilkan secara teratur sesuai kategori yang dipilih.

---

### User Story 2 — Memeriksa Informasi Rinci Satu Kejadian Gempa (Priority: P2)

Sebagai pengunjung, saya ingin membuka halaman detail dari suatu kejadian gempa untuk melihat informasi lengkap meliputi waktu kejadian (WIB), koordinat presisi, magnitudo numerik, kedalaman numerik, nama wilayah, status potensi tsunami, daftar wilayah yang merasakan beserta skala MMI, dan gambar peta guncangan (bila tersedia), sehingga saya memperoleh gambaran komprehensif mengenai dampak gempa tersebut.

**Why this priority**: Setelah melihat rangkuman di daftar gempa, pengunjung membutuhkan informasi mendalam untuk menilai tingkat bahaya dan dampak di daerah terdampak.

**Independent Test**: Dapat diuji dengan memilih salah satu gempa dari daftar dan membuka halaman detailnya, lalu memverifikasi kelengkapan seluruh atribut gempa yang ditampilkan.

**Acceptance Scenarios**:

1. **Given** pengunjung memilih salah satu gempa M5.0+ atau gempa terbaru dari daftar, **When** halaman detail terbuka, **Then** sistem menampilkan waktu kejadian dalam format WIB, nilai magnitudo numerik, kedalaman dalam km, koordinat lintang dan bujur, keterangan wilayah, status potensi tsunami, serta gambar peta guncangan (shakemap).
2. **Given** pengunjung memilih gempa dari kategori gempa dirasakan, **When** halaman detail terbuka, **Then** sistem menampilkan rincian wilayah-wilayah yang merasakan beserta skala intensitas MMI masing-masing, serta menangani ketiadaan potensi tsunami dan peta guncangan secara wajar tanpa menampilkan broken image atau teks galat kosong [NEEDS CLARIFICATION: Perlakuan visual peta guncangan pada gempa dirasakan].
3. **Given** data magnitudo, kedalaman, dan koordinat berasal dari sumber luar dalam format teks, **When** disimpan dan ditampilkan pada detail, **Then** nilai-nilai tersebut telah terkonversi menjadi format angka yang valid dan terstandarisasi.

---

### User Story 3 — Mengakses Data Gempa Saat Layanan BMKG Tidak Tersedia (Priority: P3)

Sebagai pengunjung, saya ingin tetap dapat melihat data gempa terakhir yang telah tersimpan di basis data ketika layanan BMKG sedang mengalami gangguan atau tidak dapat diakses, disertai keterangan waktu pembaruan terakhir, agar saya tidak mendapati halaman rusak atau kosong total.

**Why this priority**: Menjaga keandalan layanan (availability) sistem informasi kebencanaan sangat penting, terutama saat sumber utama mengalami lonjakan beban atau down saat bencana terjadi.

**Independent Test**: Dapat diuji dengan memutus sambungan ke sumber data eksternal (BMKG), kemudian memuat halaman gempa; sistem harus menyajikan data terakhir dari basis data lokal dengan penanda status pembaruan.

**Acceptance Scenarios**:

1. **Given** sumber data BMKG sedang tidak dapat dijangkau dan basis data sudah memiliki riwayat gempa, **When** pengunjung membuka halaman daftar atau detail gempa, **Then** sistem menyajikan data gempa terakhir yang tersimpan dan menampilkan pesan informatif bahwa data yang disajikan adalah data tersimpan terakhir beserta catatan waktu pembaruan terakhir.
2. **Given** sumber data BMKG pulih kembali, **When** pengunjung memuat ulang halaman, **Then** sistem secara otomatis memperbarui data ke versi terbaru dari BMKG dan memperbarui catatan waktu pembaruan terakhir.
3. **Given** sumber data BMKG tidak dapat dijangkau dan basis data dalam keadaan kosong (kondisi awal tanpa data), **When** pengunjung membuka halaman, **Then** sistem menampilkan pesan penjelas bahwa data belum tersedia dan menyarankan untuk mencoba beberapa saat lagi, tanpa menyebabkan galat sistem (*unhandled exception*).

---

### Edge Cases

- **Koneksi Sumber Data Luar Putus Saat Basis Data Kosong**: Sistem menampilkan status ramah pengguna (*empty state*) dalam Bahasa Indonesia dan tidak menampilkan layar galat teknis.
- **Data Teks Sumber Tidak Standar**: Apabila nilai magnitudo, kedalaman, atau koordinat mengandung karakter non-numerik tak terduga (misalnya "5.2 SR" atau "-6.78 LS"), sistem membersihkan dan mengonversi format ke nilai angka yang tepat sebelum disimpan.
- **Ketiadaan Atribut Tertentu pada Gempa Dirasakan**: Gempa dirasakan tidak menyediakan informasi potensi tsunami maupun gambar peta guncangan; antarmuka tidak menampilkan label "null", teks kosong tanpa konteks, atau gambar rusak.
- **Akses Halaman Simultan**: Ketika beberapa pengguna mengakses halaman pada detik yang sama di saat data baru dirilis, mekanisme penyimpanan menjamin tidak ada duplikasi data (*idempotent*).
- **Gempa Susulan di Lokasi yang Berdekatan**: Dua gempa yang terjadi pada koordinat berdekatan tetapi pada waktu yang berbeda dianggap sebagai dua kejadian gempa yang berbeda. Identifikasi keunikan mengombinasikan waktu dan koordinat kejadian secara presisi.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Sistem HARUS menyediakan model data `Gempa` pada basis data untuk menyimpan catatan kejadian gempa bumi yang mencakup waktu kejadian (WIB), koordinat lintang dan bujur (numerik), magnitudo (numerik), kedalaman (numerik km), nama wilayah, potensi tsunami, data wilayah dirasakan (skala MMI), tautan peta guncangan, dan waktu pencatatan.
- **FR-002**: Sistem HARUS menyediakan model data `Berita` pada basis data sebagai fondasi persiapan fitur berikutnya, yang mencakup identitas berita, judul, ringkasan, isi, tautan sumber/gambar, dan tanggal publikasi [NEEDS CLARIFICATION: Kebutuhan data awal seed untuk Berita].
- **FR-003**: Sistem HARUS menyinkronkan data gempa dari API BMKG ke dalam basis data setiap kali halaman gempa diakses oleh pengunjung (on-demand synchronization).
- **FR-004**: Sistem HARUS menerapkan aturan pencegahan duplikasi data, di mana gempa dengan kombinasi waktu kejadian dan koordinat yang sama hanya boleh tersimpan tepat satu kali di basis data.
- **FR-005**: Sistem HARUS mengonversi data teks magnitudo, kedalaman, dan koordinat dari respons sumber BMKG menjadi tipe data angka (numerik) sebelum disimpan ke basis data.
- **FR-006**: Sistem HARUS menampilkan daftar gempa terbaru, daftar gempa M5.0+ (terkini), dan daftar gempa dirasakan pada antarmuka halaman daftar gempa.
- **FR-007**: Sistem HARUS menyediakan halaman detail untuk setiap kejadian gempa yang menampilkan seluruh informasi rinci: waktu (WIB), koordinat, magnitudo, kedalaman, wilayah, potensi tsunami, wilayah dirasakan (beserta skala MMI), dan peta guncangan.
- **FR-008**: Sistem HARUS menangani kondisi ketiadaan potensi tsunami dan peta guncangan pada gempa dirasakan secara wajar tanpa merusak tata letak antarmuka.
- **FR-009**: Sistem HARUS menyajikan data terakhir yang ada di basis data disertai penanda waktu pembaruan terakhir apabila sumber data BMKG sedang tidak dapat dijangkau.
- **FR-010**: Sistem HARUS mendukung pengisian data awal (*seeding*) dan pengelolaan data menggunakan perkakas basis data pengembang tanpa memerlukan antarmuka halaman admin pengguna.
- **FR-011**: Seluruh teks antarmuka, pesan status, indikator waktu, dan pemberitahuan galat HARUS disajikan dalam Bahasa Indonesia yang baku dan komunikatif.

---

### Key Entities *(feature involves data)*

- **Gempa**:
  - Merepresentasikan satu kejadian gempa bumi yang dicatat dari BMKG.
  - Atribut utama: Waktu gempa (WIB), magnitudo (angka desimal), kedalaman (angka bilangan bulat/desimal km), koordinat lintang & bujur (angka desimal presisi), nama wilayah, status potensi tsunami, informasi wilayah yang merasakan (skala MMI), tautan/nama berkas peta guncangan (*shakemap*), kategori gempa (terbaru / M5.0+ / dirasakan), dan waktu sinkronisasi terakhir.
  - Batasan keunikan: Kombinasi waktu gempa dan koordinat adalah unik (tidak boleh ada catatan ganda).
- **WilayahDirasakan**:
  - Merepresentasikan hubungan wilayah terdampak dan tingkat getaran (skala MMI) untuk gempa yang dirasakan masyarakat.
  - Atribut utama: Relasi ke kejadian Gempa, nama wilayah/kota, dan tingkatan skala MMI (I sampai XII).
- **Berita**:
  - Merepresentasikan artikel/informasi kebencanaan yang disiapkan sebagai fondasi entitas data untuk fitur mendatang.
  - Atribut utama: Judul berita, ringkasan singkat, konten/isi artikel, tautan gambar/sumber, tanggal dibuat, dan status publikasi.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Membuka dan menyegarkan (*refresh*) halaman gempa berkali-kali tidak menambah jumlah baris data duplikat di basis data (0% baris duplikat untuk waktu dan koordinat yang sama).
- **SC-002**: Saat sumber BMKG tidak dapat dihubungi atau diputus secara simulasi, halaman gempa tetap berhasil menampilkan data tersimpan dari basis data dalam waktu kurang dari 2 detik dengan indikator waktu pembaruan terakhir yang jelas.
- **SC-003**: Gempa terbaru yang dirilis oleh BMKG dapat dilihat pada antarmuka pengguna dalam waktu maksimal 1 menit setelah dirilis begitu halaman diakses oleh pengunjung.
- **SC-004**: Pengunjung dapat berpindah dari daftar gempa ke tampilan detail gempa dan melihat seluruh parameter teknis (magnitudo numerik, kedalaman, skala MMI, peta guncangan jika ada) dalam waktu kurang dari 1 detik.
- **SC-005**: 100% data teks magnitudo, kedalaman, dan koordinat dari BMKG berhasil terkonversi menjadi format numerik yang valid pada basis data tanpa kegagalan konversi tipe data.

---

## Assumptions

- **Mekanisme Pembaruan Data**: Sinkronisasi dilakukan secara on-demand saat pengunjung memuat halaman (server-side fetch saat request), bukan melalui daemon scheduler atau cron job background.
- **Manajemen Data**: Tidak disediakan antarmuka administratif (*admin dashboard*). Pengelolaan dan inspeksi data dilakukan melalui skrip data awal (*seed*) dan antarmuka manajemen basis data pengembang (*Studio*).
- **Zona Waktu**: Seluruh waktu kejadian gempa disajikan dan distandarisasi dalam Waktu Indonesia Barat (WIB) sesuai rujukan data BMKG.
- **Status Tsunami dan Peta Guncangan**: Gempa kategori "dirasakan" dari BMKG memang secara alami tidak memiliki data potensi tsunami dan peta guncangan; sistem menganggap hal ini sebagai perilaku normal (*expected empty value*).
- **Entitas Berita**: Model Berita dibuat pada skema basis data sebagai fondasi persiapan, namun halaman antarmuka pengguna untuk berita tidak dikembangkan pada cakupan fitur ini.

