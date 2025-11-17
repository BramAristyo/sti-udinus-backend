# Technical Architecture & Database Schema
## STI API - Sistem Informasi Kerja Praktik (Laravel + MySQL)

**Version:** 2.0  
**Date:** December, 2025  
**Stack:** Laravel 11.x | MySQL  
**Deployment:** Self-hosted  
**Multi-user:** Yes (with data isolation)  
**Sub-Applications:** Tugas Akhir, Kerja Praktik, Bimbingan Karir, Alumni
**Author:** Bram Aristyo, Timotius Kelvin

---

## 1. System Architecture Overview

### 1.1 Architecture Diagram

```
┌───────────────────────────────────────────────────────────────┐
│                     Laravel Application (API)                 │
│                                                               │
│ ┌───────────────────────────────────────────────────────────┐ │
│ │                        Route Layer                        │ │
│ │  routes/web/v1/kp/* (REST API endpoints, Sanctum protected) │ │
│ └───────────┬───────────────────────────────────────────────┘ │
│             │ Requests (HTTP JSON)                            │
│ ┌───────────▼───────────────────────────────────────────────┐ │
│ │                     Controller Layer                      │ │
│ │ AuthController / PengajuanController / LogbookController  │ │
│ │ Mengatur request → call service → response JSON           │ │
│ └───────────┬───────────────────────────────────────────────┘ │
│             │ Business logic executed via Service Layer       │
│ ┌───────────▼───────────────────────────────────────────────┐ │
│ │                       Service Layer                       │ │
│ │ PengajuanService / LogbookService / SidangService         │ │
│ │ Business logic, workflows, calling repositories           │ │
│ │ (ex: kuota management, sidang plotting, penilaian)        │ │
│ └───────────┬───────────────────────────────────────────────┘ │
│             │ Request Validation & Resources                  │
│ ┌───────────▼───────────────────────────────────────────────┐ │
│ │            HTTP Request & Response Layer                  │ │
│ │ Form Request (Validation)                                 │ │
│ │ API Resources (Format JSON Response)                      │ │
│ └───────────┬───────────────────────────────────────────────┘ │
│             │ Calls ORM / Models                              │
│ ┌───────────▼───────────────────────────────────────────────┐ │
│ │                  Model & ORM Layer                        │ │
│ │ Eloquent Models (KerjaPraktek, Logbook, Sidang, etc)     │ │
│ │ Relationship (hasMany, belongsTo, polymorphic)            │ │
│ └───────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘


┌───────────────────────────────────────────────────────────────┐
│                        MySQL Database                         │
│                                                               │
│ ┌───────────────────────────────────────────────────────────┐ │
│ │ Users, Roles, Permissions (Spatie)                        │ │
│ │ Kerja Praktik: kerja_praktek, penyelias, logbooks        │ │
│ │ gelombang_sidang_kps, pengajuan_sidangs, sidangs         │ │
│ │ periode, dosen_periode, pengumuman, projects             │ │
│ │ Activity logs, error logs, tokens                         │ │
│ └───────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘
```

### 1.2 Key Architectural Principles

**1. API-First & Headless Architecture**
- Laravel digunakan murni sebagai backend REST API 
- Frontend (Next.js/mobile) terpisah sepenuhnya 
- Semua komunikasi melalui HTTP JSON + Laravel Sanctum
- Dokumentasi API terstandarisasi (Swagger/OpenAPI)
- Mendukung integrasi multi-platform (web, mobile)
- Multi-sub-application architecture (TA, KP, BK, Alumni)

**2. Strict Authentication & Authorization**
- Laravel Sanctum untuk token-based authentication
- SSO Google OAuth untuk dosen dan mahasiswa
- Role & Permission menggunakan Spatie Laravel Permission
- Access control pada setiap endpoint (middleware: auth:sanctum, role, can)
- Tidak ada akses resource cross-role (koordinator, dosen, mahasiswa)
- Menjamin setiap user hanya mengakses data mereka sendiri
- Permission-based access (contoh: 'kerja-praktek', 'tugas-akhir')

**3. Service-Oriented Layered Architecture**
- Controller ringan (hanya routing request → service → response)
- Service layer mengelola business logic
- Reusable business functions
- Testing lebih mudah
- Penerapan Separation of Concerns (SoC) & Single Responsibility Principle (SRP)

**4. Validation & Structured API Responses**
- Form Request untuk validasi input konsisten
- API Resource untuk response JSON terformat
- Standar respon:
    - success / message / data
    - error detail saat validasi gagal
- Mencegah data tidak valid masuk ke sistem

**5. Secure & Scalable Database Design**
- Relasi terstruktur: Projects → KerjaPraktek → Logbooks → PengajuanSidang → Sidang
- Pivot tables untuk dosen_periode (kuota management)
- Polymorphic relationships untuk logbooks (reusable dengan TA)
- Soft deletes untuk data retention
- Logs untuk aktivitas penting (Spatie Activity Log)
- Support horizontal scaling untuk future microservices
- **Multiple periode aktif bersamaan** (fix dari sistem lama)
- **Kuota dosen terkelola baik** (fix dari error sistem lama)

**6. Audit, Logging & Traceability**
- Spatie Activity Log untuk tracking aktivitas user
- Custom ErrorLog model untuk exception tracking
- Database logs untuk aktivitas user, pengajuan, logbook, sidang
- Activity logs dapat difilter berdasarkan role dan tipe user
- Monitoring issue & debugging lebih mudah

**7. Extensible & Maintainable System**
- Architecture mendukung penambahan sub-application baru
- Modular folder structure per sub-application (TA, KP, BK, Alumni)
- Service layer terpisah per modul untuk maintainability
- Versioning API (V1, V2) untuk backward compatibility
- Shared services untuk common functionality
- **Integrasi UDI otomatis** untuk eliminasi input manual berulang

---

## 2. Technology Stack Details

### 2.1 Backend Stack

**Framework:** Laravel 11.x
- Framework PHP modern untuk aplikasi backend
- Eloquent ORM untuk manajemen database
- Middleware & Pipeline untuk keamanan API
- Struktur modular (Controller, Service, Request, Resource)
- Mendukung pengembangan scalable & clean architecture

**Database:** MySQL 8.0.43
- ACID compliant untuk data transaksi KP (pengajuan, logbook, sidang)
- JSON field support untuk data fleksibel (misal metadata penyelia)
- Indexing & relasi kompleks (gelombang sidang, dosen periode, dll)
- Mendukung full-text search untuk pencarian mahasiswa/dosen
- Support multiple active periode (improvement dari sistem lama)

**Storage:** MinIO
- Object storage untuk upload file seperti dokumen sidang, laporan KP, surat magang
- Terintegrasi dengan Laravel Filesystem (Storage::disk('minio'))
- Memudahkan deploy file storage terpisah dari server aplikasi
- Alternatif scalable untuk cloud storage mirip AWS S3

**Additional Backend Libraries:**
```
laravel/sanctum                 # Token-based authentication untuk API
laravel/socialite               # OAuth authentication (Google SSO)
spatie/laravel-permission       # Role & Permission system
spatie/laravel-activitylog      # Activity logging & audit trail
maatwebsite/excel               # Export/import data Excel (dosen, mahasiswa)
barryvdh/laravel-dompdf         # Generate PDF daftar nilai
phpoffice/phpword               # Generate Word documents
tecnickcom/tcpdf                # Advanced PDF generation
kreait/laravel-firebase         # Firebase Cloud Messaging untuk notifikasi
zircote/swagger-php             # API documentation (OpenAPI/Swagger)
```

### 2.2 Development & Deployment

**Development:**
```
Application: Laravel development server (php artisan serve)
Database: MySQL 8.0.43
Queue & Scheduler: Queue jobs via php artisan queue:work
Storage: MinIO untuk upload files
Logging: custom Log::insert (insert, update, delete)
Sync: UDI/SIADIN API integration untuk data dosen & mahasiswa
```

**Production (Self-Hosted):**
```
Application: PHP-FPM + Nginx (serve Laravel API)
Reverse Proxy: Nginx
Process Manager: Supervisor untuk queue worker / schedule
Database: MySQL 8.0.43 
Queue & Scheduler: Laravel queue jobs & cron scheduler
Storage: MinIO untuk upload files
Logging: Laravel default logs + custom Log::(insert, update, delete)
Sync: Automated sync dengan UDI/SIADIN (eliminasi input manual)
```

---

## 3. Project Structure

```
STI-API/
├── .github
|   └── workflows
|       └── main.yml        # GitHub Actions CI/CD (build, test, deploy)
├── app/
│   ├── Console/           # Artisan commands & Scheduler (tugas rutin/background)
│   ├── Constants/         # Menyimpan class berisi konstanta aplikasi
│   ├── Exceptions/        # Custom exception handler
│   ├── Export/            # Logic export data (Excel, PDF, CSV)
│   ├── Helpers/           # Fungsi utilitas umum/reusable
│   ├── Http/
│   │   ├── Controllers/   # Controller classes untuk API/web
│   │   │    ├── API/  
│   │   │    │    ├── Web/  
│   │   │    │    │   ├── V1/   
│   │   │    │    │   │   ├── KP/         # Kerja Praktik V1
│   │   │    │    │   │   │   ├── Auth/   # Authentication (SSO Google, Login)
│   │   │    │    │   │   │   ├── Koor/   # Koordinator controllers
│   │   │    │    │   │   │   │   ├── PeriodeController.php
│   │   │    │    │   │   │   │   ├── GelombangController.php
│   │   │    │    │   │   │   │   ├── DosenController.php
│   │   │    │    │   │   │   │   ├── MahasiswaController.php
│   │   │    │    │   │   │   │   ├── PengajuanController.php
│   │   │    │    │   │   │   │   ├── LogbookController.php
│   │   │    │    │   │   │   │   ├── SidangController.php
│   │   │    │    │   │   │   │   ├── PengumumanController.php
│   │   │    │    │   │   │   │   ├── DashboardController.php
│   │   │    │    │   │   │   │   └── LogAktivitasController.php
│   │   │    │    │   │   │   ├── Dosen/  # Dosen controllers
│   │   │    │    │   │   │   │   ├── PengajuanController.php
│   │   │    │    │   │   │   │   ├── LogbookController.php
│   │   │    │    │   │   │   │   ├── SidangController.php
│   │   │    │    │   │   │   │   ├── DashboardController.php
│   │   │    │    │   │   │   │   └── LogAktivitasController.php
│   │   │    │    │   │   │   └── Mahasiswa/ # Mahasiswa controllers
│   │   │    │    │   │   │       ├── PengajuanController.php
│   │   │    │    │   │   │       ├── LogbookController.php
│   │   │    │    │   │   │       ├── SidangController.php
│   │   │    │    │   │   │       ├── DashboardController.php
│   │   │    │    │   │   │       └── PenyeliaController.php
│   │   │    │    │   │   ├── TA/         # Tugas Akhir V1
│   │   │    │    │   │   ├── BK/         # Bimbingan Karir V1
│   │   │    │    │   │   └── Common/    # Shared controllers
│   │   │    │    │   └── V2/            # API versi 2
│   │   │    │    │       └── KP/        # Kerja Praktik V2
│   │   │    │    └── Mobile/            # Mobile API endpoints
│   │   │    │        └── Mahasiswa/     # Mobile mahasiswa endpoints
│   │   │    └── Controller.php          # Base controller (error handling)
│   │   ├── Middleware/    # Middleware untuk filter request (auth, role, throttle)
│   │   ├── Requests/      # Validasi request API/web
│   │   │    └── KP/       # KP Form Requests
│   │   │        ├── PengajuanRequest.php
│   │   │        ├── LogbookRequest.php
│   │   │        ├── SidangRequest.php
│   │   │        └── PenyeliaRequest.php
│   │   ├── Resources/     # API Resources untuk format response JSON
│   │   │    └── KP/       # KP API Resources
│   │   │        ├── PengajuanResource.php
│   │   │        ├── LogbookResource.php
│   │   │        ├── SidangResource.php
│   │   │        └── PenyeliaResource.php
│   │   └── Kernel.php     # Register middleware & schedule tasks
│   ├── Imports/           # Logic import data (mahasiswa via Excel)
│   ├── Jobs/              # Queue/background jobs
│   ├── Mail/              # Email notification logic
│   ├── Models/            # Eloquent models
│   │   ├── KerjaPraktek.php
│   │   ├── Penyelia.php
│   │   ├── Logbook.php
│   │   ├── GelombangSidangKp.php
│   │   ├── PengajuanSidang.php
│   │   ├── Sidang.php
│   │   ├── Periode.php
│   │   ├── DosenPeriode.php
│   │   └── Project.php
│   ├── Providers/         # Service providers
│   ├── Services/          # Business logic layer
│   │   ├── KP/            # Kerja Praktik services
│   │   │   ├── Pengajuan/ # PengajuanService
│   │   │   ├── Logbook/   # LogbookService
│   │   │   ├── Sidang/    # SidangService, GelombangService
│   │   │   ├── Periode/   # PeriodeService
│   │   │   ├── Dosen/     # DosenService, KuotaService
│   │   │   ├── Mahasiswa/ # MahasiswaService
│   │   │   ├── Penyelia/  # PenyeliaService
│   │   │   └── Koordinator/ # KoordinatorServices
│   │   ├── TA/            # Tugas Akhir services
│   │   ├── BK/            # Bimbingan Karir services
│   │   ├── Alumni/        # Alumni services
│   │   └── Common/        # Shared services (Auth, File, Image, UDI, etc)   
├── bootstrap/
│   └── app.php            # Bootstrapping Laravel
├── config/                # Configuration files
├── database/
│   ├── factories/         # Model factories for testing
│   ├── migrations/        # Database migrations
│   │   ├── 2024_09_06_171727_create_periodes_table.php
│   │   ├── 2024_10_01_020908_create_pengumumans_table.php
│   │   ├── 2024_10_07_004131_create_dosen_periodes_table.php
│   │   ├── 2024_10_10_133123_create_logbooks_table.php
│   │   ├── 2024_10_15_150234_create_kerja_prakteks_table.php
│   │   ├── 2024_10_20_081530_create_penyelias_table.php
│   │   ├── 2024_11_05_094521_create_gelombang_sidang_kps_table.php
│   │   ├── 2024_12_01_061625_create_pengajuan_sidangs_table.php
│   │   ├── 2024_12_01_142625_create_sidangs_table.php
│   │   ├── 2025_01_16_045441_add_tanggal_on_logbook_table.php
│   │   ├── 2025_03_05_074405_add_type_to_pengumumans_table.php
│   │   ├── 2025_04_30_032842_add_link_wa_to_dosen_periode_table.php
│   │   ├── 2025_06_29_150445_add_tgl_kp_to_periode_table.php
│   │   └── 2025_07_06_145630_update_link_fields_kerja_praktek.php
│   └── seeders/           # Database seeders
├── public/                # Public folder (entry point index.php)
│   └── assets/           
├── resources/
│   └── views/             # Blade templates frontend/email/certificate
│       ├── pdf/           # PDF templates (daftar nilai)
|       └── emails/        
├── routes/
│   └── api                # API routes
|       └── mobile/        
|       └── web/
|           └── v1/
|               └── kp/    # KP routes
├── storage/
│   ├── app/
│   ├── framework/
│   └── logs/              # Laravel default logs
├── tests/                 # PHPUnit tests
├── vendor/                # Composer dependencies
├── artisan                # Artisan CLI
├── composer.json          # Composer package info
├── .env                   # Environment variables
```

---

## 4. Data Security & Multi-Tenancy

### 4.1 User Isolation Strategy

**Row-Level Security:**
```php
// Mahasiswa hanya bisa akses KP mereka sendiri
$mahasiswaId = auth()->id();
$kerjaPraktek = KerjaPraktek::whereHas('project', function($query) use ($mahasiswaId) {
    $query->where('mahasiswa_id', $mahasiswaId);
})->first();
```

```php
// Dosen hanya bisa akses mahasiswa bimbingannya
$dosenId = auth()->id();
$logbooks = Logbook::where('model_type', KerjaPraktek::class)
    ->whereHas('modelable', function($query) use ($dosenId) {
        $query->where('dosen_id', $dosenId);
    })
    ->with('modelable.project.mahasiswa')
    ->get();
```

**Template-Level Safety:**
- Tidak hardcode ID di URL, menggunakan route model binding atau selalu filter dengan auth()->id()
```php
Route::get('/{pengajuanId}/detail', [PengajuanController::class, 'showPengajuan']);
```
- CSRF protection tidak diperlukan untuk API stateless, tetapi menggunakan token Sanctum auth
- Menggunakan hidden property di model
```php
protected $hidden = ['password','remember_token'];
```

### 4.2 Authentication & Authorization

**User Authentication:**
- Login dengan email/password menggunakan Laravel Sanctum
- SSO Google OAuth untuk dosen dan mahasiswa
- Password dienkripsi dengan bcrypt
- Role otomatis ditetapkan dengan Spatie Permission
- Koordinator dengan permission 'kerja-praktek' otomatis login ke UDI API

**Session Management:**
- Menggunakan Laravel Sanctum token untuk autentikasi
- Token dapat di-revoke per device
- Token tidak memiliki expiry time default (dapat dikonfigurasi)

**Permissions:**
- Akses diatur dengan Spatie Role & Permission
- Permission-based access: 'kerja-praktek', 'tugas-akhir', dll
- Pengguna hanya bisa mengakses datanya sendiri
- Koordinator memiliki akses penuh ke sub-application terkait
- Endpoint API dilindungi dengan middleware:
  - `auth:sanctum` - Authentication
  - `role:koordinator|dosen|mahasiswa` - Role-based access
  - `can:kerja-praktek` - Permission-based access

---

## 5. Key Service Layer Functions

### 5.1 Kerja Praktik - Pengajuan Service

```php
class PengajuanService {
    public function pengajuanKPMahasiswa($request) {
        // Validasi periode aktif
        // Create project jika belum ada
        // Create kerja_praktek dengan status 'pending'
        // penyelia_id nullable (mahasiswa bisa belum punya tempat magang)
        // Return kerja praktek data
    }

    public function updatePengajuanKP($request, $kerjaPraktekId) {
        // Update data pengajuan (judul, perusahaan, posisi, tgl_awal, tgl_akhir)
        // Hanya bisa update jika status masih 'pending'
        // Update penyelia jika ada
        // Return updated kerja praktek
    }

    public function updateStatusPengajuanByDosen($request, $kerjaPraktekId) {
        // Update status: pending, rejected, accepted
        // Hanya dosen pembimbing yang bisa update
        // Jika accepted, update project dosen_id
        // Validasi kuota dosen (fix dari error sistem lama)
        // Return updated kerja praktek
    }

    public function updateStatusPengajuanByKoor($request, $kerjaPraktekId) {
        // Update status: pending, rejected, accepted
        // Update project dosen_id jika accepted
        // Validasi kuota dosen
        // Return updated kerja praktek
    }

    public function getDosenPeriode($request) {
        // Get list dosen dengan kuota untuk periode aktif
        // Filter berdasarkan kuota tersedia
        // Sistem kuota terkelola baik (improvement dari sistem lama)
    }
}
```

### 5.2 Kerja Praktik - Logbook Service

```php
class LogbookService {
    public function createLogbook($request) {
        // Validasi kerja praktek status = 'accepted'
        // Create logbook dengan status 'pending'
        // Polymorphic relationship dengan KerjaPraktek
    }

    public function updateLogbook($request, $logbookId) {
        // Update logbook (tanggal, bab, uraian, dokumen)
        // Hanya bisa update jika status 'pending' atau 'revised'
        // Return updated logbook
    }

    public function updateStatusLogbook($logbookId, $status) {
        // Update status: pending, revised, accepted
        // Hanya dosen pembimbing yang bisa update
    }

    public function getLogbookByMahasiswa($mahasiswaId) {
        // Get all logbooks untuk mahasiswa
        // Filter by kerja praktek yang accepted
    }

    public function getLogbookByDosen($dosenId) {
        // Get all logbooks mahasiswa bimbingan dosen
        // Filter by kerja praktek yang accepted
        // Include mahasiswa info
    }
}
```

### 5.3 Kerja Praktik - Penyelia Service

```php
class PenyeliaService {
    public function createOrUpdatePenyelia($request) {
        // Create penyelia baru atau update existing
        // Data: nama, posisi, perusahaan, telepon, alamat
        // Return penyelia data
    }

    public function getPenyeliaByMahasiswa($mahasiswaId) {
        // Get penyelia dari kerja praktek mahasiswa
        // Return penyelia data atau null
    }
}
```

### 5.4 Kerja Praktik - Gelombang Service

```php
class GelombangService {
    public function createGelombang($request) {
        // Create gelombang sidang baru
        // Data: periode_id, nama_gelombang, tgl_awal, tgl_akhir, status
        // Hanya koordinator yang bisa create
        // Return gelombang data
    }

    public function updateGelombang($request, $gelombangId) {
        // Update gelombang sidang
        // Update tanggal, nama, atau status
        // Return updated gelombang
    }

    public function toggleStatusGelombang($gelombangId) {
        // Toggle status aktif/nonaktif
        // Hanya satu gelombang aktif per periode
        // Return updated gelombang
    }

    public function getGelombangAktif() {
        // Get gelombang yang aktif saat ini
        // Return gelombang data atau null
    }
}
```

### 5.5 Kerja Praktik - Sidang Service

```php
class SidangService {
    public function pengajuanSidangKP($request) {
        // Return pengajuan sidang data
    }

    public function updatePengajuanSidang($request, $pengajuanSidangId) {
        // Return updated pengajuan sidang
    }

    public function deletePengajuanSidang($pengajuanSidangId) {
    }

    public function plotJadwalSidang($request, $pengajuanSidangId) {
        // Return sidang data
    }

    public function inputNilaiSidang($request, $sidangId) {
        // Return sidang data
    }

    public function getJadwalSidangByMahasiswa($mahasiswaId) {
        // Return jadwal sidang
    }

    public function getJadwalSidangByDosen($dosenId) {
        // data jadwal sidang
    }
}
```

### 5.6 Kerja Praktik - Periode Service

```php
class PeriodeService {
    public function syncPeriodeFromUDI() {
        // Return synced periode data
    }

    public function updateTanggalPeriode($request, $periodeId) {
        // Return updated periode
    }

    public function toggleStatusPeriode($periodeId) {
        // Return updated periode
    }
}
```

### 5.7 Kerja Praktik - Dosen Service

```php
class DosenService {
    public function syncDosenFromUDI() {
        // Return synced dosen data
    }

    public function updateKuotaDosen($request, $dosenPeriodeId) {
        // Return updated dosen periode
    }

    public function plottingMahasiswaToDosen($request) {
        // Return plotting result
    }
}
```

### 5.8 Common - Auth Service

```php
class AuthService {
    public function siadinLogin() {
        // Return access_token dan refresh_token
    }

    public function googleOAuth($request) {
        // Handle Google OAuth callback
        // Create/update user
        // Generate Sanctum token
    }
}
```

---

## 6. API Endpoints

### 6.1 Kerja Praktik Endpoint Specification

```
# ===== Authentication =====
POST   /api/web/v1/kp/koor/login              # Login koordinator
POST   /api/web/v1/kp/auth/google             # SSO Google (dosen, mahasiswa)

# ===== Koordinator =====
# Dashboard
GET    /api/web/v1/kp/koor/dashboard/logbook-mahasiswa
GET    /api/web/v1/kp/koor/dashboard/jumlah-mahasiswa
GET    /api/web/v1/kp/koor/dashboard/kuota-dosen-pembimbing

# Periode
GET    /api/web/v1/kp/koor/periode            # List semua periode (support multiple aktif)
POST   /api/web/v1/kp/koor/periode/sync       # Sinkronisasi periode dari UDI
PATCH  /api/web/v1/kp/koor/periode/{id}/update # Update tanggal periode KP
DELETE /api/web/v1/kp/koor/periode/{id}/nonaktif # Nonaktifkan periode

# Gelombang Sidang
GET    /api/web/v1/kp/koor/gelombang          # List semua gelombang
POST   /api/web/v1/kp/koor/gelombang/add      # Tambah gelombang baru
PATCH  /api/web/v1/kp/koor/gelombang/{id}/update # Update gelombang
DELETE /api/web/v1/kp/koor/gelombang/{id}/delete # Hapus gelombang
PATCH  /api/web/v1/kp/koor/gelombang/{id}/toggle # Toggle status aktif/nonaktif

# Dosen
GET    /api/web/v1/kp/koor/dosen              # List semua dosen
POST   /api/web/v1/kp/koor/dosen/add          # Tambah dosen
GET    /api/web/v1/kp/koor/dosen/sync         # Sinkronisasi dosen dari UDI (eliminasi input manual)
GET    /api/web/v1/kp/koor/dosen/{id}         # Detail dosen
PATCH  /api/web/v1/kp/koor/dosen/{id}/update  # Update dosen
DELETE /api/web/v1/kp/koor/dosen/{id}/delete  # Hapus dosen

# Kuota Dosen
GET    /api/web/v1/kp/koor/dosen/kuota        # List kuota dosen (terkelola baik)
PATCH  /api/web/v1/kp/koor/dosen/kuota/{id}/update # Update kuota

# Mahasiswa
GET    /api/web/v1/kp/koor/mahasiswa          # List mahasiswa KP
POST   /api/web/v1/kp/koor/mahasiswa/add      # Tambah mahasiswa (manual/excel)
GET    /api/web/v1/kp/koor/mahasiswa/sync     # Sinkronisasi mahasiswa dari UDI
PATCH  /api/web/v1/kp/koor/mahasiswa/{id}/update-dospem # Update dosen pembimbing
PATCH  /api/web/v1/kp/koor/mahasiswa/{id}/update # Edit mahasiswa KP
GET    /api/web/v1/kp/koor/mahasiswa/{id}     # Detail mahasiswa KP

# Pengajuan
GET    /api/web/v1/kp/koor/mahasiswa/pengajuan/list
PUT    /api/web/v1/kp/koor/mahasiswa/pengajuan/update-status/{id}

# Logbook
GET    /api/web/v1/kp/koor/mahasiswa/logbook/rekap
GET    /api/web/v1/kp/koor/mahasiswa/logbook/detailLogbook/{dosenId}
PATCH  /api/web/v1/kp/koor/mahasiswa/logbook/{id}/accept # Accept logbook

# Sidang
GET    /api/web/v1/kp/koor/sidang/pengajuan   # List pengajuan sidang
GET    /api/web/v1/kp/koor/sidang/jadwal      # List jadwal sidang
PATCH  /api/web/v1/kp/koor/sidang/{id}/plot-jadwal # Plot jadwal sidang
PATCH  /api/web/v1/kp/koor/sidang/{id}/input-nilai # Input nilai sidang
GET    /api/web/v1/kp/koor/sidang/{id}/download-pdf # Download PDF daftar nilai

# Pengumuman
GET    /api/web/v1/kp/koor/pengumuman         # List pengumuman
POST   /api/web/v1/kp/koor/pengumuman/add     # Tambah pengumuman
PATCH  /api/web/v1/kp/koor/pengumuman/{id}/update # Update pengumuman
DELETE /api/web/v1/kp/koor/pengumuman/{id}/delete # Hapus pengumuman

# Tanda Tangan
POST   /api/web/v1/kp/koor/ttd/upload         # Upload tanda tangan
GET    /api/web/v1/kp/koor/ttd/preview        # Preview tanda tangan

# Activity Log
GET    /api/web/v1/kp/koor/activity/all       # Semua aktivitas
GET    /api/web/v1/kp/koor/activity/dosen     # Aktivitas dosen
GET    /api/web/v1/kp/koor/activity/mahasiswa # Aktivitas mahasiswa

# ===== Dosen =====
# Dashboard
GET    /api/web/v1/kp/dosen/dashboard/statistic-logbook

# Pengajuan
GET    /api/web/v1/kp/dosen/pengajuan         # List pengajuan mahasiswa bimbingan
PUT    /api/web/v1/kp/dosen/pengajuan/update-status/{id} # Accept/reject pengajuan

# Logbook
GET    /api/web/v1/kp/dosen/logbook           # List logbook mahasiswa bimbingan
PUT    /api/web/v1/kp/dosen/logbook/update-status/{id} # Accept/revise logbook
GET    /api/web/v1/kp/dosen/logbook/{id}      # Detail logbook

# Sidang
GET    /api/web/v1/kp/dosen/sidang/jadwal     # Jadwal sidang mahasiswa bimbingan
PATCH  /api/web/v1/kp/dosen/sidang/{id}/input-nilai # Input nilai sidang

# Pengumuman
GET    /api/web/v1/kp/dosen/pengumuman        # List pengumuman

# Tanda Tangan
POST   /api/web/v1/kp/dosen/ttd/upload        # Upload tanda tangan
GET    /api/web/v1/kp/dosen/ttd/preview       # Preview tanda tangan

# Activity Log
GET    /api/web/v1/kp/dosen/activity/mahasiswa # Aktivitas mahasiswa bimbingan

# ===== Mahasiswa =====
# Dashboard
GET    /api/web/v1/kp/mahasiswa/dashboard/progress
GET    /api/web/v1/kp/mahasiswa/dashboard/logbook-count

# Pengajuan
POST   /api/web/v1/kp/mahasiswa/pengajuan     # Buat pengajuan KP (dengan/tanpa tempat magang)
GET    /api/web/v1/kp/mahasiswa/pengajuan/show # Status pengajuan
PATCH  /api/web/v1/kp/mahasiswa/pengajuan/{id}/update # Update pengajuan
DELETE /api/web/v1/kp/mahasiswa/pengajuan/{id}/delete # Hapus pengajuan
GET    /api/web/v1/kp/mahasiswa/pengajuan/histori # Histori pengajuan
GET    /api/web/v1/kp/mahasiswa/pengajuan/dosen-periode # List dosen untuk dipilih

# Penyelia
GET    /api/web/v1/kp/mahasiswa/penyelia      # Get data penyelia
POST   /api/web/v1/kp/mahasiswa/penyelia      # Create/update penyelia
PATCH  /api/web/v1/kp/mahasiswa/penyelia/{id}/update # Update penyelia

# Logbook
GET    /api/web/v1/kp/mahasiswa/logbook       # List logbook
POST   /api/web/v1/kp/mahasiswa/logbook       # Buat logbook
PUT    /api/web/v1/kp/mahasiswa/logbook/{id}  # Update logbook
DELETE /api/web/v1/kp/mahasiswa/logbook/{id}  # Hapus logbook
GET    /api/web/v1/kp/mahasiswa/logbook/folder # Get/Set folder KP

# Sidang
GET    /api/web/v1/kp/mahasiswa/sidang/status # Get status sidang
POST   /api/web/v1/kp/mahasiswa/sidang        # Ajukan sidang (setelah bab 5 di-ACC)
PATCH  /api/web/v1/kp/mahasiswa/sidang/{id}/update # Update pengajuan sidang
DELETE /api/web/v1/kp/mahasiswa/sidang/{id}/delete # Hapus pengajuan sidang
GET    /api/web/v1/kp/mahasiswa/sidang/gelombang # List gelombang aktif

# Pengumuman
GET    /api/web/v1/kp/mahasiswa/pengumuman    # List pengumuman
```

### 6.2 API Response Format

**Response Sukses (Standard):**
```json
{
    "success": true,
    "message": "Berhasil menambahkan pengajuan",
    "data": {
        "id": 1,
        "judul": "Sistem Informasi Akademik",
        "perusahaan": "PT Example",
        "status": "pending"
    }
}
```

**Response Sukses (Meta Format):**
```json
{
    "meta": {
        "status_code": 200,
        "success": true,
        "message": "Data berhasil diambil"
    },
    "data": {
        "id": 1,
        "nama_gelombang": "Gelombang 1",
        "status": 1
    }
}
```

**Response Error (Validasi Gagal):**
```json
{
    "success": false,
    "message": {
        "judul": [
            "Judul KP wajib diisi."
        ],
        "dosen_id": [
            "Dosen pembimbing wajib dipilih."
        ]
    }
}
```

**Response Error (Business Logic):**
```json
{
    "success": false,
    "message": "Logbook bab 5 belum di-ACC, tidak bisa mengajukan sidang"
}
```

**Response Error (Kuota):**
```json
{
    "success": false,
    "message": "Kuota dosen pembimbing sudah penuh"
}
```

**Response Error (Exception):**
```json
{
    "success": false,
    "message": "Terjadi kesalahan saat memproses data",
    "errors": "Exception Error : InvalidArgumentException"
}
```

**Response Error (Not Found):**
```json
{
    "success": false,
    "message": "Data pengajuan tidak ditemukan"
}
```

**Response Error (Forbidden):**
```json
{
    "meta": {
        "status_code": 403,
        "success": false,
        "message": "Anda tidak memiliki akses ke resource ini"
    }
}
```

---

## 7. Database Performance Considerations

### 7.1 Query Optimization

**N+1 Query Prevention:**
```php
# BAD:
$kerjaPraktek = KerjaPraktek::all();
foreach ($kerjaPraktek as $kp) {
    echo $kp->project->mahasiswa->name; // N+1 query
    echo $kp->penyelia->nama; // N+1 query
}

# GOOD:
$kerjaPraktek = KerjaPraktek::with([
    'project.mahasiswa', 
    'dosen',
    'penyelia'
])->get();
foreach ($kerjaPraktek as $kp) {
    echo $kp->project->mahasiswa->name; // hanya 1 query dengan eager loading
    echo $kp->penyelia->nama; // sudah di-load
}
```

**Polymorphic Relationship Query:**
```php
// Logbook menggunakan polymorphic relationship (shared dengan TA)
$logbooks = Logbook::where('model_type', KerjaPraktek::class)
    ->where('model_id', $kerjaPraktekId)
    ->where('status', 'accepted')
    ->with('modelable.project.mahasiswa') // Eager load polymorphic relation
    ->get();
```

**Query with Gelombang Sidang:**
```php
// Get pengajuan sidang dengan gelombang aktif
$pengajuanSidang = PengajuanSidang::with([
    'mahasiswa',
    'pembimbing',
    'gelombang.periode',
    'sidang'
])
->whereHas('gelombang', function($query) {
    $query->where('status', 1); // gelombang aktif
})
->get();
```

**Aggregation on Database:**
```php
// Hitung jumlah logbook accepted per mahasiswa
$logbookCount = DB::table('logbooks')
    ->join('kerja_praktek', function($join) {
        $join->on('logbooks.model_id', '=', 'kerja_praktek.id')
             ->where('logbooks.model_type', '=', KerjaPraktek::class);
    })
    ->where('logbooks.status', 'accepted')
    ->whereNull('logbooks.deleted_at')
    ->select('kerja_praktek.id', DB::raw('count(*) as total'))
    ->groupBy('kerja_praktek.id')
    ->get();
```

**Query Builder Optimization:**
```php
// Select specific columns untuk mengurangi memory usage
$kerjaPraktek = KerjaPraktek::select('id', 'judul', 'status', 'dosen_id', 'perusahaan')
    ->where('status', 'accepted')
    ->with(['dosen:id,nama', 'penyelia:id,nama,perusahaan'])
    ->get();
```

**Multiple Periode Support:**
```php
// Query dengan multiple periode aktif (improvement dari sistem lama)
$periodeAktif = Periode::where('status', 1)
    ->where('tgl_awal_kp', '<=', now())
    ->where('tgl_akhir_kp', '>=', now())
    ->get(); // Bisa return multiple periode aktif
```

**Kuota Dosen Management:**
```php
// Query kuota dosen dengan validation (fix dari error sistem lama)
$dosenPeriode = DosenPeriode::where('dosen_id', $dosenId)
    ->where('periode_id', $periodeId)
    ->first();

$mahasiswaCount = KerjaPraktek::where('dosen_id', $dosenId)
    ->whereHas('project', function($query) use ($periodeId) {
        $query->where('periode_mulai_id', $periodeId);
    })
    ->where('status', 'accepted')
    ->count();

$kuotaTersedia = $dosenPeriode->kuota - $mahasiswaCount;
```

---

## 8. Deployment Checklist

### 8.1 Pre-Deployment

- [ ] Environment variables configured (.env file)
- [ ] Database migrations applied (termasuk migration KP)
- [ ] Storage dan permission disesuaikan
- [ ] Optimasi konfigurasi dan autoload
- [ ] APP_DEBUG dimatikan di environment production
- [ ] APP_KEY digenerate
- [ ] UDI API credentials configured
- [ ] MinIO storage configured
- [ ] Queue worker configured (Supervisor)

### 8.2 Post-Deployment

- [ ] Monitoring error log aktif
- [ ] Strategi backup database diterapkan
- [ ] SSL certificate aktif
- [ ] UDI sync scheduler berjalan
- [ ] Test sync periode, dosen, mahasiswa dari UDI
- [ ] Validasi kuota dosen berfungsi dengan baik
- [ ] Test multiple periode aktif
- [ ] Test gelombang sidang management

---

## 9. Improvements from Old System

### 9.1 Critical Fixes

**1. Kuota Dosen Management**
```php
// OLD SYSTEM: Error karena kuota tidak terkelola
// NEW SYSTEM: Validasi kuota real-time
public function validateKuotaDosen($dosenId, $periodeId) {
    $dosenPeriode = DosenPeriode::where('dosen_id', $dosenId)
        ->where('periode_id', $periodeId)
        ->first();
    
    if (!$dosenPeriode) {
        throw new Exception('Dosen tidak memiliki kuota untuk periode ini');
    }
    
    $mahasiswaCount = $this->countMahasiswaBimbingan($dosenId, $periodeId);
    
    if ($mahasiswaCount >= $dosenPeriode->kuota) {
        throw new Exception('Kuota dosen pembimbing sudah penuh');
    }
    
    return true;
}
```

**2. Multiple Periode Support**
```php
// OLD SYSTEM: Hanya 1 periode aktif, harus manual switch
// NEW SYSTEM: Multiple periode aktif bersamaan
$periodeAktif = Periode::where('status', 1)->get();
// Mahasiswa bisa mengajukan KP di periode manapun yang aktif
```

**3. Auto Sync Dosen dari UDI**
```php
// OLD SYSTEM: Input data dosen berulang secara manual
// NEW SYSTEM: Sinkronisasi otomatis dari UDI
public function syncDosenFromUDI() {
    $udiDosens = $this->udiApiService->getAllDosen();
    
    foreach ($udiDosens as $udiDosen) {
        Dosen::updateOrCreate(
            ['npp' => $udiDosen['npp']],
            [
                'nama' => $udiDosen['nama'],
                'email' => $udiDosen['email'],
                // ... fields lainnya
            ]
        );
    }
    
    return true; // Eliminasi input manual berulang
}
```

**4. Structured Penyelia Data**
```php
// OLD SYSTEM: Data penyelia tidak terstruktur
// NEW SYSTEM: Tabel penyelias terpisah, data rapi
$penyelia = Penyelia::create([
    'nama' => $request->nama_penyelia,
    'posisi' => $request->posisi_penyelia,
    'perusahaan' => $request->perusahaan,
    'telepon' => $request->telepon_penyelia,
    'alamat' => $request->alamat_perusahaan
]);

$kerjaPraktek->penyelia_id = $penyelia->id;
$kerjaPraktek->save();
```

**5. Gelombang Sidang Management**
```php
// OLD SYSTEM: Tidak ada manajemen gelombang
// NEW SYSTEM: Multiple gelombang per periode
$gelombang = GelombangSidangKp::create([
    'periode_id' => $periodeId,
    'nama_gelombang' => 'Gelombang 1',
    'tgl_awal' => '2025-01-01',
    'tgl_akhir' => '2025-01-31',
    'status' => 1 // aktif
]);

// Hanya 1 gelombang aktif per periode
GelombangSidangKp::where('periode_id', $periodeId)
    ->where('id', '!=', $gelombang->id)
    ->update(['status' => 0]);
```

### 9.2 Comparison Table

| Feature | Old System | New System |
|---------|-----------|------------|
| Multiple Periode | ❌ Hanya 1 periode | ✅ Multiple periode aktif |
| Kuota Dosen | ❌ Error, tidak terkelola | ✅ Terkelola baik dengan validasi |
| Input Dosen | ❌ Manual berulang | ✅ Auto sync dari UDI |
| Data Penyelia | ❌ Tidak terstruktur | ✅ Tabel terpisah, rapi |
| Gelombang Sidang | ❌ Tidak ada | ✅ Multiple gelombang per periode |
| Pengajuan Error | ❌ Sering error | ✅ Validasi komprehensif |
| Monitoring Sidang | ❌ Manual, sulit | ✅ Dashboard terpusat |

---

## 10. Future Extensions

### 10.1 Background Tasks
- Pengiriman notifikasi email untuk pengajuan, logbook, sidang menggunakan queue
- Automated reminder untuk mahasiswa yang belum mengajukan sidang
- Scheduled sync data dari UDI (periode, dosen, mahasiswa)
- Auto-generate laporan mingguan untuk koordinator

### 10.2 API for Mobile
- Satu basis API untuk platform web dan mobile dengan autentikasi berbasis Sanctum
- Endpoint mobile disederhanakan untuk efisiensi request dan respon data
- Integrasi dengan aplikasi mobile untuk akses pengajuan, logbook, dan sidang
- Firebase Cloud Messaging untuk push notification (pengajuan diterima/ditolak, logbook di-ACC, jadwal sidang)
- Support token revocation per device untuk keamanan
- Offline support untuk logbook drafts

### 10.3 Data Management and Reporting
- Fitur ekspor data ke format Spreadsheet (XLSX/CSV) untuk kebutuhan administrasi
- Fitur impor data mahasiswa dan dosen secara massal via Excel
- Sinkronisasi otomatis data dengan UDI/SIADIN API (periode, dosen, mahasiswa)
- Otomatisasi backup database dan laporan mingguan
- Export laporan rekap logbook, sidang, dan penilaian
- Generate PDF daftar nilai dengan tanda tangan digital
- Analytics dashboard untuk koordinator (trend pengajuan, distribusi nilai, dll)

### 10.4 Advanced Features
- AI-based recommendation untuk dosen pembimbing berdasarkan minat mahasiswa
- Smart progress tracking untuk prediksi kelulusan sidang
- Integrasi dengan sistem yudisium untuk otomasi kelulusan
- Real-time collaboration tools untuk bimbingan online
- Automated workflow untuk approval process multi-level
- Integration dengan sistem perusahaan untuk validasi penyelia

---

## 11. GitHub Repository Structure & Branching Workflow

### 11.1 Struktur Branch GitHub

Repository STI API menggunakan struktur branch yang terorganisir untuk memisahkan alur pengembangan dan produksi:

- **staging**  
  Branch utama untuk pengembangan fitur. Semua fitur baru, perbaikan bug, dan perubahan logic dilakukan di sini.

- **main/master**  
  Branch yang digunakan untuk kode produksi (live). Hanya menerima kode dari hasil merge staging yang sudah melalui review dan pengujian.

- **Feature Branch (dinamis)**  
  Dibuat setiap kali developer menambahkan atau memperbarui fitur tertentu. Contoh:
  - feat/kp-pengajuan
  - feat/kp-logbook
  - feat/kp-sidang
  - feat/kp-gelombang
  - fix/kp-kuota-bug
  - feat/kp-udi-integration

---

### 11.2 Alur Pengembangan & Update Code

#### 1. Membuat Branch Baru dari Staging

1. Checkout ke staging:
   ```bash
   git checkout staging
   ```

2. Pull perubahan terbaru:
   ```bash
   git pull origin staging
   ```

3. Buat branch fitur:
   ```bash
   git checkout -b feat/kp-nama-fitur
   ```

#### 2. Pengembangan Fitur

```bash
git add .
git commit -m "feat(kp): menambahkan fitur pengajuan sidang"
```

#### 3. Sinkronisasi Jika Staging Terbaru Berubah

1. Stash sementara:
   ```bash
   git stash
   ```

2. Update staging:
   ```bash
   git pull origin staging
   ```

3. Kembali ke fitur:
   ```bash
   git stash pop
   ```

#### 4. Push Branch Fitur

```bash
git push origin feat/kp-nama-fitur
```

#### 5. Merge Request

- Developer membuat MR ke staging
- Admin/Lead melakukan review dan merge
- Testing di staging environment

#### 6. Merge ke Production

- Admin merge staging → main/master
- Deployment dijalankan sesuai CI/CD
- Monitoring post-deployment

---

### 11.3 Keuntungan Workflow

- Menghindari konflik kode di production
- Memastikan fitur telah diuji sebelum rilis
- Pengembangan lebih terstruktur dan kolaboratif
- Memudahkan rollback jika terjadi masalah
- Isolasi fitur KP dari modul lain (TA, BK, Alumni)

---


_Architecture focused on simplicity, scalability, data security, and improvements from old system for multi-user self-hosted deployment with multi-sub-application support (Tugas Akhir, Kerja Praktik, Bimbingan Karir, Alumni). Specifically designed to address critical issues from the previous Kerja Praktik system including kuota management, multiple periode support, and automated UDI synchronization._