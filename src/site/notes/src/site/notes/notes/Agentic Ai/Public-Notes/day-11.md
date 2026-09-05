---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/day-11/","dg-note-properties":{}}
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
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Bhai, Day 11 — Hybrid Search. Zero se Hero tak. No copy-paste. Samajh ke build karenge.**

---

## 🧠 What You'll Learn Today (The "Why")

| Concept | What It Is | Why You Need It |
| :--- | :--- | :--- |
| **Pure Vector Search** | Semantic search (meaning-based) | Finds similar concepts, but fails on exact matches |
| **Full-Text Search** | Keyword search (exact word) | Finds exact words, but fails on synonyms |
| **Hybrid Search** | Vector + Full-Text combined | **The best of both worlds** |
| **RRF (Reciprocal Rank Fusion)** | Merging two result sets | Fairly combines scores from both searches |

---

## 🔥 The Problem — Pure Vector Search Fails Here

| User Query | Pure Vector Search | Why It Fails |
| :--- | :--- | :--- |
| "How to return an item?" | ✅ Works | Finds similar meaning |
| **"Order #12345"** | ❌ Fails | Serial number has no meaning |
| **"Warranty 2024"** | ⚠️ Partial | "2024" is a number, not a concept |
| **"Sagar's policy"** | ❌ Fails | Proper name, no semantic meaning |

**Solution:** When vector search fails, full-text search catches exact matches.

---

## 🧠 The Mental Model (Samajh lo, fir kabhi nahi bhoologe)

### Analogy — The Library

| Search Type | Analogy |
| :--- | :--- |
| **Vector Search** | You describe the book's *idea* to the librarian. "I need a book about returning things." |
| **Full-Text Search** | You give the librarian the *exact title*. "I need the book called 'Return Policy'." |
| **Hybrid Search** | The librarian searches both ways and gives you the best results from both. |

### Real-World Example

| User Says | Vector Search Finds | Full-Text Finds | Hybrid Finds |
| :--- | :--- | :--- |
| "How to return?" | "Return Policy", "Refund Policy" | "Return Policy" | Both |
| **"Order #12345"** | Nothing (no meaning) | **"Order #12345"** | **"Order #12345"** ✅ |
| "Warranty issues" | "Warranty", "defects" | "Warranty" | Both |

---

## 🏗️ The Architecture (Kaise Kaam Karega)

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                         HYBRID SEARCH PIPELINE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  User Query: "Order #12345 warranty"                                       │
│       ↓                                                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Step 1: Generate Query Embedding (Vector)                         │   │
│  │  POST /api/embed → mxbai-embed-large → 1024-dim vector            │   │
│  └──────────┬──────────────────────────────────────────────────────────┘   │
│             ↓                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Step 2: Vector Search (pgvector)                                  │   │
│  │  WHERE embedding <=> query_embedding < 0.5                        │   │
│  │  → Returns semantically similar chunks                           │   │
│  │  → score: 0.85 (Return Policy)                                   │   │
│  └──────────┬──────────────────────────────────────────────────────────┘   │
│             ↓                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Step 3: Full-Text Search (PostgreSQL tsvector)                   │   │
│  │  WHERE search_vector @@ plainto_tsquery('Order #12345 warranty')  │   │
│  │  → Returns exact keyword matches                                 │   │
│  │  → score: 0.99 (Order #12345 record)                             │   │
│  └──────────┬──────────────────────────────────────────────────────────┘   │
│             ↓                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Step 4: Hybrid Ranking (Reciprocal Rank Fusion)                  │   │
│  │  Combined Score = (1 / rank_vector + 1 / rank_fulltext)          │   │
│  │  → Top result: "Order #12345" (exact match from full-text)       │   │
│  └──────────┬──────────────────────────────────────────────────────────┘   │
│             ↓                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  Step 5: LLM Generation (llama3.1)                                │   │
│  │  Context = Top 3 chunks from hybrid search                        │   │
│  │  Answer = "Order #12345 has a 1-year warranty..."                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ The 3 Core Technologies

### 1. PostgreSQL Full-Text Search (tsvector)

**What It Does:** Converts text into a searchable format (`tsvector`) and matches keywords.

| Component | Purpose |
| :--- | :--- |
| `to_tsvector('english', text)` | Converts text to searchable tokens |
| `plainto_tsquery('english', 'query')` | Converts user query to search format |
| `@@` operator | Checks if tsvector matches tsquery |
| `ts_rank()` | Calculates relevance score |

**Example:**
```sql
SELECT content, ts_rank(search_vector, plainto_tsquery('english', 'return policy')) AS score
FROM document_chunks
WHERE search_vector @@ plainto_tsquery('english', 'return policy')
ORDER BY score DESC;
```

**Why It Matters:** Full-text search handles exact keywords, proper nouns, and serial numbers that vector search misses.

---

### 2. pgvector (Semantic Search)

**What It Does:** Converts text to embeddings (vectors) and finds similar meanings.

| Component | Purpose |
| :--- | :--- |
| `embedding <=> query_embedding` | Cosine similarity (lower = more similar) |
| `ORDER BY embedding <=> query_embedding` | Sorts by similarity |

**Example:**
```sql
SELECT content, 1 - (embedding <=> query_embedding) AS score
FROM document_chunks
WHERE embedding <=> query_embedding < 0.5
ORDER BY score DESC;
```

---

### 3. Reciprocal Rank Fusion (RRF) — The Secret Sauce

**Problem:** Vector and full-text return different scores on different scales. You can't just add them.

**Solution:** RRF combines ranks instead of scores.

**Formula:**
```
RRF Score = Σ (1 / (rank + 60))
```

**Why 60?** Constant that prevents outliers from dominating.

**Example:**

| Chunk | Vector Rank | Full-Text Rank | RRF Score |
| :--- | :--- | :--- | :--- |
| A | 1 | 3 | 1/61 + 1/63 = 0.032 |
| B | 4 | 1 | 1/64 + 1/61 = 0.032 |
| C | 2 | 2 | 1/62 + 1/62 = 0.032 |

**Simpler Approach (we'll use):** `(vector_score + text_score) / 2`

---

## 💻 Code Walkthrough — Step by Step

### Step 1: Add Full-Text Column

```bash
./vendor/bin/sail artisan make:migration add_fulltext_to_document_chunks
```

**Migration:**

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
            // 🔥 Column for full-text search
            $table->tsvector('search_vector')->nullable();
        });

        // 🔥 GIN index for fast full-text search
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

### Step 2: Update Seeder

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

            // 🔥 Step 1: Generate embedding
            $response = Http::post($ollamaUrl . '/api/embed', [
                'model' => 'mxbai-embed-large:latest',
                'input' => $faq['content'],
            ]);

            $embedding = $response->json()['embeddings'][0] ?? [];

            if (empty($embedding)) {
                $this->error('❌ Failed to generate embedding for: ' . $faq['title']);
                continue;
            }

            // 🔥 Step 2: Save document and chunk
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

            // 🔥 Step 3: Update search_vector for full-text
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

```bash
./vendor/bin/sail artisan documents:seed
```

---

### Step 3: Hybrid Search Controller

```bash
./vendor/bin/sail artisan make:controller HybridSearchController
```

**`app/Http/Controllers/HybridSearchController.php`**

```php
<?php

namespace App\Http\Controllers;

use App\Models\DocumentChunk;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Http;

class HybridSearchController extends Controller
{
    public function search(Request $request)
    {
        $query = $request->input('q');

        if (empty($query)) {
            return response()->json(['error' => 'Missing query parameter.'], 400);
        }

        $ollamaUrl = env('OLLAMA_URL', 'http://host.docker.internal:11434');

        // 🔥 Step 1: Generate query embedding
        $embeddingResponse = Http::post($ollamaUrl . '/api/embed', [
            'model' => 'mxbai-embed-large:latest',
            'input' => $query,
        ]);

        $queryEmbedding = $embeddingResponse->json()['embeddings'][0] ?? [];

        if (empty($queryEmbedding)) {
            return response()->json(['error' => 'Failed to generate query embedding.'], 500);
        }

        // 🔥 Step 2: Hybrid Search
        $results = DocumentChunk::query()
            ->whereNotNull('embedding')
            ->whereNotNull('search_vector')
            ->select([
                'document_chunks.*',
                // 🔥 Vector similarity score (higher = better)
                DB::raw('1 - (embedding <=> ?) as vector_score', [$queryEmbedding]),
                // 🔥 Full-text rank (higher = better)
                DB::raw("ts_rank(search_vector, plainto_tsquery('english', ?)) as text_score", [$query]),
            ])
            ->where(function ($queryBuilder) use ($query, $queryEmbedding) {
                // 🔥 OR condition: vector OR full-text
                $queryBuilder->whereRaw('embedding <=> ? < 0.5', [$queryEmbedding])
                    ->orWhereRaw("search_vector @@ plainto_tsquery('english', ?)", [$query]);
            })
            // 🔥 Combined score = average of both
            ->orderByRaw('(vector_score + text_score) DESC')
            ->limit(5)
            ->get(['id', 'content']);

        if ($results->isEmpty()) {
            return response()->json([
                'answer' => "I don't know. No relevant documents found.",
                'results' => []
            ]);
        }

        // 🔥 Step 3: Generate Answer
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

### Step 4: Route

**`routes/web.php`**

```php
use App\Http\Controllers\HybridSearchController;

Route::get('/hybrid-search', [HybridSearchController::class, 'search']);
```

---

### Step 5: Test

```bash
./vendor/bin/sail artisan config:clear
./vendor/bin/sail artisan serve
```

**Test 1 — Semantic:**
```bash
curl "http://localhost/hybrid-search?q=How%20to%20return%20an%20item%3F"
```

**Test 2 — Exact:**
```bash
curl "http://localhost/hybrid-search?q=30%20days"
```

**Test 3 — Serial Number:**
```bash
curl "http://localhost/hybrid-search?q=Order%20%2312345"
```

---

## ✅ Day 11 Checklist

- [ ] Migration added (search_vector column)
- [ ] Seeder updated (generates search_vector)
- [ ] HybridSearchController created
- [ ] Hybrid search (vector + full-text) working
- [ ] All 3 tests passed

---

## 🎙️ Interview Prep

**Q: *"When does pure vector search fail?"** *

**A:** *"Pure vector search fails on exact matches like serial numbers, proper names, or specific years. For example, if a user searches for 'Order #12345', vector search looks for semantic meaning and finds nothing. Full-text search catches the exact serial number and returns the correct result."*

**Q: *"What is hybrid search?"** *

**A:** *"Hybrid search combines vector search (semantic) with full-text search (keyword). This ensures both meaning-based and exact-match queries return relevant results. I use PostgreSQL tsvector for full-text and pgvector for semantic search."*

**Q: *"How do you combine scores from two different search methods?"** *

**A:** *"I use a simple weighted average: (vector_score + text_score) / 2. Vector_score comes from cosine similarity, and text_score comes from ts_rank. Both are normalized to a 0-1 range."*

---

**Bata, ab hybrid search ka concept clear hai? 🐇🔥**