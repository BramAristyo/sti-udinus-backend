# Laravel Pengumuman Module - Best Practice Implementation

**Purpose:** Implementasi module Pengumuman sesuai Laravel Coding Standards

**Stack:** Laravel 11 | MySQL

---

## Code Patterns

### 1. Route Pattern
- Setiap route baru wajib diberi middleware permission menggunakan Spatie.
- Hanya role yang memiliki permission tersebut yang dapat mengakses route.
- Route dikelompokkan menggunakan prefix untuk menjaga struktur yang rapi.
- Setiap aksi CRUD tetap menggunakan HTTP method yang sesuai (GET, POST, PATCH, DELETE).

```php
Route::prefix('pengumuman')->middleware(['auth:sanctum'])->group(function () {
    Route::middleware('can:manage pengumuman')->group(function () {
        Route::get('/', [PengumumanController::class, 'index']);
        Route::post('/', [PengumumanController::class, 'store']);
        Route::get('/{pengumumanId}', [PengumumanController::class, 'show']);
        Route::patch('/{pengumumanId}', [PengumumanController::class, 'update']);
        Route::delete('/{pengumumanId}', [PengumumanController::class, 'destroy']);
    });
});
```

### 2. Controller Pattern
- Controller hanya berfungsi sebagai request handler.
- Tidak boleh menaruh logic bisnis di controller.
- Menggunakan Service Layer untuk seluruh logic.
- Response harus menggunakan Resource atau Collection.
- Kesalahan ditangani menggunakan try–catch dengan format error standar API.

```php
<?php

namespace App\Http\Controllers\API\Web\V1\Common;

use App\Http\Controllers\Controller;
use App\Http\Requests\API\V1\Web\Common\PengumumanRequest;
use App\Http\Resources\API\V1\Web\Common\PengumumanResource;
use App\Http\Resources\API\V1\Web\Common\PengumumanCollection;
use App\Services\API\V1\Web\Common\PengumumanService;
use Illuminate\Http\Request;

class PengumumanController extends Controller
{
    protected PengumumanService $pengumumanService;

    public function __construct(PengumumanService $pengumumanService)
    {
        $this->pengumumanService = $pengumumanService;
    }

    public function index(Request $request)
    {
        try {
            $announcements = $this->pengumumanService->getAllPengumuman($request->all());
            $result = new PengumumanCollection($announcements, "Sukses");
            return $this->respond($result);
        } catch (\Exception $e) {
            return $this->ApiExceptionError($e->getMessage(), $e->getCode() ?: 404);
        }
    }

    public function store(PengumumanRequest $request)
    {
        try {
            $announcement = $this->pengumumanService->createPengumuman($request);
            $result = new PengumumanResource($announcement, "Berhasil membuat pengumuman", 201);
            return $this->respond($result, 201);
        } catch (\Exception $e) {
            return $this->ApiExceptionError($e->getMessage(), $e->getCode() ?: 400);
        }
    }

    public function show($pengumumanId)
    {
        try {
            $announcement = $this->pengumumanService->getPengumumanById($pengumumanId);
            $result = new PengumumanResource($announcement, "Sukses");
            return $this->respond($result);
        } catch (\Exception $e) {
            return $this->ApiExceptionError($e->getMessage(), $e->getCode() ?: 404);
        }
    }

    public function update(PengumumanRequest $request, $pengumumanId)
    {
        try {
            $announcement = $this->pengumumanService->updatePengumuman($request, $pengumumanId);
            $result = new PengumumanResource($announcement, "Berhasil memperbarui pengumuman");
            return $this->respond($result);
        } catch (\Exception $e) {
            return $this->ApiExceptionError($e->getMessage(), $e->getCode() ?: 404);
        }
    }

    public function destroy($pengumumanId)
    {
        try {
            $this->pengumumanService->deletePengumuman($pengumumanId);
            return $this->respondMessage('Berhasil menghapus pengumuman', 200);
        } catch (\Exception $e) {
            return $this->ApiExceptionError($e->getMessage(), $e->getCode() ?: 404);
        }
    }
}
```

### 3. Request Pattern
- Semua input harus tervalidasi menggunakan Form Request.
- Setiap field wajib memiliki rule yang jelas.
- Validasi harus memisahkan logic dari controller untuk menjaga kebersihan kode

```php
<?php

namespace App\Http\Requests\API\V1\Web\Common;

use Illuminate\Foundation\Http\FormRequest;

class PengumumanRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'judul' => 'required|string|max:255',
            'user' => 'required|string|max:255',
            'isi' => 'required|string',
            'type' => 'required|string|in:ta,kp,bk,alumni'
        ];
    }

    public function messages(): array
    {
        return [
            'judul.required' => 'Judul pengumuman wajib diisi',
            'judul.max' => 'Judul pengumuman maksimal 255 karakter',
            'user.required' => 'User wajib diisi',
            'user.max' => 'User maksimal 255 karakter',
            'isi.required' => 'Isi pengumuman wajib diisi',
            'type.required' => 'Tipe pengumuman wajib diisi',
            'type.in' => 'Tipe pengumuman harus salah satu dari: ta, kp, bk, alumni',
        ];
    }
}
```

### 4. Resource Pattern
- Resource digunakan untuk single item, Collection untuk multiple items.
- Struktur JSON harus konsisten: data, lalu meta.
- Attribute tanggal harus dikonversi ke format string yang rapi (toDateTimeString())

```php
// Collection
<?php

namespace App\Http\Resources\API\V1\Web\Common;

use Illuminate\Http\Resources\Json\ResourceCollection;

class PengumumanCollection extends ResourceCollection
{
    public function toArray($request)
    {
        return [
            'data' => $this->collection->map(function ($pengumuman) {
                return [
                    'id' => $pengumuman->id,
                    'judul' => $pengumuman->judul,
                    'user' => $pengumuman->user,
                    'isi' => $pengumuman->isi,
                    'type' => $pengumuman->type,
                    'published_at' => $pengumuman->published_at?->toDateTimeString(),
                    'created_at' => $pengumuman->created_at->toDateTimeString(),
                    'updated_at' => $pengumuman->updated_at->toDateTimeString(),
                ];
            }),
            'meta' => [
                'status_code' => $this->code,
                'success' => true,
                'message' => $this->message,
                'pagination' => $this->metaData(),
            ],
        ];
    }

    private function metaData()
    {
        return [
            'total' => $this->total(),
            'count' => $this->count(),
            'per_page' => $this->perPage(),
            'current_page' => $this->currentPage(),
            'total_pages' => $this->lastPage(),
        ];
    }
}

// Resource
<?php

namespace App\Http\Resources\API\V1\Web\Common;

use Illuminate\Http\Resources\Json\JsonResource;

class PengumumanResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'data' => [
                'id' => $this->id,
                'judul' => $this->judul,
                'user' => $this->user,
                'isi' => $this->isi,
                'type' => $this->type,
                'published_at' => $this->published_at?->toDateTimeString(),
                'created_at' => $this->created_at->toDateTimeString(),
                'updated_at' => $this->updated_at->toDateTimeString(),
            ]
        ];
    }
}
```

### 5. Service Layer
- Semua logic bisnis ditempatkan di Service.
- Service harus menangani validasi pada level data (misal: checking exist).
- Service yang mengembalikan data harus clean dan siap dikirim ke Resource.
- Error handling dilakukan dengan melempar Exception agar controller menangkap.

```php
<?php

namespace App\Services\API\V1\Web\Common;

use App\Models\Pengumuman;
use Exception;

class PengumumanService
{
    public function getAllPengumuman(array $params)
    {
        $user = auth()->user();
        if (!$user) {
            throw new Exception("User tidak terautentikasi", 401);
        }

        if (empty($params['type'])) {
            throw new Exception('Parameter type wajib diisi', 400);
        }

        $query = Pengumuman::query();

        $query->where('type', $params['type']);

        if (!empty($params['search'])) {
            $query->where('judul', 'like', '%' . $params['search'] . '%');
        }

        $sortOrder = isset($params['sort_order']) ? strtolower($params['sort_order']) : 'desc';
        if (!in_array($sortOrder, ['asc', 'desc'])) {
            throw new Exception('Sort order harus asc atau desc', 400);
        }
        $query->orderBy('created_at', $sortOrder);

        $perPage = isset($params['limit']) ? (int)$params['limit'] : 10;
        $announcements = $query->paginate($perPage);

        return $announcements;
    }

    public function createPengumuman($request)
    {
        $announcement = Pengumuman::create([
            'judul' => $request->judul,
            'user' => $request->user,
            'isi' => $request->isi,
            'type' => $request->type,
            'published_at' => now(),
        ]);

        return $announcement;
    }

    public function getPengumumanById($pengumumanId)
    {
        $announcement = Pengumuman::find($pengumumanId);
        
        if (!$announcement) {
            throw new Exception("Pengumuman tidak ditemukan", 404);
        }

        return $announcement;
    }

    public function updatePengumuman($request, $pengumumanId)
    {
        $announcement = Pengumuman::find($pengumumanId);

        if (!$announcement) {
            throw new Exception("Pengumuman tidak ditemukan", 404);
        }

        $announcement->update([
            'judul' => $request->judul,
            'user' => $request->user,
            'isi' => $request->isi,
            'type' => $request->type,
        ]);

        return $announcement;
    }

    public function deletePengumuman($pengumumanId)
    {
        $announcement = Pengumuman::find($pengumumanId);

        if (!$announcement) {
            throw new Exception("Pengumuman tidak ditemukan", 404);
        }

        $announcement->delete();

        return $announcement;
    }
}
```
## Struktur Folder Modular

```
sti-api/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── API/
│   │   │       ├── Web/
│   │   │       │   └── V1/
│   │   │       │       ├── Auth/
│   │   │       │       ├── Common/
│   │   │       │       ├── KP/
│   │   │       │       │   ├── Koor/
│   │   │       │       │   ├── Dosen/
│   │   │       │       │   └── Mahasiswa/
│   │   │       │       ├── TA/
│   │   │       │       │   ├── Koor/
│   │   │       │       │   ├── Dosen/
│   │   │       │       │   └── Mahasiswa/
│   │   │       │       ├── BK/
│   │   │       │       │   ├── Dosen/
│   │   │       │       │   └── Mahasiswa/
│   │   │       │       └── Alumni/
│   │   │       │           ├── Alumnus/
│   │   │       │           ├── Koor/
│   │   │       │           ├── Mahasiswa/
│   │   │       │           └── Mitra/
│   │   │       └── Mobile/
│   │   │           └── V1/
│   │   │
│   │   ├── Requests/
│   │   │   ├── Auth/
│   │   │   ├── Common/
│   │   │   ├── KP/
│   │   │   ├── TA/
│   │   │   ├── BK/
│   │   │   └── Alumni/
│   │   │
│   │   └── Resources/
│   │       └── API/
│   │           ├── V1/
│   │           │   ├── Auth/
│   │           │   ├── Common/
│   │           │   ├── KP/
│   │           │   ├── TA/
│   │           │   ├── BK/
│   │           │   └── Alumni/
│   │           └── V2/
│   │
│   ├── Models/
│   │
│   └── Services/
│       ├── Auth/
│       ├── Common/
│       ├── KP/
│       ├── TA/
│       ├── BK/
│       └── Alumni/
│
├── database/
│   ├── migrations/
│   └── seeders/
│
├── routes/
│   └── api/
│       └── v1/
│
├── tests/
│   ├── Feature/
│   │   └── API/
│   │       └── V1/
│   │           ├── Auth/
│   │           ├── Common/
│   │           ├── KP/
│   │           ├── TA/
│   │           ├── BK/
│   │           └── Alumni/
│   └── Unit/
│       ├── Services/
│       │   └── KP/
│       │   └── TA/
│       │   └── BK/
│       │   └── Alumni/
│       └── Models/
│
└── storage/
    ├── logs/
    ├── uploads/
    │   ├── kp/
    │   ├── ta/
    │   ├── bk/
    │   └── alumni/
    └── temp/
```

---

## Resources

- **Laravel Docs:** https://laravel.com/docs/11.x/
- **Database Schema:** See [Database Schema](../development-guide-Database-Schema.md)
- **Architecture:** See [Technical Architecture](../technical-architectures.md)

---

**Maintained By:** Backend Development STI Team