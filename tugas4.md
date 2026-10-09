# Tugas 4 — Laravel Book API dan Pengujian Postman

## 1. Tujuan
Membuat API buku menggunakan Laravel dan menguji endpoint menggunakan Postman.

## 2. Resource Book
Tabel `books` memiliki kolom `id`, `title`, `isbn`, `available`, `created_at`, dan `updated_at`.

## 3. Script yang Digunakan

### 3.1 Migration — `database/migrations/...create_books_table.php`

```php
Schema::create('books', function (Blueprint $table) {
    $table->id();
    $table->string('title', 200);
    $table->string('isbn', 13)->unique();
    $table->boolean('available')->default(true);
    $table->timestamps();
});
```

### 3.2 Model — `app/Models/Book.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    protected $fillable = [
        'title',
        'isbn',
        'available',
    ];
}
```

### 3.3 Seeder — `database/seeders/BookSeeder.php`

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Book;

class BookSeeder extends Seeder
{
    public function run(): void
    {
        Book::create([
            'title' => 'Clean Code',
            'isbn' => '9780132350884',
            'available' => true,
        ]);

        Book::create([
            'title' => 'The Pragmatic Programmer',
            'isbn' => '9780135957059',
            'available' => true,
        ]);
    }
}
```

### 3.4 Route — `routes/api.php`

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\BookController;

Route::apiResource('books', BookController::class);
```

### 3.5 Controller — `app/Http/Controllers/BookController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Models\Book;
use Illuminate\Http\Request;

class BookController extends Controller
{
    public function index()
    {
        return response()->json(Book::all());
    }

    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required',
            'isbn' => 'required|unique:books,isbn',
            'available' => 'boolean',
        ]);

        $book = Book::create([
            'title' => $request->title,
            'isbn' => $request->isbn,
            'available' => $request->available ?? true,
        ]);

        return response()->json($book, 201);
    }

    public function show(Book $book)
    {
        return response()->json($book);
    }

    public function update(Request $request, Book $book)
    {
        $request->validate([
            'title' => 'required',
            'isbn' => 'required|unique:books,isbn,' . $book->id,
            'available' => 'boolean',
        ]);

        $book->update([
            'title' => $request->title,
            'isbn' => $request->isbn,
            'available' => $request->available ?? $book->available,
        ]);

        return response()->json($book);
    }

    public function destroy(Book $book)
    {
        $book->delete();

        return response()->json([
            'message' => 'Book deleted successfully'
        ]);
    }
}
```

## 4. Menjalankan API

Jalankan perintah ini dari terminal pada folder proyek Laravel. Biarkan terminal tetap berjalan selama pengujian.

```powershell
php artisan serve
```

URL dasar API: `http://127.0.0.1:8000`

## 5. Pengujian API di Postman

### 5.1 Daftar Buku — `GET /api/books`

**URL:**

```http
GET http://127.0.0.1:8000/api/books
```

**Hasil:** `200 OK`

**Response JSON:**

```json
[
    {
        "id": 1,
        "title": "Clean Code",
        "isbn": "9780132350884",
        "available": 1,
        "created_at": "2026-10-08T07:05:47.000000Z",
        "updated_at": "2026-10-08T07:05:47.000000Z"
    },
    {
        "id": 2,
        "title": "The Pragmatic Programmer",
        "isbn": "9780135957059",
        "available": 1,
        "created_at": "2026-10-08T07:05:47.000000Z",
        "updated_at": "2026-10-08T07:05:47.000000Z"
    }
]
```

**Penjelasan:** Endpoint berhasil mengembalikan dua buku yang tersimpan di database.

### 5.2 Detail Buku — `GET /api/books/{id}`

**URL untuk menguji buku yang tersedia:**

```http
GET http://127.0.0.1:8000/api/books/1
```

**Hasil yang diharapkan:** `200 OK`, dengan satu objek JSON:

```json
{
    "id": 1,
    "title": "Clean Code",
    "isbn": "9780132350884",
    "available": 1,
    "created_at": "2026-10-08T07:05:47.000000Z",
    "updated_at": "2026-10-08T07:05:47.000000Z"
}
```

**Catatan:** Tangkapan layar pengujian menggunakan `/api/books/15` dan menghasilkan `404 Not Found` karena ID 15 tidak tersedia. Untuk pengujian detail berhasil, gunakan ID `1` atau `2`.

### 5.3 Buku Tidak Ditemukan — `GET /api/books/999999`

**URL:**

```http
GET http://127.0.0.1:8000/api/books/999999
```

**Hasil:** `404 Not Found`

Pada konfigurasi debug Laravel, contoh pesan yang muncul adalah:

```json
{
    "message": "No query results for model [App\\Models\\Book] 999999"
}
```

Detail properti error dapat berbeda bergantung pada konfigurasi Laravel.

## 6. Ringkasan Hasil Pengujian

| Pengujian | Method | Endpoint | Hasil |
|---|---|---|---|
| Daftar buku | GET | `/api/books` | `200 OK` |
| Detail buku tersedia | GET | `/api/books/1` | Seharusnya `200 OK` |
| Detail ID tidak tersedia | GET | `/api/books/15` | `404 Not Found` pada pengujian |
| Buku tidak ditemukan | GET | `/api/books/999999` | `404 Not Found` |

## 7. Kesimpulan

API berhasil dijalankan dan endpoint daftar buku mengembalikan dua data dengan status `200 OK`. Permintaan terhadap ID yang tidak tersedia menghasilkan `404 Not Found`. Untuk melengkapi bukti detail buku berhasil, jalankan `GET http://127.0.0.1:8000/api/books/1` atau `/api/books/2` di Postman.

## 8. Bukti Pengujian

Tambahkan tangkapan layar Postman berikut ke bagian ini:

1. **Daftar Buku — 200 OK:** respons `GET /api/books`.
2. **Detail Buku — 200 OK:** respons `GET /api/books/1` atau `/api/books/2`.
3. **Buku Tidak Ditemukan — 404 Not Found:** respons `GET /api/books/999999`.
