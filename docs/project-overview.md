# Project Overview: SIGAP Gempa

## Ringkasan

**SIGAP Gempa** adalah aplikasi web informasi dan edukasi kegempaan untuk masyarakat Indonesia. Aplikasi menampilkan data gempa terkini dari BMKG, menjelaskan proses gempa, mitigasi, dan penanggulangan, memuat berita gempa dari sumber tepercaya, serta menyediakan chatbot untuk tanya jawab seputar gempa.

Project ini adalah tugas kuliah Sistem Informasi, dikerjakan sendiri dengan Next.js (React), integrasi API BMKG dan Gemini, serta PostgreSQL + Prisma di VPS.

**Masalah yang diselesaikan:** informasi gempa tersebar di banyak kanal, sulit dipahami orang awam, dan jarang disertai panduan tindakan. SIGAP memadukan data resmi, penjelasan sederhana, dan langkah praktis dalam satu tempat.

**Pengguna sasaran:** warga umum, pelajar atau mahasiswa, dan penilai tugas yang perlu melihat alur sistem informasi secara utuh.

## Ruang lingkup

| Termasuk (dibangun) | Tidak termasuk |
| --- | --- |
| Gempa terkini dan daftar gempa M5+ dari BMKG, riwayat gempa dari database | Peringatan dini atau notifikasi push |
| Peta sebaran gempa | Prediksi gempa |
| Detail gempa (kedalaman, dirasakan, potensi tsunami, shakemap) | Login dan akun pengguna (belum diputuskan perlu) |
| Konten edukasi: pengertian, proses, penanggulangan, mitigasi | Aplikasi mobile native |
| Berita gempa (judul, sumber, tanggal, tautan) | Menyalin isi penuh artikel berita |
| Chatbot bertopik gempa dengan fallback FAQ | Chatbot untuk topik di luar gempa |
| Daftar sumber dan pustaka | Integrasi MySQL (tidak dipakai) |

## Persona dan user story

**Persona:** P1 Warga umum (butuh info cepat dan langkah tindakan), P2 Pelajar atau mahasiswa (belajar dan butuh sumber), P3 Pengelola konten (opsional, mengelola isi situs).

Prioritas memakai MoSCoW: M = wajib, S = sebaiknya, C = bila sempat.

| ID | Persona | User story | Prioritas |
| --- | --- | --- | --- |
| US-01 | P1 | Sebagai warga, saya ingin melihat gempa terkini (magnitudo, lokasi, waktu) supaya tahu ada tidaknya gempa baru. | M |
| US-02 | P1 | Sebagai warga, saya ingin melihat daftar gempa M5+ terbaru supaya tahu gempa signifikan belakangan ini. | M |
| US-03 | P1 | Sebagai warga, saya ingin melihat peta sebaran gempa supaya paham lokasinya relatif ke daerah saya. | M |
| US-04 | P1 | Sebagai warga, saya ingin membuka detail gempa (kedalaman, dirasakan, potensi tsunami) supaya tahu dampaknya. | M |
| US-05 | P2 | Sebagai pelajar, saya ingin memfilter riwayat gempa menurut magnitudo, tanggal, dan wilayah supaya bisa menganalisis data. | S |
| US-06 | P1 | Sebagai warga, saya ingin membaca berita gempa dari sumber tepercaya supaya mendapat konteks. | M |
| US-07 | P2 | Sebagai pelajar, saya ingin memahami pengertian dan proses terjadinya gempa supaya paham penyebabnya. | M |
| US-08 | P1 | Sebagai warga, saya ingin tahu langkah sebelum, saat, dan sesudah gempa supaya siap bertindak. | M |
| US-09 | P1 | Sebagai warga, saya ingin daftar isi tas siaga dan kontak darurat supaya bisa bersiap. | S |
| US-10 | P1 | Sebagai warga, saya ingin bertanya ke chatbot soal gempa supaya dapat jawaban cepat. | M |
| US-11 | P1 | Sebagai warga, saya ingin chatbot tetap menjawab saat layanan AI tidak tersedia supaya tidak buntu. | M |
| US-12 | P1 | Sebagai pengguna HP, saya ingin tampilan yang nyaman di layar kecil supaya mudah dibaca. | M |
| US-13 | P2 | Sebagai pelajar, saya ingin melihat sumber setiap materi supaya bisa mengutipnya. | M |
| US-14 | P3 | Sebagai pengelola, saya ingin mengubah konten tanpa deploy ulang supaya mudah dirawat. | C |

## Kebutuhan fungsional

| ID | Modul | Kebutuhan | User story | Prioritas |
| --- | --- | --- | --- | --- |
| FR-01 | Data gempa | Menampilkan gempa terkini dari BMKG (`autogempa`) dengan cache 60 detik. | US-01 | M |
| FR-02 | Data gempa | Menampilkan daftar gempa M5+ (`gempaterkini`) dengan cache 300 detik. | US-02 | M |
| FR-03 | Data gempa | Menormalisasi data BMKG: string ke angka, pecah koordinat, buang satuan, petakan potensi tsunami. | US-01, US-02 | M |
| FR-04 | Data gempa | Menyimpan gempa ke database dengan upsert (kunci: waktu + koordinat), tanpa duplikat. | US-05 | M |
| FR-05 | Data gempa | Saat BMKG tidak dapat diakses, menampilkan data terakhir dari database beserta waktu pembaruan. | US-01 | M |
| FR-06 | Peta | Menampilkan marker gempa di peta (Leaflet + OpenStreetMap) dengan atribusi. | US-03 | M |
| FR-07 | Detail | Halaman detail gempa: waktu WIB, koordinat, kedalaman, wilayah, dirasakan, potensi tsunami, shakemap. | US-04 | M |
| FR-08 | Riwayat | Riwayat gempa dari database dengan filter magnitudo, rentang tanggal, wilayah, dan paginasi. | US-05 | S |
| FR-09 | Edukasi | Halaman pengertian dan proses gempa (diagram hiposentrum dan episentrum, gelombang P dan S, skala magnitudo). | US-07 | M |
| FR-10 | Edukasi | Halaman penanggulangan dan mitigasi per fase (sebelum, saat, sesudah) dengan checklist. | US-08 | M |
| FR-11 | Edukasi | Daftar tas siaga dan kontak darurat (112, 115, 117, BPBD setempat). | US-09 | S |
| FR-12 | Sumber | Setiap materi menampilkan sumbernya; halaman Tentang memuat daftar pustaka. | US-13 | M |
| FR-13 | Berita | Daftar berita (judul, sumber, tanggal, tautan keluar) dari sumber yang dipilih. | US-06 | M |
| FR-14 | Chatbot | Antarmuka chat dengan riwayat sesi, saran pertanyaan, batas 500 karakter per pesan. | US-10 | M |
| FR-15 | Chatbot | Endpoint `POST /api/chat`: memanggil Gemini dengan system prompt bertopik gempa, kunci API hanya di server. | US-10 | M |
| FR-16 | Chatbot | Fallback bertingkat: model utama, model cadangan, lalu FAQ rule-based. | US-11 | M |
| FR-17 | Chatbot | Pembatasan laju per IP (10 per menit) dan penghitung kuota harian per model. | US-11 | S |
| FR-18 | Chatbot | Pencatatan percakapan (tanpa IP mentah) untuk evaluasi. | US-10 | C |
| FR-19 | Konten | Konten edukasi, berita, FAQ, dan sumber disimpan di database dan diisi lewat seed. | US-14 | S |
| FR-20 | UI | Tampilan responsif dari lebar 360 px dan format angka Indonesia (4,6) dengan zona WIB. | US-12 | M |

## Kebutuhan non-fungsional

| ID | Aspek | Kebutuhan |
| --- | --- | --- |
| NFR-01 | Performa | Halaman utama tampil kurang dari 3 detik pada koneksi umum; data BMKG di-cache. |
| NFR-02 | Aksesibilitas | Kontras teks minimal 4,5:1, navigasi dengan keyboard, teks alternatif untuk peta dan diagram, tidak mengandalkan warna saja untuk menyampaikan magnitudo. |
| NFR-03 | Responsif | Nyaman dari lebar 360 px sampai desktop. |
| NFR-04 | Keamanan | Tidak ada kunci API atau kredensial database di repositori atau bundel klien; rate limit di endpoint chat. |
| NFR-05 | Keandalan | Aplikasi tetap menampilkan halaman saat BMKG, Gemini, database, atau Redis tidak tersedia (degradasi anggun). |
| NFR-06 | Privasi | IP pengguna tidak disimpan mentah; log percakapan memiliki batas retensi. |
| NFR-07 | Akurasi konten | Materi dan nomor darurat diverifikasi ke sumber resmi (BMKG, BNPB, PVMBG) sebelum dipublikasikan. |
| NFR-08 | Atribusi | Mencantumkan sumber data BMKG dan atribusi OpenStreetMap contributors. |

## Arsitektur dan teknologi

| Komponen | Peran | Lokasi |
| --- | --- | --- |
| Next.js (App Router, TypeScript) | Antarmuka, Server Components, Route Handlers | Laptop (dev); tempat deploy belum ditentukan |
| Tailwind CSS, Recharts, react-leaflet | Gaya, grafik, peta | Klien |
| API BMKG (`data.bmkg.go.id`, tanpa kunci) | Sumber data gempa terkini, M5+, dirasakan | Eksternal |
| Gemini API (Google AI Studio, tingkat gratis) | Jawaban chatbot; kuota 20 permintaan per hari per model | Eksternal |
| OpenStreetMap | Ubin peta | Eksternal |
| PostgreSQL 16 + Prisma ORM | Riwayat gempa, konten, FAQ, log chat | VPS Sumopod (Docker), hanya `127.0.0.1` |
| Redis 7 (opsional) | Rate limit, penghitung kuota, cache | VPS Sumopod (Docker), hanya `127.0.0.1` |
| Akses pengembangan | SSH tunnel dan DataGrip | Laptop ke VPS lewat kunci SSH |

Alur data: halaman dan Route Handler memanggil `lib/bmkg.ts` untuk mengambil dan menormalisasi data, menyimpannya ke PostgreSQL lewat Prisma, lalu menampilkannya. `POST /api/chat` memanggil Gemini dari server, dan jatuh ke FAQ bila kuota habis.

## Halaman dan sumber data

| Halaman | Rute (usulan) | Isi utama | Sumber data | Desain |
| --- | --- | --- | --- | --- |
| Beranda | `/` | Gempa terkini, statistik, daftar M5+, peta, pintu ke edukasi | BMKG + DB | Sudah |
| Data gempa | `/gempa` | Tab terkini, M5+, dirasakan, riwayat; filter; peta | BMKG + DB | Sudah |
| Detail gempa | `/gempa/[id]` | Detail, shakemap, peta zoom | BMKG + DB | Sudah |
| Proses gempa | `/edukasi/proses` | Pengertian, proses, gelombang, skala magnitudo | DB (seed) | Sudah |
| Mitigasi | `/mitigasi` | Sebelum, saat, sesudah; tas siaga; kontak darurat | DB (seed) | Sudah |
| Penanggulangan | `/penanggulangan` | Tanggap darurat dan pemulihan | DB (seed) | Belum |
| Berita | `/berita` | Daftar berita gempa | DB atau RSS | Belum |
| Chatbot | `/chat` dan tombol mengambang | Tanya jawab gempa | Gemini + FAQ | Sudah |
| Tentang | `/tentang` | Tujuan, sumber data, daftar pustaka | DB (seed) | Belum |

Model data (tabel di PostgreSQL): `Gempa`, `Berita`, `Artikel`, `Sumber`, `FaqGempa`, `ChatLog`. Rincian kolom ada di `PRD-DATABASE.md`.

## Status, risiko, dan pertanyaan terbuka

**Status per 11 Okt 2026:** PRD, PRD database, DESIGN.md, dan rancangan antarmuka (7 papan) sudah ada. Infrastruktur VPS selesai: SSH hanya dengan kunci, firewall hanya 22/80/443, PostgreSQL, MySQL, dan Redis dibuat ulang dengan password baru dan port terikat lokal, DataGrip tersambung. Berikutnya: user dan database `sigap`, Prisma, lalu integrasi data BMKG.

| Risiko | Dampak | Mitigasi |
| --- | --- | --- |
| Kuota Gemini 20 permintaan per hari per model | Chatbot sering jatuh ke FAQ | Fallback bertingkat, penghitung kuota, FAQ yang bagus |
| Format data BMKG berubah | Sinkronisasi gagal | Normalisasi defensif, simpan teks asli, catat error |
| VPS atau jaringan bermasalah saat demo | Aplikasi tak dapat diakses | Data demo atau rekaman layar sebagai cadangan, uji sehari sebelum |
| Konten edukasi atau nomor darurat keliru | Informasi menyesatkan | Verifikasi ke BMKG, BNPB, PVMBG sebelum publikasi |
| Kebijakan ubin OpenStreetMap | Peta diblokir bila melanggar | Baca kebijakan penggunaan ubin, cantumkan atribusi |
| Layanan database terpapar internet | Pembobolan (sudah pernah terjadi) | Port hanya `127.0.0.1`, firewall, password acak, jangan buka port database |

## Keputusan:
- Pembaruan data BMKG: saat halaman diakses (fetch dengan revalidate, lalu upsert ke database); tambah cron di VPS bila riwayat sering bolong.
- Tanpa halaman admin: konten diisi lewat seed dan diedit lewat Prisma Studio atau DataGrip.
- Next.js dideploy di VPS (Docker, satu jaringan dengan Postgres); domain dibuat kemudian.
- Berita dikurasi manual (8-10 item: judul, sumber, tanggal, tautan); RSS ditunda.
- Tidak ada ketentuan khusus dari dosen; yang penting ada penjelasan gempa: pengertian, jenis, proses, penanggulangan, dan mitigasi.
- Struktur gempadirasakan.json sudah diperiksa: array 15 gempa di Infogempa.gempa, tanpa Potensi dan Shakemap; teks Dirasakan memuat skala MMI per wilayah.