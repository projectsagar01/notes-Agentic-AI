---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/roadmap/day-9-3-1/","dg-note-properties":{}}
---

**Day 10 — Async Ingestion Pipeline (PDFIngestor)**

---

## 🎯 Today's Goal

AI ko **documents (PDF, text files) padhna sikhana** using background jobs.

**Why:** Pehle humne manually 4 documents seed kiye the. Real world mein 500 PDFs upload honge — user ko wait nahi karvana. Background mein process karna hai.

---

## 🔥 Key Concepts You'll Learn

| Concept | What It Is |
| :--- | :--- |
| **Async Ingestion** | Document upload → Queue → Chunk → Embed → Store (all in background) |
| **Backpressure** | Rate limiting queue workers to prevent DB overwhelm |
| **Chunking** | PDF ko small pieces mein break karna (context window ke andar) |

---

## 🧠 Backpressure (Mental Model)

Soch: Ek restaurant mein 500 orders aa gaye. Ek cook hai toh sab wait karenge. 5 cooks hain toh fast hoga. Lekin agar 100 orders ek saath aayein aur cooks zyada slow hain, toh queue badhti jaayegi. **Backpressure** = cook ko kaam karne ki speed limit dena taaki kitchen overload na ho.

**Technical:** Queue worker ko rate limit karo — 5 jobs per second se zyada na chale.

---

## 🛠️ What to Build: `PDFIngestor`

**Project Structure:**

```
app/
├── Models/
│   ├── Document.php          # Existing
│   └── DocumentChunk.php     # NEW: Chunks store karega
├── Jobs/
│   └── ProcessDocument.php   # NEW: Background job
├── Http/
│   └── Controllers/
│       └── DocumentController.php  # NEW: Upload endpoint
└── Services/
    └── Chunker.php           # NEW: Chunking logic
```

---

## 📁 Step 1: Migration — DocumentChunks Table

```bash
php artisan make:migration create_document_chunks_table
```

```php
public function up()
{
    Schema::create('document_chunks', function (Blueprint $table) {
        $table->id();
        $table->foreignId('document_id')->constrained()->onDelete('cascade');
        $table->integer('chunk_index');
        $table->text('content');
        $table->vector('embedding', 768);
        $table->timestamps();
    });
}
```

---

## 📁 Step 2: DocumentChunk Model

```bash
php artisan make:model DocumentChunk
```

**File: `app/Models/DocumentChunk.php`**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class DocumentChunk extends Model
{
    protected $fillable = ['document_id', 'chunk_index', 'content', 'embedding'];

    protected function casts(): array
    {
        return ['embedding' => 'array'];
    }

    public function document()
    {
        return $this->belongsTo(Document::class);
    }
}
```

---

## 📁 Step 3: Document Model — Relationship

**File: `app/Models/Document.php`**

```php
public function chunks()
{
    return $this->hasMany(DocumentChunk::class);
}
```

---

## 📁 Step 4: Chunker Service

```bash
php artisan make:service Chunker
```

**File: `app/Services/Chunker.php`**

```php
<?php

namespace App\Services;

class Chunker
{
    public function chunk(string $text, int $size = 500): array
    {
        // 🔥 Split by sentences
        $sentences = preg_split('/(?<=[.!?])\s+/', $text, -1, PREG_SPLIT_NO_EMPTY);
        
        $chunks = [];
        $current = '';

        foreach ($sentences as $sentence) {
            if (strlen($current . $sentence) > $size) {
                if (!empty($current)) {
                    $chunks[] = trim($current);
                }
                $current = $sentence;
            } else {
                $current .= ' ' . $sentence;
            }
        }

        if (!empty($current)) {
            $chunks[] = trim($current);
        }

        return $chunks;
    }
}
```

---

## 📁 Step 5: Background Job

```bash
php artisan make:job ProcessDocument
```

**File: `app/Jobs/ProcessDocument.php`**

```php
<?php

namespace App\Jobs;

use App\Models\Document;
use App\Models\DocumentChunk;
use App\Services\Chunker;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class ProcessDocument implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, SerializesModels;

    public $tries = 3;
    public $backoff = [5, 10, 30];

    protected Document $document;

    public function __construct(Document $document)
    {
        $this->document = $document;
    }

    public function handle(Chunker $chunker): void
    {
        // 🔥 Step 1: Chunk
        $chunks = $chunker->chunk($this->document->content);

        foreach ($chunks as $index => $chunk) {
            // 🔥 Step 2: Embed each chunk
            $response = Http::post('http://localhost:11434/api/embeddings', [
                'model' => 'nomic-embed-text',
                'prompt' => $chunk,
            ]);

            $embedding = $response->json()['embedding'] ?? [];

            // 🔥 Step 3: Store chunk
            DocumentChunk::create([
                'document_id' => $this->document->id,
                'chunk_index' => $index,
                'content' => $chunk,
                'embedding' => $embedding,
            ]);
        }

        // 🔥 Step 4: Update document status
        $this->document->update(['status' => 'processed']);

        Log::info('✅ Document #' . $this->document->id . ' processed with ' . count($chunks) . ' chunks.');
    }
}
```

---

## 📁 Step 6: Document Controller

```bash
php artisan make:controller DocumentController
```

**File: `app/Http/Controllers/DocumentController.php`**

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessDocument;
use App\Models\Document;
use Illuminate\Http\Request;

class DocumentController extends Controller
{
    public function upload(Request $request)
    {
        $request->validate([
            'title' => 'required|string',
            'content' => 'required|string',
        ]);

        // 🔥 Create document (pending status)
        $document = Document::create([
            'title' => $request->title,
            'content' => $request->content,
            'status' => 'pending',
        ]);

        // 🔥 Dispatch job (async)
        ProcessDocument::dispatch($document);

        return response()->json([
            'message' => 'Document uploaded. Processing in background.',
            'document_id' => $document->id,
        ]);
    }

    public function status($id)
    {
        $document = Document::with('chunks')->findOrFail($id);

        return response()->json([
            'id' => $document->id,
            'title' => $document->title,
            'status' => $document->status ?? 'pending',
            'chunks_count' => $document->chunks->count(),
        ]);
    }

    public function search(Request $request)
    {
        $query = $request->input('q');

        $results = DocumentChunk::query()
            ->whereVectorSimilarTo('embedding', $query, minSimilarity: 0.3)
            ->limit(5)
            ->get(['content']);

        return response()->json($results);
    }
}
```

---

## 📁 Step 7: Routes

**`routes/web.php`**

```php
use App\Http\Controllers\DocumentController;
use App\Http\Controllers\SearchController;

Route::post('/documents/upload', [DocumentController::class, 'upload']);
Route::get('/documents/{id}/status', [DocumentController::class, 'status']);
Route::get('/search', [DocumentController::class, 'search']);
```

---

## 📁 Step 8: Migration Update — Add status

```bash
php artisan make:migration add_status_to_documents
```

```php
public function up()
{
    Schema::table('documents', function (Blueprint $table) {
        $table->string('status')->default('pending');
    });
}
```

---

## ✅ Step 9: Test

```bash
php artisan queue:work
```

**Upload a document:**

```bash
curl -X POST http://127.0.0.1:8000/documents/upload \
-H "Content-Type: application/json" \
-d '{
    "title": "Return Policy",
    "content": "You can return items within 30 days. Items must be unused. Refunds are processed within 5-7 business days."
}'
```

**Check status:**

```bash
curl http://127.0.0.1:8000/documents/1/status
```

**Search:**

```bash
curl "http://127.0.0.1:8000/search?q=How to return an item?"
```

---

## 🧠 Interview Prep (For Day 10)

**Q: *"How do you handle large document uploads?"***

**A:** *"I use async ingestion. The user uploads the file, I return an immediate response with a job ID. The document is chunked, embedded, and stored in the background. The user can check the status via an API endpoint. This prevents the HTTP request from timing out and provides a better user experience."*

**Q: *"What is backpressure?"** *

**A:** *"Backpressure is rate limiting the queue workers to prevent the database from being overwhelmed by a large number of concurrent jobs. I use Laravel's `throttle` middleware on the queue worker or set `--max-jobs` to control the number of jobs processed per minute."*

---

**Type `NEXT` when ready for Day 11 — RAG Pipeline.** 🐇🔥