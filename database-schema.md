# Database Schema - Kerja Praktik

Dokumentasi ini menjelaskan struktur database untuk sistem Kerja Praktik STI. Schema ini fokus pada tabel-tabel yang digunakan dalam implementasi fitur Kerja Praktik.

---

## Overview

Database schema untuk Kerja Praktik terdiri dari beberapa tabel utama yang saling berelasi:

- **Core Tables**: `kerja_praktek`, `logbooks`, `pengajuan_sidangs`, `sidangs`
- **Management Tables**: `periode`, `dosen_periode`, `gelombang_sidang_kps`, `pengumuman`
- **Supporting Tables**: `projects` (shared dengan TA), `penyelias`

---

## 1. Tabel `kerja_praktek`

Tabel utama untuk menyimpan data pengajuan Kerja Praktik oleh mahasiswa.

### Struktur Tabel

| Column                       | Type            | Constraints                 | Description                                    |
| ---------------------------- | --------------- | --------------------------- | ---------------------------------------------- |
| `id`                         | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | ID unik pengajuan Kerja Praktik                |
| `project_id`                 | BIGINT UNSIGNED | FOREIGN KEY → `projects.id` | Relasi ke tabel projects                       |
| `dosen_id`                   | BIGINT UNSIGNED | FOREIGN KEY → `dosen.id`    | ID dosen pembimbing                            |
| `penyelia_id`                | BIGINT UNSIGNED | FOREIGN KEY → `penyelias.id`, NULLABLE | ID penyelia perusahaan (nullable saat pengajuan) |
| `judul`                      | VARCHAR(255)    | NULLABLE                    | Judul Kerja Praktik                            |
| `perusahaan`                 | VARCHAR(255)    | NULLABLE                    | Nama perusahaan tempat magang                  |
| `posisi`                     | VARCHAR(255)    | NULLABLE                    | Posisi/jabatan mahasiswa di perusahaan         |
| `tgl_awal`                   | DATE            | NULLABLE                    | Tanggal mulai magang                           |
| `tgl_akhir`                  | DATE            | NULLABLE                    | Tanggal selesai magang                         |
| `status`                     | ENUM            | NOT NULL, DEFAULT           | Status: 'pending', 'rejected', 'accepted'      |
| `alasan`                     | LONGTEXT        | NULLABLE                    | Alasan jika ditolak atau catatan               |
| `tipe_kp`                    | ENUM            | NULLABLE                    | Tipe KP: 'reguler', 'konversi'                 |
| `link_folder`                | TEXT            | NULLABLE                    | Link folder Google Drive/Dropbox Kerja Praktik |
| `link_surat_diterima_magang` | TEXT            | NULLABLE                    | Link surat penerimaan magang dari perusahaan   |
| `created_at`                 | TIMESTAMP       | NULLABLE                    | Waktu pembuatan                                |
| `updated_at`                 | TIMESTAMP       | NULLABLE                    | Waktu update terakhir                          |
| `deleted_at`                 | TIMESTAMP       | NULLABLE                    | Soft delete timestamp                          |

### Relasi

- **Belongs To**: `projects` (via `project_id`)
- **Belongs To**: `dosen` (via `dosen_id`)
- **Belongs To**: `penyelia` (via `penyelia_id`)
- **Has Many**: `logbooks` (polymorphic via `model_id`, `model_type`)

### Indexes

- Primary key: `id`
- Foreign key: `project_id`
- Foreign key: `dosen_id`
- Foreign key: `penyelia_id`
- Index: `status`

### Notes

- Menggunakan soft deletes
- `penyelia_id` nullable karena mahasiswa bisa mengajukan KP tanpa tempat magang
- Data perusahaan (`perusahaan`, `posisi`, `tgl_awal`, `tgl_akhir`) dilengkapi sebelum sidang
- Status `accepted` berarti pengajuan sudah diterima dan mahasiswa sudah memiliki dosen pembimbing
- `project_id` menghubungkan ke tabel `projects` yang berisi informasi mahasiswa dan periode
- `link_folder` dan `link_surat_diterima_magang` menggunakan TEXT untuk mendukung URL panjang

---

## 2. Tabel `penyelias`

Tabel untuk menyimpan data penyelia dari perusahaan tempat mahasiswa magang.

### Struktur Tabel

| Column       | Type            | Constraints                 | Description                   |
| ------------ | --------------- | --------------------------- | ----------------------------- |
| `id`         | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | ID unik penyelia              |
| `nama`       | VARCHAR(255)    | NOT NULL                    | Nama penyelia                 |
| `posisi`     | VARCHAR(255)    | NULLABLE                    | Jabatan/posisi penyelia       |
| `perusahaan` | VARCHAR(255)    | NULLABLE                    | Nama perusahaan               |
| `telepon`    | VARCHAR(255)    | NULLABLE                    | Nomor telepon penyelia        |
| `alamat`     | TEXT            | NULLABLE                    | Alamat perusahaan/penyelia    |
| `created_at` | TIMESTAMP       | NULLABLE                    | Waktu pembuatan               |
| `updated_at` | TIMESTAMP       | NULLABLE                    | Waktu update terakhir         |

### Relasi

- **Has One**: `kerjaPraktek` (via `penyelia_id`)

### Indexes

- Primary key: `id`

### Notes

- Data penyelia tersimpan terstruktur dan dapat digunakan kembali
- Satu penyelia dapat membimbing satu mahasiswa per record `kerja_praktek`
- Data ini dilengkapi mahasiswa sebelum mengajukan sidang

---

## 3. Tabel `logbooks`

Tabel untuk menyimpan logbook bimbingan Kerja Praktik. Menggunakan polymorphic relationship untuk mendukung multiple model types.

### Struktur Tabel

| Column       | Type            | Constraints                 | Description                              |
| ------------ | --------------- | --------------------------- | ---------------------------------------- |
| `id`         | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | ID unik logbook                          |
| `model_id`   | INTEGER         | INDEX                       | ID dari model terkait (polymorphic)      |
| `model_type` | VARCHAR(255)    | INDEX                       | Type model (polymorphic)                 |
| `tanggal`    | DATE            | NULLABLE                    | Tanggal bimbingan                        |
| `bab`        | INTEGER         | NOT NULL                    | Nomor bab logbook                        |
| `uraian`     | LONGTEXT        | NOT NULL                    | Uraian isi logbook                       |
| `dokumen`    | LONGTEXT        | NULLABLE                    | Link dokumen logbook                     |
| `status`     | ENUM            | DEFAULT 'pending'           | Status: 'pending', 'revised', 'accepted' |
| `created_at` | TIMESTAMP       | NULLABLE                    | Waktu pembuatan                          |
| `updated_at` | TIMESTAMP       | NULLABLE                    | Waktu update terakhir                    |
| `deleted_at` | TIMESTAMP       | NULLABLE                    | Soft delete timestamp                    |

### Relasi

- **Polymorphic**: `modelable()` - dapat berelasi dengan `KerjaPraktek` atau model lain
- **Belongs To**: `kerjaPraktek()` (via `model_id` ketika `model_type` = `App\Models\KerjaPraktek`)

### Indexes

- Primary key: `id`
- Index: `model_id`
- Index: `model_type`
- Composite index: `(model_id, model_type)`

### Notes

- Menggunakan soft deletes untuk menjaga data historis
- `dokumen` dapat berisi link atau path file
- Status `accepted` berarti logbook sudah disetujui dosen pembimbing
- `bab` digunakan untuk tracking progress per bab Kerja Praktik
- Shared table dengan modul Tugas Akhir (polymorphic relationship)

---

## 4. Tabel `gelombang_sidang_kps`

Tabel untuk menyimpan data gelombang sidang Kerja Praktik. Satu periode dapat memiliki multiple gelombang.

### Struktur Tabel

| Column           | Type            | Constraints                  | Description                                |
| ---------------- | --------------- | ---------------------------- | ------------------------------------------ |
| `id`             | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT  | ID unik gelombang sidang                   |
| `periode_id`     | BIGINT UNSIGNED | FOREIGN KEY → `periode.id`   | Relasi ke tabel periode                    |
| `nama_gelombang` | VARCHAR(255)    | NOT NULL                     | Nama gelombang (contoh: "Gelombang 1")     |
| `tgl_awal`       | DATE            | NOT NULL                     | Tanggal awal gelombang                     |
| `tgl_akhir`      | DATE            | NOT NULL                     | Tanggal akhir gelombang                    |
| `status`         | TINYINT         | NOT NULL, DEFAULT 0          | Status aktif (1) atau nonaktif (0)         |
| `created_at`     | TIMESTAMP       | NULLABLE                     | Waktu pembuatan                            |
| `updated_at`     | TIMESTAMP       | NULLABLE                     | Waktu update terakhir                      |
| `deleted_at`     | TIMESTAMP       | NULLABLE                     | Soft delete timestamp                      |

### Relasi

- **Belongs To**: `periode` (via `periode_id`)
- **Has Many**: `pengajuanSidang` (via `gelombang_id`)

### Indexes

- Primary key: `id`
- Foreign key: `periode_id`
- Index: `status`

### Notes

- Menggunakan soft deletes
- `status` = 1 berarti gelombang aktif dan mahasiswa dapat mengajukan sidang
- Satu periode dapat memiliki multiple gelombang sidang
- Hanya satu gelombang dapat aktif pada satu waktu (handled by business logic)

---

## 5. Tabel `pengajuan_sidangs`

Tabel untuk menyimpan data pengajuan sidang Kerja Praktik oleh mahasiswa.

### Struktur Tabel

| Column                 | Type            | Constraints                            | Description                              |
| ---------------------- | --------------- | -------------------------------------- | ---------------------------------------- |
| `id`                   | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT            | ID unik pengajuan sidang                 |
| `mahasiswa_id`         | BIGINT UNSIGNED | FOREIGN KEY → `mahasiswa.id`           | ID mahasiswa yang mengajukan sidang      |
| `pembimbing_id`        | BIGINT UNSIGNED | FOREIGN KEY → `dosen.id`               | ID dosen pembimbing                      |
| `gelombang_id`         | BIGINT UNSIGNED | FOREIGN KEY → `gelombang_sidang_kps.id`| ID gelombang sidang                      |
| `link_laporan_akhir`   | TEXT            | NOT NULL                               | Link laporan akhir Kerja Praktik         |
| `link_dokumen_penyelia`| TEXT            | NOT NULL                               | Link dokumen dari penyelia (surat selesai magang) |
| `created_at`           | TIMESTAMP       | NULLABLE                               | Waktu pembuatan                          |
| `updated_at`           | TIMESTAMP       | NULLABLE                               | Waktu update terakhir                    |
| `deleted_at`           | TIMESTAMP       | NULLABLE                               | Soft delete timestamp                    |

### Relasi

- **Belongs To**: `mahasiswa` (via `mahasiswa_id`)
- **Belongs To**: `pembimbing` (dosen, via `pembimbing_id`)
- **Belongs To**: `gelombang` (via `gelombang_id`)
- **Has One**: `sidang` (via `pengajuan_sidang_id`)

### Indexes

- Primary key: `id`
- Foreign key: `mahasiswa_id`
- Foreign key: `pembimbing_id`
- Foreign key: `gelombang_id`

### Notes

- Menggunakan soft deletes
- Syarat pengajuan: logbook bab 5 sudah di-ACC oleh dosen pembimbing
- Mahasiswa harus melengkapi data penyelia dan perusahaan sebelum mengajukan sidang
- `link_dokumen_penyelia` adalah dokumen keterangan selesai magang dari perusahaan

---

## 6. Tabel `sidangs`

Tabel untuk menyimpan data sidang dan penilaian Kerja Praktik.

### Struktur Tabel

| Column                | Type            | Constraints                               | Description                                      |
| --------------------- | --------------- | ----------------------------------------- | ------------------------------------------------ |
| `id`                  | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT               | ID unik sidang                                   |
| `pengajuan_sidang_id` | BIGINT UNSIGNED | FOREIGN KEY → `pengajuan_sidangs.id`      | Relasi ke tabel pengajuan_sidangs                |
| `tgl_sidang`          | DATE            | NULLABLE                                  | Tanggal pelaksanaan sidang                       |
| `jam_sidang`          | TIME            | NULLABLE                                  | Jam pelaksanaan sidang                           |
| `nilai_penyelia`      | DECIMAL(5,2)    | NULLABLE                                  | Nilai dari penyelia (0-100)                      |
| `nilai_pembimbing`    | DECIMAL(5,2)    | NULLABLE                                  | Nilai dari dosen pembimbing (0-100)              |
| `nilai_penguji`       | DECIMAL(5,2)    | NULLABLE                                  | Nilai dari dosen penguji (0-100)                 |
| `nilai_akhir`         | DECIMAL(5,2)    | NULLABLE                                  | Nilai akhir (calculated: 40%+30%+30%)            |
| `status`              | ENUM            | DEFAULT 'belum_dinilai'                   | Status: 'belum_dinilai', 'lulus', 'tidak_lulus'  |
| `created_at`          | TIMESTAMP       | NULLABLE                                  | Waktu pembuatan                                  |
| `updated_at`          | TIMESTAMP       | NULLABLE                                  | Waktu update terakhir                            |
| `deleted_at`          | TIMESTAMP       | NULLABLE                                  | Soft delete timestamp                            |

### Relasi

- **Belongs To**: `pengajuanSidang` (via `pengajuan_sidang_id`)

### Indexes

- Primary key: `id`
- Foreign key: `pengajuan_sidang_id`
- Index: `status`

### Notes

- Menggunakan soft deletes
- `tgl_sidang` dan `jam_sidang` diisi oleh koordinator saat plotting jadwal
- `nilai_akhir` dihitung otomatis menggunakan formula:
  - **40%** nilai penyelia
  - **30%** nilai pembimbing
  - **30%** nilai penguji
- Nilai hanya dapat diinput setelah tanggal dan jam sidang terlewati
- Status berubah menjadi 'lulus' atau 'tidak_lulus' setelah penilaian selesai

---

## 7. Tabel `periode`

Tabel untuk menyimpan data periode ajaran. Digunakan untuk mengelola periode Kerja Praktik.

### Struktur Tabel

| Column          | Type            | Constraints                 | Description                         |
| --------------- | --------------- | --------------------------- | ----------------------------------- |
| `id`            | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | ID unik periode                     |
| `kode_periode`  | INTEGER         | NOT NULL                    | Kode periode (dari UDI)             |
| `tahun_ajaran`  | VARCHAR(255)    | NOT NULL                    | Tahun ajaran (contoh: "2024/2025")  |
| `tgl_awal`      | DATE            | NULLABLE                    | Tanggal awal periode umum           |
| `tgl_akhir`     | DATE            | NULLABLE                    | Tanggal akhir periode umum          |
| `tgl_awal_kp`   | DATE            | NULLABLE                    | Tanggal awal periode Kerja Praktik  |
| `tgl_akhir_kp`  | DATE            | NULLABLE                    | Tanggal akhir periode Kerja Praktik |
| `alokasi_kuota` | INTEGER         | NULLABLE                    | Alokasi kuota mahasiswa per periode |
| `status`        | TINYINT         | NULLABLE                    | Status aktif (1) atau nonaktif (0)  |
| `created_at`    | TIMESTAMP       | NULLABLE                    | Waktu pembuatan                     |
| `updated_at`    | TIMESTAMP       | NULLABLE                    | Waktu update terakhir               |
| `deleted_at`    | TIMESTAMP       | NULLABLE                    | Soft delete timestamp               |

### Relasi

- **Has Many**: `dosen_periode` (via `periode_id`)
- **Has Many**: `gelombangSidangKps` (via `periode_id`)
- **Has Many**: `projects` (via `periode_mulai_id` atau `periode_selesai_id`)

### Indexes

- Primary key: `id`
- Index: `status`

### Notes

- Menggunakan soft deletes
- `status` = 1 berarti periode aktif
- `tgl_awal_kp` dan `tgl_akhir_kp` khusus untuk periode Kerja Praktik
- `kode_periode` disinkronkan dari UDI/SIADIN
- Sistem mendukung multiple periode aktif secara bersamaan (perbaikan dari sistem lama)

---

## 8. Tabel `dosen_periode`

Tabel untuk menyimpan data kuota dosen pembimbing per periode. Menghubungkan dosen dengan periode dan menyimpan informasi kuota.

### Struktur Tabel

| Column       | Type            | Constraints                 | Description                                  |
| ------------ | --------------- | --------------------------- | -------------------------------------------- |
| `id`         | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | ID unik relasi dosen-periode                 |
| `dosen_id`   | BIGINT UNSIGNED | FOREIGN KEY → `dosen.id`    | ID dosen pembimbing                          |
| `periode_id` | BIGINT UNSIGNED | FOREIGN KEY → `periode.id`  | ID periode                                   |
| `kategori`   | VARCHAR(255)    | NULLABLE                    | Kategori dosen (jika ada)                    |
| `kuota`      | INTEGER         | NOT NULL                    | Kuota mahasiswa yang dapat dibimbing         |
| `link_wa`    | VARCHAR(255)    | NULLABLE                    | Link grup WhatsApp untuk mahasiswa bimbingan |
| `created_at` | TIMESTAMP       | NULLABLE                    | Waktu pembuatan                              |
| `updated_at` | TIMESTAMP       | NULLABLE                    | Waktu update terakhir                        |
| `deleted_at` | TIMESTAMP       | NULLABLE                    | Soft delete timestamp                        |

### Relasi

- **Belongs To**: `dosen()` (via `dosen_id`)
- **Belongs To**: `periode()` (via `periode_id`)

### Indexes

- Primary key: `id`
- Foreign key: `dosen_id`
- Foreign key: `periode_id`
- Composite index: `(dosen_id, periode_id)`

### Notes

- Menggunakan soft deletes
- `kuota` dapat diupdate per periode
- `link_wa` digunakan untuk komunikasi dengan mahasiswa bimbingan
- Satu dosen dapat memiliki beberapa record untuk periode berbeda
- Sistem baru mengelola kuota dengan baik (fix dari error sistem lama)

---

## 9. Tabel `pengumuman`

Tabel untuk menyimpan pengumuman Kerja Praktik. Shared table dengan modul lain (TA, BK, Alumni).

### Struktur Tabel

| Column         | Type            | Constraints                 | Description                        |
| -------------- | --------------- | --------------------------- | ---------------------------------- |
| `id`           | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | ID unik pengumuman                 |
| `judul`        | VARCHAR(255)    | NOT NULL                    | Judul pengumuman                   |
| `type`         | ENUM            | DEFAULT 'kp'                | Type: 'ta', 'kp', 'bk', 'alumni'   |
| `user`         | VARCHAR(255)    | NOT NULL                    | User yang membuat pengumuman       |
| `isi`          | LONGTEXT        | NOT NULL                    | Isi pengumuman (dapat berisi HTML) |
| `published_at` | TIMESTAMP       | NULLABLE                    | Waktu publikasi pengumuman         |
| `created_at`   | TIMESTAMP       | NULLABLE                    | Waktu pembuatan                    |
| `updated_at`   | TIMESTAMP       | NULLABLE                    | Waktu update terakhir              |

### Relasi

- Tidak ada relasi langsung dengan tabel Kerja Praktik lainnya

### Indexes

- Primary key: `id`
- Index: `type`

### Notes

- `type` = 'kp' untuk pengumuman Kerja Praktik
- `isi` dapat berisi HTML dari text editor
- `published_at` digunakan untuk scheduling pengumuman
- Shared table dengan modul TA, BK, dan Alumni

---

## 10. Tabel `projects` (Shared)

Tabel untuk menyimpan data project mahasiswa. Shared dengan modul Tugas Akhir.

### Struktur Tabel (Relevant Fields)

| Column               | Type            | Constraints                          | Description                        |
| -------------------- | --------------- | ------------------------------------ | ---------------------------------- |
| `id`                 | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT          | ID unik project                    |
| `mahasiswa_id`       | BIGINT UNSIGNED | FOREIGN KEY → `mahasiswa.id`         | ID mahasiswa                       |
| `dosen_id`           | BIGINT UNSIGNED | FOREIGN KEY → `dosen.id`, NULLABLE   | ID dosen (nullable saat pengajuan) |
| `jenis_projek_id`    | BIGINT UNSIGNED | FOREIGN KEY → `jenis_projeks.id`     | ID jenis project (TA/KP)           |
| `periode_mulai_id`   | BIGINT UNSIGNED | FOREIGN KEY → `periode.id`           | ID periode mulai                   |
| `periode_selesai_id` | BIGINT UNSIGNED | FOREIGN KEY → `periode.id`, NULLABLE | ID periode selesai (nullable)      |
| `created_at`         | TIMESTAMP       | NULLABLE                             | Waktu pembuatan                    |
| `updated_at`         | TIMESTAMP       | NULLABLE                             | Waktu update terakhir              |
| `deleted_at`         | TIMESTAMP       | NULLABLE                             | Soft delete timestamp              |

### Relasi

- **Belongs To**: `mahasiswa()` (via `mahasiswa_id`)
- **Belongs To**: `dosen()` (via `dosen_id`)
- **Belongs To**: `periodeMulai()` (via `periode_mulai_id`)
- **Has One**: `kerjaPraktek()` (via `project_id`)

### Notes

- Tabel ini shared dengan modul Tugas Akhir
- `dosen_id` di `projects` mungkin berbeda dengan `dosen_id` di `kerja_praktek` (plotting)
- `kerja_praktek` berelasi dengan `projects` untuk mendapatkan informasi mahasiswa

---

## Entity Relationship Diagram (ERD) Summary

```
┌─────────────┐
│   projects  │
│─────────────│
│ id (PK)     │
│ mahasiswa_id│
│ dosen_id    │
│ periode_id  │
└──────┬──────┘
       │
       │ 1:1
       │
┌──────▼──────────┐
│ kerja_praktek   │
│─────────────────│
│ id (PK)         │
│ project_id (FK) │◄──┐
│ dosen_id (FK)   │   │
│ penyelia_id(FK) │   │
│ status          │   │
│ link_folder     │   │
│ tipe_kp         │   │
└──────┬──────────┘   │
       │              │
       │ 1:N          │ 1:1
       │              │
┌──────▼──────────┐   │
│   logbooks      │   │
│─────────────────│   │
│ id (PK)         │   │
│ model_id        │───┘
│ model_type      │
│ bab             │
│ status          │
└─────────────────┘

       ┌─────────────┐
       │  penyelias  │
       │─────────────│
       │ id (PK)     │
       │ nama        │
       │ perusahaan  │
       └──────┬──────┘
              │
              │ 1:1
              │
       ┌──────▼──────────────┐
       │   kerja_praktek     │
       │─────────────────────│
       │ penyelia_id (FK)    │
       └─────────────────────┘

┌─────────────┐
│   periode   │
│─────────────│
│ id (PK)     │
│ kode_periode│
│ tgl_awal_kp │
│ tgl_akhir_kp│
│ status      │
└──────┬──────┘
       │
       │ 1:N
       │
┌──────▼──────────────┐
│ gelombang_sidang_kps│
│─────────────────────│
│ id (PK)             │
│ periode_id (FK)     │
│ nama_gelombang      │
│ tgl_awal            │
│ tgl_akhir           │
│ status              │
└──────┬──────────────┘
       │
       │ 1:N
       │
┌──────▼──────────────┐
│ pengajuan_sidangs   │
│─────────────────────│
│ id (PK)             │
│ mahasiswa_id (FK)   │
│ pembimbing_id (FK)  │
│ gelombang_id (FK)   │
│ link_laporan_akhir  │
└──────┬──────────────┘
       │
       │ 1:1
       │
┌──────▼──────────────┐
│     sidangs         │
│─────────────────────│
│ id (PK)             │
│ pengajuan_sidang_id │
│ tgl_sidang          │
│ jam_sidang          │
│ nilai_penyelia      │
│ nilai_pembimbing    │
│ nilai_penguji       │
│ nilai_akhir         │
│ status              │
└─────────────────────┘

┌─────────────┐
│   periode   │
│─────────────│
│ id (PK)     │
└──────┬──────┘
       │
       │ 1:N
       │
┌──────▼──────────┐
│ dosen_periode   │
│─────────────────│
│ id (PK)         │
│ dosen_id (FK)   │
│ periode_id (FK) │
│ kuota           │
│ link_wa         │
└─────────────────┘
```

---

## Indexes & Performance

### Recommended Indexes

1. **kerja_praktek**
   - Index on `status` untuk filtering
   - Index on `dosen_id` untuk query dosen
   - Index on `tipe_kp` untuk filtering
   - Composite index on `(project_id, status)`

2. **logbooks**
   - Composite index on `(model_id, model_type)` untuk polymorphic queries
   - Index on `status` untuk filtering
   - Index on `bab` untuk sorting

3. **gelombang_sidang_kps**
   - Index on `status` untuk filtering gelombang aktif
   - Index on `periode_id` untuk filtering per periode
   - Composite index on `(periode_id, status)`

4. **pengajuan_sidangs**
   - Index on `gelombang_id` untuk filtering per gelombang
   - Index on `mahasiswa_id` untuk lookup mahasiswa
   - Index on `pembimbing_id` untuk query dosen

5. **sidangs**
   - Index on `status` untuk filtering
   - Index on `pengajuan_sidang_id` (already FK index)
   - Index on `tgl_sidang` untuk sorting jadwal

6. **dosen_periode**
   - Composite index on `(dosen_id, periode_id)` untuk unique lookup
   - Index on `periode_id` untuk filtering per periode

---

## Constraints & Business Rules

### Foreign Key Constraints

- `kerja_praktek.project_id` → `projects.id` (CASCADE on delete)
- `kerja_praktek.dosen_id` → `dosen.id` (RESTRICT on delete)
- `kerja_praktek.penyelia_id` → `penyelias.id` (SET NULL on delete)
- `logbooks.model_id` → polymorphic (no FK constraint)
- `gelombang_sidang_kps.periode_id` → `periode.id` (CASCADE on delete)
- `pengajuan_sidangs.mahasiswa_id` → `mahasiswa.id` (CASCADE on delete)
- `pengajuan_sidangs.pembimbing_id` → `dosen.id` (RESTRICT on delete)
- `pengajuan_sidangs.gelombang_id` → `gelombang_sidang_kps.id` (RESTRICT on delete)
- `sidangs.pengajuan_sidang_id` → `pengajuan_sidangs.id` (CASCADE on delete)
- `dosen_periode.dosen_id` → `dosen.id` (CASCADE on delete)
- `dosen_periode.periode_id` → `periode.id` (CASCADE on delete)

### Business Rules

1. **Pengajuan Kerja Praktik**
   - Status harus 'accepted' sebelum dapat membuat logbook
   - Satu mahasiswa hanya dapat memiliki satu `kerja_praktek` dengan status 'accepted' per periode
   - `penyelia_id` nullable saat pengajuan, wajib diisi sebelum mengajukan sidang

2. **Logbook**
   - Hanya dapat dibuat jika `kerja_praktek.status` = 'accepted'
   - Status logbook: 'pending' → 'revised' atau 'accepted'
   - Logbook dengan status 'accepted' tidak dapat diubah

3. **Gelombang Sidang**
   - Hanya satu gelombang dapat aktif (`status` = 1) per periode pada satu waktu
   - Mahasiswa hanya dapat mengajukan sidang pada gelombang yang aktif
   - Tanggal `tgl_awal` harus lebih kecil dari `tgl_akhir`

4. **Pengajuan Sidang**
   - Hanya dapat dibuat jika logbook bab 5 sudah di-ACC
   - Hanya dapat dibuat jika `kerja_praktek.status` = 'accepted'
   - Data penyelia dan perusahaan harus sudah lengkap
   - Harus mengupload link laporan akhir dan dokumen penyelia

5. **Sidang & Penilaian**
   - `tgl_sidang` dan `jam_sidang` diisi koordinator saat plotting
   - Nilai hanya dapat diinput setelah tanggal dan jam sidang terlewati
   - `nilai_akhir` dihitung otomatis: (nilai_penyelia × 0.4) + (nilai_pembimbing × 0.3) + (nilai_penguji × 0.3)
   - Status berubah menjadi 'lulus' setelah semua nilai diinput

6. **Periode**
   - Sistem mendukung multiple periode aktif secara bersamaan
   - Periode aktif harus memiliki `tgl_awal_kp` dan `tgl_akhir_kp`

7. **Dosen Periode**
   - Kuota tidak boleh negatif
   - Satu dosen dapat memiliki multiple record untuk periode berbeda
   - Kuota terkelola dengan baik (fix dari error sistem lama)

---

## Data Synchronization

### UDI/SIADIN Integration

Data yang disinkronkan dari UDI/SIADIN:

1. **Periode**
   - `kode_periode`
   - `tahun_ajaran`
   - `tgl_awal`, `tgl_akhir`

2. **Dosen**
   - Data dosen dari tabel `dosen` (shared table)
   - Eliminasi input manual berulang

3. **Mahasiswa**
   - Data mahasiswa dari tabel `mahasiswa` (shared table)
   - Status KRS Kerja Praktik
   - Status kelulusan

### Sync Frequency

- **Periode**: Manual sync atau scheduled (harian)
- **Dosen**: Manual sync atau scheduled (mingguan) - **OTOMATIS**, tidak perlu input manual lagi
- **Mahasiswa**: Manual sync atau scheduled (harian)

---

## Migration History

Migration files terkait Kerja Praktik:

1. `2024_09_06_171727_create_periodes_table.php` - Create periode table
2. `2024_10_01_020908_create_pengumumans_table.php` - Create pengumuman table
3. `2024_10_07_004131_create_dosen_periodes_table.php` - Create dosen_periode table
4. `2024_10_10_133123_create_logbooks_table.php` - Create logbooks table (shared)
5. `2024_10_15_150234_create_kerja_prakteks_table.php` - Create kerja_praktek table
6. `2024_10_20_081530_create_penyelias_table.php` - Create penyelias table
7. `2024_11_05_094521_create_gelombang_sidang_kps_table.php` - Create gelombang_sidang_kps table
8. `2024_12_01_061625_create_pengajuan_sidangs_table.php` - Create pengajuan_sidangs table
9. `2024_12_01_142625_create_sidangs_table.php` - Create sidangs table
10. `2025_01_16_045441_add_tanggal_on_logbook_table.php` - Add tanggal to logbooks
11. `2025_03_05_074405_add_type_to_pengumumans_table.php` - Add type to pengumuman
12. `2025_04_30_032842_add_link_wa_to_dosen_periode_table.php` - Add link_wa to dosen_periode
13. `2025_06_29_150445_add_tgl_kp_to_periode_table.php` - Add tgl_awal_kp and tgl_akhir_kp
14. `2025_07_06_145630_update_link_fields_kerja_praktek.php` - Update link fields to TEXT

---

## Comparison with Old System

### Problems Fixed in New System

| Old System Problem                      | New System Solution                            |
| --------------------------------------- | ---------------------------------------------- |
| Error pengajuan karena kuota bermasalah | Sistem kuota terkelola dengan baik             |
| Hanya support 1 periode aktif           | Support multiple periode aktif secara bersamaan|
| Input data dosen berulang               | Sinkronisasi otomatis dengan UDI               |
| Data penyelia tidak terstruktur         | Tabel `penyelias` untuk data terstruktur       |
| Tidak ada manajemen gelombang sidang    | Tabel `gelombang_sidang_kps` untuk multiple gelombang |
| Monitoring sidang sulit                 | Tabel terpisah untuk pengajuan dan pelaksanaan sidang |

---

## Notes

- Semua tabel menggunakan `timestamps()` (created_at, updated_at)
- Tabel `kerja_praktek`, `logbooks`, `gelombang_sidang_kps`, `pengajuan_sidangs`, `sidangs`, `periode`, `dosen_periode` menggunakan soft deletes
- Polymorphic relationship pada `logbooks` memungkinkan penggunaan untuk model lain di masa depan
- Tabel `projects` dan `logbooks` adalah shared table dengan modul Tugas Akhir
- Tabel `pengumuman` adalah shared table dengan modul TA, BK, dan Alumni
- Sistem baru memperbaiki semua masalah yang ada di sistem lama

---

**Last Updated**: December 2025  
**Version**: 1.0
