# Sigap Gempa

Kerangka aplikasi Next.js untuk informasi gempa, edukasi, mitigasi, dan penanggulangan bencana.

## Menjalankan

```bash
npm install
npm run dev
```

Salin `.env.example` menjadi `.env.local` sebelum menghubungkan sumber data BMKG atau layanan LLM.

## Struktur

- `src/app`: halaman, route handler API, dan layout global.
- `src/components`: komponen UI berdasarkan domain.
- `src/data`: konten edukasi dan mitigasi statis.
- `src/lib`: integrasi BMKG, utilitas, dan chatbot.
- `src/types`: tipe data aplikasi.

Lihat [visual struktur project](./docs/PROJECT-STRUCTURE.md) untuk diagram arsitektur,
tree file lengkap, peta domain, dan alur data utama.
