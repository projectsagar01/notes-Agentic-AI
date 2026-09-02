---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/day-11/","dg-note-properties":{}}
---

**Bhai, Day 11 — Hybrid Search (SmartSearch). Let's go.**

---

## 🎯 Day 11: Hybrid Search — Combining Vector + Full-Text

### 📚 What You'll Learn Today

| Concept | What It Means |
| :--- | :--- |
| **Hybrid Search** | Combining pgvector (semantic) + PostgreSQL Full-Text Search (exact keyword) |
| **Why Hybrid?** | Pure vector search fails on serial numbers, exact names, or misspellings |
| **RRF (Reciprocal Rank Fusion)** | Merging results from multiple search methods |
| **PostgreSQL Full-Text** | `tsvector` + `tsquery` for exact keyword matching |

---

## 🧠 Why Hybrid Search Matters

| Scenario | Pure Vector Search | Hybrid Search |
| :--- | :--- | :--- |
| "Return policy" | ✅ Works | ✅ Works |
| **"Order #12345"** | ❌ Fails (serial number) | ✅ Works (exact match) |
| "Refund process" | ✅ Works | ✅ Works |
| **"Warranty 2024"** | ⚠️ Partial | ✅ Works (year + keyword) |
| "How to return?" | ✅ Works | ✅ Works |

**The Problem:** Pure vector search is great for *meaning*, but terrible for *exact matches*. If a user asks about "Order #12345", the vector search will look for semantically similar text, not the exact serial number.

**The Solution:** Combine vector search (semantic) with full-text search (exact keyword).

---

## 🏗️ What We'll Build: `SmartSearch`

| Feature | Implementation |
| :--- | :--- |
| **Document Chunks** | Existing `document_chunks` table |
| **Full-Text Column** | Add `tsvector` column for keyword search |
| **Search Endpoint** | `/smart-search?q=...` |
| **Hybrid Scoring** | Combine vector similarity + full-text relevance |
| **RRF (Reciprocal Rank Fusion)** | Merge both result sets into one ranked list |

---

## 📁 Step 1: Add Full-Text Column

```bash
./vendor/bin/sail artisan make:migration add_fulltext_to_document_chunks
```

**Migration: `database/migrations/...add_fulltext_to_document_chunks.php`**

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('document_chunks', function (Blueprint $table) {
            // 🔥 Add tsvector column for full-text search
            $table->tsvector('search_vector')->nullable();
        });

        // 🔥 Create GIN index for fast full-text search
        DB::statement('CREATE INDEX idx_chunks_search_vector ON document_chunks USING GIN (search_vector);');
    }

    public function down(): void
    {
        Schema::table('document_chunks', function (Blueprint $table) {
            $table->dropColumn('search_vector');
        });
    }
};
```

```bash
./vendor/bin/sail artisan migrate
```

---

## 📁 Step 2: Update Model to Generate `search_vector`

**`app/Models/DocumentChunk.php`**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class DocumentChunk extends Model
{
    protected $fillable = [
        'document_id',
        'chunk_index',
        'content',
        'embedding',
        'search_vector',
    ];

    protected function casts(): array
    {
        return [
            'embedding' => 'array',
        ];
    }

    /**
     * 🔥 Boot method to auto-generate search_vector on save
     */
    protected static function booted()
    {
        static::saving(function ($chunk) {
            // 🔥 Convert content to tsvector for full-text search
            $chunk->search_vector = DB::raw(
                "to_tsvector('english', " . DB::connection()->getPdo()->quote($chunk->content) . ")"
            );
        });
    }
}
```

**⚠️ Issue:** The `booted` method with `DB::raw` might not work in all environments. Let's use a simpler approach: update the seeder.

---

## 📁 Step 3: Update Seeder with `search_vector`

**`app/Console/Commands/SeedDocumentsCommand.php`**

```php
<?php

namespace App\Console\Commands;

use App\Models\Document;
use App\Models\DocumentChunk;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;

class SeedDocumentsCommand extends Command
{
    protected $signature = 'documents:seed';
    protected $description = 'Seed documents with embeddings and full-text search vectors';

    public function handle(): void
    {
        $faqs = [
            ['title' => 'Return Policy', 'content' => 'You can return items within 30 days. Items must be unused.'],
            ['title' => 'Shipping Policy', 'content' => 'Free shipping on orders over $50. Delivery in 3-5 business days.'],
            ['title' => 'Warranty', 'content' => 'All products have a 1-year warranty against manufacturing defects.'],
            ['title' => 'Refund Policy', 'content' => 'Refunds are processed within 5-7 business days after return approval.'],
        ];

        $ollamaUrl = env('OLLAMA_URL', 'http://host.docker.internal:11434');

        foreach ($faqs as $faq) {
            $this->info('🔮 Embedding: ' . $faq['title']);

            // 🔥 Generate embedding
            $response = Http::post($ollamaUrl . '/api/embed', [
                'model' => 'mxbai-embed-large:latest',
                'input' => $faq['content'],
            ]);

            $embedding = $response->json()['embeddings'][0] ?? [];

            if (empty($embedding)) {
                $this->error('❌ Failed to generate embedding for: ' . $faq['title']);
                continue;
            }

            // 🔥 Save document and chunk with search_vector
            $doc = Document::create([
                'title' => $faq['title'],
                'content' => $faq['content'],
                'status' => 'processed',
            ]);

            $chunk = DocumentChunk::create([
                'document_id' => $doc->id,
                'chunk_index' => 0,
                'content' => $faq['content'],
                'embedding' => $embedding,
            ]);

            // 🔥 Manually update search_vector for full-text search
            DB::table('document_chunks')
                ->where('id', $chunk->id)
                ->update([
                    'search_vector' => DB::raw("to_tsvector('english', " . DB::connection()->getPdo()->quote($faq['content']) . ")")
                ]);

            $this->info('✅ Embedded: ' . $faq['title'] . ' (Vector: ' . count($embedding) . ' dims)');
        }

        $this->info('✅ All documents seeded with embeddings and full-text search vectors!');
    }
}
```

---

## 📁 Step 4: Run Seeder Again

```bash
./vendor/bin/sail artisan documents:seed
```

---

## 📁 Step 5: Hybrid Search Controller

```bash
./vendor/bin/sail artisan make:controller SmartSearchController
```

**`app/Http/Controllers/SmartSearchController.php`**

```php
<?php

namespace App\Http\Controllers;

use App\Models\DocumentChunk;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;

class SmartSearchController extends Controller
{
    public function search(Request $request)
    {
        $query = $request->input('q');

        if (empty($query)) {
            return response()->json(['error' => 'Missing query parameter.'], 400);
        }

        $ollamaUrl = env('OLLAMA_URL', 'http://host.docker.internal:11434');

        // 🔥 Step 1: Generate Query Embedding
        $embeddingResponse = Http::post($ollamaUrl . '/api/embed', [
            'model' => 'mxbai-embed-large:latest',
            'input' => $query,
        ]);

        $queryEmbedding = $embeddingResponse->json()['embeddings'][0] ?? [];

        if (empty($queryEmbedding)) {
            return response()->json(['error' => 'Failed to generate query embedding.'], 500);
        }

        // 🔥 Step 2: Hybrid Search (Vector + Full-Text)
        $results = DocumentChunk::query()
            ->whereNotNull('embedding')
            ->whereNotNull('search_vector')
            ->select([
                'document_chunks.*',
                // 🔥 Vector similarity score
                DB::raw('1 - (embedding <=> ?) as vector_score', [$queryEmbedding]),
                // 🔥 Full-text search rank
                DB::raw("ts_rank(search_vector, plainto_tsquery('english', ?)) as text_score", [$query]),
            ])
            ->where(function ($queryBuilder) use ($query, $queryEmbedding) {
                // 🔥 Match either vector OR full-text
                $queryBuilder->whereRaw('embedding <=> ? < 0.5', [$queryEmbedding])
                    ->orWhereRaw("search_vector @@ plainto_tsquery('english', ?)", [$query]);
            })
            ->orderByRaw('(vector_score + text_score) DESC')
            ->limit(5)
            ->get(['id', 'content']);

        if ($results->isEmpty()) {
            return response()->json([
                'answer' => "I don't know. No relevant documents found.",
                'results' => []
            ]);
        }

        // 🔥 Step 3: Generate Answer (like RAG)
        $context = $results->pluck('content')->implode("\n\n---\n\n");
        $citationIds = $results->pluck('id')->toArray();

        $prompt = "You are a helpful assistant. Answer based ONLY on the provided context.\n\n";
        $prompt .= "Context:\n\"\"\"\n{$context}\n\"\"\"\n\n";
        $prompt .= "Question: {$query}\n\n";
        $prompt .= "If the answer is not in the context, say \"I don't know.\"";

        $response = Http::post($ollamaUrl . '/api/generate', [
            'model' => 'llama3.1',
            'prompt' => $prompt,
            'stream' => false,
        ]);

        $answer = $response->json()['response'] ?? 'No response from AI.';

        return response()->json([
            'answer' => $answer,
            'citations' => $citationIds,
            'chunks' => $results->pluck('content'),
        ]);
    }
}
```

---

## 📁 Step 6: Route

**`routes/web.php`**

```php
use App\Http\Controllers\SmartSearchController;

Route::get('/smart-search', [SmartSearchController::class, 'search']);
```

---

## 🧪 Step 7: Test

```bash
./vendor/bin/sail artisan config:clear
./vendor/bin/sail artisan serve
```

**Test 1 — Semantic Search:**
```bash
curl "http://localhost/smart-search?q=How%20to%20return%20an%20item%3F"
```

**Test 2 — Exact Match:**
```bash
curl "http://localhost/smart-search?q=30%20days"
```

**Test 3 — Serial Number (Vector fails, Full-Text works):**
```bash
curl "http://localhost/smart-search?q=Order%20%2312345"
```

---

## ✅ Day 11 Checklist

- [ ] Migration for `search_vector` column added
- [ ] Seeder updated to generate `search_vector`
- [ ] SmartSearchController created
- [ ] Hybrid search (vector + full-text) working
- [ ] Tests passed

---

## 🧠 Interview Pitch

> *"I built a hybrid search system that combines pgvector semantic search with PostgreSQL full-text search. When a user searches for a serial number like Order #12345, vector search fails because it looks for meaning, not exact text. My hybrid approach uses full-text search for exact matches and vector search for semantic meaning, then ranks results using a combination of both scores. This gives the best of both worlds."*

---

**Bata, ab hybrid search kaam kar raha hai? 🐇🔥**