---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/day-9/","dg-note-properties":{}}
---



Day 9 — pgvector & HNSW Indexing

---

## 🧠 What You'll Learn Today

| Concept | What It Means |
| :--- | :--- |
| **Embeddings** | Text ko numbers (vectors) mein convert karna taaki AI "meaning" samajh sake |
| **pgvector** | PostgreSQL ka extension jo vectors store aur search kar sakta hai |
| **HNSW Index** | Approximate search index — fast queries ke liye (recommended default) |
| **Index Tuning** | `m`, `ef_construction`, `ef_search` parameters — recall vs speed balance |

---

## 🔥 Why This Exists

**Problem:** AI ko private documents (PDFs, policies, manuals) padhna hai. Par LLM ko directly 100-page PDF nahi bhej sakte — token limit.

**Solution:** Documents ko **chunks** mein break karo, har chunk ko **embedding** (vector) mein convert karo, aur pgvector mein store karo. Jab user query aaye, uski embedding generate karo aur **semantic search** se relevant chunks dhundho.

**Without pgvector:** Har query ke liye sab documents scan karna padega (O(n) — slow).  
**With HNSW:** Approximate search (O(log n) — fast).

---

## 📖 Where to Learn From

| Resource | What to Read | Why |
| :--- | :--- | :--- |
| **[Laravel AI SDK — Vector Search](https://laravel.com/docs/12.x/ai-sdk#3)** | Vector columns, `whereVectorSimilarTo()` | Official Laravel docs — best reference |
| **[Laravel News — RAG with pgvector](https://laravel-news.com/ship-ai-with-laravel-rag-with-embeddings-and-pgvector-in-laravel-13)** | Full tutorial with code | Practical step-by-step guide |
| **[pgvector GitHub](https://github.com/pgvector/pgvector)** | Installation, HNSW, IVFFlat | Official extension docs |
| **[Supabase — Going to Production](https://supabase.com/docs/guides/ai/going-to-prod)** | Index tuning (`m`, `ef_construction`, `ef_search`) | Production-grade tuning guide |

---

## 💻 Examples (Try These First)

### 1. Migration — Vector Column

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        // 🔥 Step 1: Enable pgvector extension
        Schema::ensureVectorExtensionExists();

        Schema::create('documents', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('content');
            
            // 🔥 Step 2: Add vector column (1536 dimensions for Gemini/OpenAI)
            $table->vector('embedding', dimensions: 1536)->index(); // HNSW index automatically
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('documents');
    }
};
```

### 2. Model — Cast Vector to Array

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Document extends Model
{
    protected $fillable = ['title', 'content', 'embedding'];

    protected function casts(): array
    {
        return [
            'embedding' => 'array', // 🔥 Vector ko array mein convert karo
        ];
    }
}
```

### 3. Generate & Store Embedding

```php
use App\Models\Document;
use Laravel\Ai\Facades\Embeddings;

$embedding = Embeddings::for(['Your document text here'])
    ->provider('gemini')   // or 'ollama'
    ->model('gemini-embedding-001')
    ->generate();

Document::create([
    'title' => 'Return Policy',
    'content' => 'You can return items within 30 days...',
    'embedding' => $embedding->first(), // 🔥 Store vector
]);
```

### 4. Semantic Search — Find Similar Documents

```php
use App\Models\Document;

$query = "How do I return a damaged item?";

// 🔥 Laravel automatically generates embedding for the query
$results = Document::query()
    ->whereVectorSimilarTo('embedding', $query, minSimilarity: 0.4) // 🔥 0.4 = similarity threshold
    ->limit(5)
    ->get();

foreach ($results as $doc) {
    echo $doc->title . ' — ' . $doc->content . "\n";
}
```

---

## 🛠️ Small Project: `VectorStore`

**Goal:** Documents ka vector store banao aur semantic search implement karo.

### Files to Create

```
app/
├── Models/
│   └── Document.php
├── Console/
│   └── Commands/
│       └── SeedDocuments.php      # Dummy documents + embeddings
└── Http/
    └── Controllers/
        └── SearchController.php   # Search endpoint

database/
└── migrations/
    └── create_documents_table.php
```

### Step 1: Migration (as above)

### Step 2: Model (as above)

### Step 3: Seeder Command

```bash
php artisan make:command SeedDocuments
```

```php
<?php

namespace App\Console\Commands;

use App\Models\Document;
use Illuminate\Console\Command;
use Laravel\Ai\Facades\Embeddings;

class SeedDocuments extends Command
{
    protected $signature = 'documents:seed';
    protected $description = 'Seed documents with embeddings';

    public function handle(): void
    {
        $faqs = [
            ['title' => 'Return Policy', 'content' => 'You can return items within 30 days. Items must be unused.'],
            ['title' => 'Shipping Policy', 'content' => 'Free shipping on orders over $50. Delivery in 3-5 business days.'],
            ['title' => 'Warranty', 'content' => 'All products have a 1-year warranty against manufacturing defects.'],
            ['title' => 'Refund Policy', 'content' => 'Refunds are processed within 5-7 business days after return approval.'],
        ];

        foreach ($faqs as $faq) {
            $embedding = Embeddings::for([$faq['content']])->generate();
            
            Document::create([
                'title' => $faq['title'],
                'content' => $faq['content'],
                'embedding' => $embedding->first(),
            ]);
        }

        $this->info('✅ 4 documents seeded with embeddings!');
    }
}
```

Run it:

```bash
php artisan documents:seed
```

### Step 4: Search Controller

```bash
php artisan make:controller SearchController
```

```php
<?php

namespace App\Http\Controllers;

use App\Models\Document;
use Illuminate\Http\Request;

class SearchController extends Controller
{
    public function search(Request $request)
    {
        $query = $request->input('q');

        $results = Document::query()
            ->whereVectorSimilarTo('embedding', $query, minSimilarity: 0.3)
            ->limit(5)
            ->get(['title', 'content']);

        return response()->json($results);
    }
}
```

### Step 5: Route

```php
use App\Http\Controllers\SearchController;

Route::get('/search', [SearchController::class, 'search']);
```

### Step 6: Test

```bash
curl "http://127.0.0.1:8000/search?q=How to return a damaged item?"
```

**Expected Output:** Return Policy wala document aana chahiye — bina exact keywords ke.

---

## 🧠 HNSW Index Tuning (Interview Prep)

| Parameter | What It Does | Default | When to Change |
| :--- | :--- | :--- | :--- |
| **`m`** | Bi-directional links per element | 16 | Higher = better accuracy, more memory |
| **`ef_construction`** | Build-time candidate list | 64 | Higher = better quality, slower build |
| **`ef_search`** | Query-time candidate list | 40 | Higher = better recall, slower query |

**Interview Pitch:**

> *"I use HNSW indexes for vector search because they provide better query speed and recall compared to IVFFlat. For production, I set `m=16`, `ef_construction=64`, and tune `ef_search` based on my recall requirements. Increasing `ef_search` improves recall but increases latency."*

---

## ✅ Day 9 Checklist

- [ ] pgvector extension enabled in PostgreSQL
- [ ] Migration with vector column + HNSW index
- [ ] Model with `embedding` cast to array
- [ ] Documents seeded with embeddings
- [ ] Semantic search endpoint working
- [ ] HNSW parameters understood (`m`, `ef_construction`, `ef_search`)

---

**Type `NEXT` when you're ready to build.** 🐇🔥
