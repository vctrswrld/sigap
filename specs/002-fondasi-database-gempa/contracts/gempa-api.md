# Interface & Service Contracts: Fondasi Data Gempa

**Feature**: `002-fondasi-database-gempa`  
**Date**: 2026-10-15

---

## 1. Modul Library BMKG (`src/lib/bmkg.ts`)

Modul ini adalah **satu-satunya gerbang akses** untuk fetching data BMKG dan sinkronisasi ke PostgreSQL (mematuhi Konstitusi Prinsip I).

### Tipe Data Input dari BMKG (Raw)
```typescript
export interface BMKGRawGempa {
  Tanggal: string;
  Jam: string;
  DateTime: string;      // Contoh mentah: "2026-10-15T07:30:00+00:00"
  Coordinates: string;   // Contoh mentah: "-6.78,105.12"
  Lintang: string;
  Bujur: string;
  Magnitude: string;
  Kedalaman: string;
  Wilayah: string;
  Potensi?: string;      // Hanya ada di autogempa & gempaterkini
  Dirasakan?: string;    // Ada di gempadirasakan & kadang autogempa
  Shakemap?: string;     // Hanya ada di autogempa & gempaterkini
}

export interface BMKGAutogempaResponse {
  Infogempa: {
    gempa: BMKGRawGempa;
  };
}

export interface BMKGListResponse {
  Infogempa: {
    gempa: BMKGRawGempa[];
  };
}
```

### Fungsi Utama
```typescript
/**
 * Fungsi sinkronisasi ber-cache (unstable_cache dengan revalidate: 60).
 * Menjalankan fetch ke 3 endpoint BMKG dan melakukan upsert ke PostgreSQL maksimal 1x per 60 detik.
 */
export const syncBMKGWithCache: () => Promise<{
  insertedOrUpdated: number;
  errors: string[];
}>;

/**
 * Mengambil data gempa dari database dengan informasi status fallback jika BMKG gagal.
 */
export async function getGempaFeed(kategori: 'semua' | 'terbaru' | 'terkini_m5' | 'dirasakan'): Promise<{
  data: GempaModel[];
  isFallback: boolean;
  lastSync: Date | null;
}>;
```

---

## 2. API Route Handler (`src/app/api/gempa/route.ts`)

Endpoint REST internal untuk konsumsi antarmuka atau pengujian API.

### `GET /api/gempa`

#### Query Parameters:
| Parameter | Tipe | Wajib | Nilai yang Diizinkan | Keterangan |
|-----------|------|-------|----------------------|------------|
| `kategori`| string | Tidak | `terbaru`, `terkini_m5`, `dirasakan`, `semua` | Default: `semua` |

#### Respon Sukses (BMKG Normal):
```json
{
  "success": true,
  "data": [
    {
      "id": "cm2a1b2c3d4e5f6g7h8i9j0k",
      "bmkgKey": "2026-10-15T07:30:00+00:00_-6.78,105.12",
      "waktuUtc": "2026-10-15T07:30:00.000Z",
      "tanggal": "15 Okt 2026",
      "jam": "14:30:00 WIB",
      "lat": -6.78,
      "lon": 105.12,
      "magnitudo": 5.4,
      "kedalamanKm": 10.0,
      "wilayah": "14 km BaratDaya KAB-SUMEDANG-JABAR",
      "potensi": "Tidak berpotensi tsunami",
      "statusTsunami": "AMAN",
      "dirasakan": "III-IV Sumedang, II-III Bandung",
      "shakemap": "20261015143000.mmi.jpg",
      "jenis": "TERBARU",
      "createdAt": "2026-10-15T07:30:05.000Z",
      "updatedAt": "2026-10-15T07:30:05.123Z"
    }
  ],
  "meta": {
    "kategori": "terbaru",
    "total": 1,
    "isFallback": false,
    "lastSync": "2026-10-15T07:30:05.123Z"
  }
}
```

#### Respon Sukses Saat BMKG Tidak Tersedia (Fallback Database):
```json
{
  "success": true,
  "data": [ ... ],
  "meta": {
    "kategori": "terbaru",
    "total": 15,
    "isFallback": true,
    "lastSync": "2026-10-15T06:00:00.000Z",
    "message": "Menampilkan data tersimpan terakhir — BMKG sedang tidak dapat dijangkau."
  }
}
```

---

## 3. Server Component Configuration & Props Contract

Sesuai standar Next.js 16 App Router:

### Konfigurasi Halaman Dinamis
Pada `src/app/gempa/page.tsx` dan `src/app/gempa/[id]/page.tsx`:
```typescript
export const dynamic = 'force-dynamic';
```

### Props Halaman Detail (`src/app/gempa/[id]/page.tsx`)
```typescript
export interface GempaDetailPageProps {
  params: Promise<{
    id: string;
  }>;
}
```
