# SIGAP Constitution

## Core Principles

### I. Data-First: BMKG sebagai Sumber Kebenaran Tunggal
Semua data gempa **wajib** berasal dari API BMKG melalui `src/lib/bmkg.ts`.
- Tidak ada fetching BMKG langsung dari komponen client — selalu lewat `lib/bmkg.ts` atau API route
- API route di `src/app/api/gempa/` bertindak sebagai proxy/cache layer antara client dan BMKG
- Tipe data gempa didefinisikan sekali di `src/types/gempa.ts` dan digunakan di seluruh codebase
- Caching & revalidation wajib dikonfigurasi pada setiap `fetch()` ke BMKG (Next.js `revalidate`)

### II. Server-First Rendering (NON-NEGOTIABLE)
Gunakan React Server Components (RSC) sebagai default; Client Components hanya bila mutlak perlu.
- Halaman di `src/app/` adalah Server Components secara default
- `'use client'` hanya diizinkan untuk: komponen yang membutuhkan `useState`/`useEffect`/event handlers/browser APIs
- Chatbot widget (`src/components/chatbot/`) adalah pengecualian — seluruhnya client-side
- Data fetching awal dilakukan di Server Component, bukan di `useEffect`
- Jangan tambahkan `'use client'` tanpa justifikasi yang jelas

### III. TypeScript Strict — Tidak Ada `any`
Seluruh kode TypeScript harus mengikuti strict mode.
- Dilarang menggunakan tipe `any` — gunakan tipe eksplisit atau `unknown` + type guard
- Semua props komponen harus punya interface bertipe dengan suffix `Props`
- Fungsi harus punya explicit return type
- Tipe data domain (gempa, chat) didefinisikan di `src/types/` — tidak boleh diduplikasi inline
- Gunakan Prisma-generated types untuk semua operasi database

### IV. UI Berbahasa Indonesia (NON-NEGOTIABLE)
Seluruh antarmuka pengguna menggunakan Bahasa Indonesia.
- Label, pesan error, placeholder, tooltip, loading state — semua dalam Bahasa Indonesia
- Variabel, fungsi, dan komentar kode boleh bahasa Inggris (konvensi teknis)
- Nama file dan folder menggunakan bahasa Inggris atau Inggris-Indonesia (misal: `GempaCard`, `bmkg.ts`)
- Satuan data gempa mengikuti standar BMKG (SR untuk magnitudo, km untuk kedalaman, WIB/WITA/WIT)

### V. Komponen UI Reusable — Jangan Duplikasi
Sebelum membuat komponen baru, cek `src/components/ui/` terlebih dahulu.
- `Button`, `Card`, `Badge`, `Skeleton`, `Tabs` sudah tersedia dan harus digunakan sebagai base
- Komponen domain-spesifik dikelompokkan per fitur: `gempa/`, `chatbot/`, `mitigasi/`, `edukasi/`
- Styling **hanya** menggunakan Tailwind CSS utility classes — dilarang inline style
- Mobile-first responsive design: semua komponen harus berfungsi di mobile

### VI. Database — Prisma 7 + PostgreSQL
Semua operasi database melalui Prisma Client yang di-generate.
- Schema didefinisikan di `prisma/schema.prisma` — satu-satunya sumber kebenaran skema DB
- Jalankan `prisma generate` setelah setiap perubahan schema
- Jalankan `prisma migrate dev` untuk membuat migration baru di development
- Tidak ada raw SQL kecuali untuk query yang tidak bisa diekspresikan via Prisma Client API
- Prisma Client diinstansiasi sekali (singleton pattern) — hindari multiple instances

### VII. Chatbot Gemini — Konteks Terbatas Gempa
Chatbot hanya menjawab pertanyaan seputar gempa bumi dan keselamatan.
- System prompt di `src/lib/chat/prompt.ts` mendefinisikan batas domain chatbot
- Fallback responses di `src/lib/chat/fallback.ts` untuk pertanyaan di luar domain
- LLM API call hanya dari server-side (`src/app/api/chat/`) — API key tidak boleh exposed ke client
- State chat dikelola di `src/hooks/useChat.ts` — tidak boleh ada state chat di luar hook ini

---

## Arsitektur & Batasan Teknis

### Stack Resmi
| Layer | Teknologi | Versi |
|-------|-----------|-------|
| Framework | Next.js App Router | 15.x |
| Bahasa | TypeScript | strict mode |
| Styling | Tailwind CSS | 3.x |
| Database ORM | Prisma | 7.x |
| Database | PostgreSQL | — |
| AI/LLM | Google Gemini API | — |
| Data Eksternal | BMKG API | — |

### Batasan Arsitektur
- **Tidak ada** library state management global (Redux, Zustand, dll) — gunakan React state + server actions
- **Tidak ada** CSS-in-JS — hanya Tailwind
- **Tidak ada** REST client library (axios, ky) di server — gunakan native `fetch()` dengan Next.js extensions
- API routes **wajib** return `NextResponse.json()` dengan status code yang tepat
- Environment variables sensitif (API keys, `DATABASE_URL`) hanya boleh diakses di server-side

### Struktur Folder Kanonikal
```
sigap/
├── src/
│   ├── app/              # Next.js App Router (halaman + API routes)
│   │   └── api/          # Route Handlers (server-only)
│   ├── components/       # React components
│   │   └── ui/           # Base UI components (reuse first!)
│   ├── hooks/            # Custom React hooks (client-side)
│   ├── lib/              # Server utilities & external integrations
│   │   ├── bmkg.ts       # BMKG API — satu-satunya titik akses
│   │   └── chat/         # Gemini LLM integration
│   ├── types/            # TypeScript type definitions
│   └── data/             # Static data (mitigasi, FAQ, dll)
├── prisma/
│   └── schema.prisma     # Database schema (single source of truth)
└── .specify/             # Spec Kit artifacts
    └── memory/
        └── constitution.md  # File ini
```

---

## Workflow Pengembangan

### Sebelum Mulai Fitur Baru
1. Baca constitution ini untuk memastikan rencana tidak melanggar prinsip
2. Cek tipe di `src/types/` — apakah perlu tipe baru?
3. Cek `src/lib/bmkg.ts` — apakah data yang dibutuhkan sudah tersedia?
4. Cek `src/components/ui/` — apakah komponen UI bisa digunakan?

### Saat Membuat Fitur
1. **Tipe dulu** → definisikan di `src/types/`
2. **Data layer** → update `bmkg.ts` atau tambah Prisma model jika perlu
3. **API route** → buat/update di `src/app/api/`
4. **Komponen** → mulai dari Server Component, tambah `'use client'` hanya jika perlu
5. **Halaman** → sambungkan di `src/app/`

### Constitution Check (wajib sebelum PR)
- [ ] Tidak ada fetching BMKG langsung dari client
- [ ] Tidak ada `any` type
- [ ] `'use client'` hanya digunakan dengan justifikasi
- [ ] Semua label UI dalam Bahasa Indonesia
- [ ] Tidak ada duplikasi komponen yang sudah ada di `ui/`
- [ ] Tidak ada raw SQL (kecuali didokumentasikan)
- [ ] API key tidak exposed ke client
- [ ] Prisma Client tidak diinstansiasi berulang

---

## Governance

Constitution ini berlaku di atas semua konvensi lain. Pengecualian harus:
1. Didokumentasikan dengan justifikasi di file terkait (komentar kode)
2. Disetujui sebelum implementasi
3. Dicatat di bagian "Complexity Tracking" pada `plan.md` fitur terkait

Amendments terhadap constitution memerlukan:
- Justifikasi tertulis mengapa prinsip lama tidak lagi memadai
- Migration plan untuk kode yang sudah ada
- Update pada `AGENTS.md` agar konsisten

**Version**: 1.0.0 | **Ratified**: 2026-10-13 | **Last Amended**: 2026-10-13
