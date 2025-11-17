# ❗ Error Handling & Logging

Dokumentasi ini menjelaskan standar **Penanganan Error** dan **Logging**, yang digunakan pada project **STI - API**.  
Seluruh controller disarankan untuk memakai **`App\Http\Controllers\Controller`** agar konsisten dalam response dan logging.

---

## Base Controller — Error & Success Response

**File:** **`app/Http/Controllers/Controller.php`**

```php
<?php

namespace App\Http\Controllers;

use App\Models\ErrorLog;
use OpenApi\Annotations as OA;

abstract class Controller
{
    public function exceptionError($e, $exception, $status = 400)
    {
        $request = request();

        ErrorLog::create([
            'user_id' => optional($request->user())->id,
            'method'  => $request->method(),
            'url'     => $request->fullUrl(),
            'message' => $e->getMessage(),
            'trace'   => $e->getTraceAsString(),
            'payload' => json_encode($request->all()),
        ]);

        return response()->json([
            'success' => false,
            'message' => $e->getMessage(),
            'errors'  => 'Exception Error : ' . $exception,
        ], $status);
    }

    public function successResponse($data, $message = 'Success', $status = 200)
    {
        return response()->json([
            'success' => true,
            'message' => $message,
            'data'    => $data
        ], $status);
    }

    public function respond($data)
    {
        return response()->json($data);
    }
}
```

---

## Struktur Standar Response API

### ✅ Response Berhasil

```json
{
    "success": true,
    "message": "Program unggulan berhasil ditambahkan",
    "data": {
        "id": 1,
        "nama": "Program A"
    }
}
```

### ❌ Response Error

```json
{
    "success": false,
    "message": "SQLSTATE[23000]: Integrity constraint violation: ...",
    "errors": "Exception Error : Gagal menambahkan program unggulan"
}
```

---

## Penggunaan di Controller

**Contoh di method `store()`**

```php
public function store(ProgramUnggulanRequest $request): JsonResponse
{
    try {
        $data = $this->service->create($request->validated());

        return $this->successResponse(
            $data,
            'Program unggulan berhasil ditambahkan',
            201
        );
    } catch (\Exception $e) {
        Log::error($e->getMessage());
        return $this->exceptionError($e, 'Gagal menambahkan program unggulan');
    }
}
```

---

## Error Logging ke Database (`error_logs`)

Setiap terjadi error, sistem mencatat log ke dalam database.

Contoh data yang disimpan:

| Field   | Deskripsi                         |
| ------- | --------------------------------- |
| user_id | ID user yang sedang login         |
| method  | HTTP method (GET/POST/PUT/DELETE) |
| url     | URL lengkap request               |
| message | Pesan error dari exception        |
| trace   | Detail stack trace                |
| payload | Body request (JSON)               |

---

## System Log (`Log::error()`)

```php
Log::error($e->getMessage());
```

Log disimpan pada:

```
storage/logs/laravel.log
```

---

## Contoh Response Postman

### ✅ Berhasil (200/201)

```json
{
    "success": true,
    "message": "Program unggulan berhasil ditambahkan",
    "data": {
        "id": 10,
        "nama": "Program Baru"
    }
}
```

### ❌ Gagal (400/500 Error)

```json
{
    "success": false,
    "message": "SQLSTATE[23000]: Integrity constraint violation...",
    "errors": "Exception Error : Gagal menambahkan program unggulan"
}
```

---

## Debugging Tools

| Tool     | Deskripsi                                        |
| -------- | ------------------------------------------------ |
| `dd()`   | Menampilkan isi variabel + menghentikan eksekusi |
| `dump()` | Menampilkan isi variabel tanpa menghentikan      |

---

## Rangkuman

-   Gunakan `successResponse()` untuk response sukses standar.
-   Gunakan `exceptionError()` untuk logging + response error.
-   Gunakan `Log::error()` untuk debugging via file log.
-   Sangat disarankan menggunakan `try/catch` pada setiap request.
