---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/day-10.2/","dg-note-properties":{}}
---

**Bhai, ab theory ka phase pause. We'll build a complete, functional RAG pipeline project with code. Here's the **complete project structure** + **step-by-step execution guide** to build your **KnowledgeBot**.**

---

## 🏗️ COMPLETE PROJECT STRUCTURE — DAY 10

```text
day10_project/
├── app/
│   ├── Console/
│   │   └── Commands/
│   │       └── SeedDocuments.php         # 🌱 Seed documents with embeddings
│   ├── Http/
│   │   └── Controllers/
│   │       └── KnowledgeBotController.php # 🤖 RAG endpoint
│   └── Models/
│       ├── Document.php                  # 📄 Original documents
│       └── DocumentChunk.php             # 📄 Chunks with embeddings
├── database/
│   └── migrations/
│       ├── create_documents_table.php
│       └── create_document_chunks_table.php
├── routes/
│   └── web.php                           # 🌐 Routes
└── .env                                  # 🔑 Database + AI config
```

---

## 🚀 STEP-BY-STEP BUILD GUIDE (Copy-Paste Ready)

### Step 1: Create New Project

```bash
composer create-project laravel/laravel day10_project
cd day10_project
```

---

### Step 2: Install Dependencies

```bash
composer require laravel/ai
composer require pgvector/pgvector
```

---

### Step 3: Setup `.env`

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=day10_project
DB_USERNAME=postgres
DB_PASSWORD=your_password

AI_PROVIDER=ollama
AI_MODEL=llama3.1
OLLAMA_URL=http://localhost:11434
```

> **Note:** If using Docker, use `OLLAMA_URL=http://host.docker.internal:11434`

---

### Step 4: Enable pgvector Extension

```bash
sudo -u postgres psql -d day10_project -c "CREATE EXTENSION IF NOT EXISTS vector;"
```

---

### Step 5: Create Migrations

**Migration 1: `create_documents_table`**

```bash
php artisan make:migration create_documents_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('documents', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('content');
            $table->string('status')->default('pending');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('documents');
    }
};
```

**Migration 2: `create_document_chunks_table`**

```bash
php artisan make:migration create_document_chunks_table
```

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('document_chunks', function (Blueprint $table) {
            $table->id();
            $table->foreignId('document_id')->constrained()->onDelete('cascade');
            $table->integer('chunk_index');
            $table->text('content');
            $table->vector('embedding', 1024);  // msbai-embed-large = 1024 dims
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('document_chunks');
    }
};
```

**Run Migrations:**

```bash
php artisan migrate
```

---

### Step 6: Create Models

**`app/Models/Document.php`**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Document extends Model
{
    protected $fillable = ['title', 'content', 'status'];

    public function chunks()
    {
        return $this->hasMany(DocumentChunk::class);
    }
}
```

**`app/Models/DocumentChunk.php`**

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

### Step 7: Create Seeder Command (Embedding Generation)

```bash
php artisan make:command SeedDocuments
```

**`app/Console/Commands/SeedDocuments.php`**

```php
<?php

namespace App\Console\Commands;

use App\Models\Document;
use App\Models\DocumentChunk;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;

class SeedDocuments extends Command
{
    protected $signature = 'documents:seed';
    protected $description = 'Seed documents with embeddings using msbai-embed-large';

    public function handle(): void
    {
        $faqs = [
            ['title' => 'Return Policy', 'content' => 'You can return items within 30 days. Items must be unused.'],
            ['title' => 'Shipping Policy', 'content' => 'Free shipping on orders over $50. Delivery in 3-5 business days.'],
            ['title' => 'Warranty', 'content' => 'All products have a 1-year warranty against manufacturing defects.'],
            ['title' => 'Refund Policy', 'content' => 'Refunds are processed within 5-7 business days after return approval.'],
        ];

        foreach ($faqs as $faq) {
            $this->info('🔮 Embedding: ' . $faq['title']);

            $response = Http::post('http://localhost:11434/api/embeddings', [
                'model' => 'msbai-embed-large:latest',
                'prompt' => $faq['content'],
            ]);

            $embedding = $response->json()['embedding'] ?? [];

            if (empty($embedding)) {
                $this->error('❌ Failed to generate embedding for: ' . $faq['title']);
                continue;
            }

            $doc = Document::create([
                'title' => $faq['title'],
                'content' => $faq['content'],
                'status' => 'processed',
            ]);

            DocumentChunk::create([
                'document_id' => $doc->id,
                'chunk_index' => 0,
                'content' => $faq['content'],
                'embedding' => $embedding,
            ]);

            $this->info('✅ Embedded: ' . $faq['title'] . ' (Vector: ' . count($embedding) . ' dims)');
        }

        $this->info('✅ All documents seeded with embeddings!');
    }
}
```

**Run Seeder:**

```bash
php artisan documents:seed
```

**Expected Output:**
```
🔮 Embedding: Return Policy
✅ Embedded: Return Policy (Vector: 1024 dims)
🔮 Embedding: Shipping Policy
✅ Embedded: Shipping Policy (Vector: 1024 dims)
...
✅ All documents seeded with embeddings!
```

---

### Step 8: Create KnowledgeBot Controller

```bash
php artisan make:controller KnowledgeBotController
```

**`app/Http/Controllers/KnowledgeBotController.php`**

```php
<?php

namespace App\Http\Controllers;

use App\Models\DocumentChunk;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

class KnowledgeBotController extends Controller
{
    public function ask(Request $request)
    {
        $query = $request->input('query');

        // 🔥 Step 1: Semantic Search
        $chunks = DocumentChunk::query()
            ->whereVectorSimilarTo('embedding', $query, minSimilarity: 0.5)
            ->limit(3)
            ->get();

        // 🔥 Step 2: Guardrail (No context found)
        if ($chunks->isEmpty()) {
            return response()->json([
                'answer' => "I don't know. No relevant documents found.",
                'citations' => []
            ]);
        }

        // 🔥 Step 3: Build Context
        $context = $chunks->map(fn($c) => $c->content)->implode("\n\n---\n\n");
        $citationIds = $chunks->pluck('id')->toArray();

        // 🔥 Step 4: Build Prompt
        $prompt = "You are a helpful assistant. Answer based ONLY on the provided context.\n\n";
        $prompt .= "Context:\n\"\"\"\n{$context}\n\"\"\"\n\n";
        $prompt .= "Question: {$query}\n\n";
        $prompt .= "If the answer is not in the context, say \"I don't know.\" Do not make up information.";

        // 🔥 Step 5: Call Ollama (Generate Answer)
        $response = Http::post('http://localhost:11434/api/generate', [
            'model' => 'llama3.1',
            'prompt' => $prompt,
            'stream' => false,
        ]);

        $answer = $response->json()['response'] ?? 'No response from AI.';

        // 🔥 Step 6: Return Answer + Citations
        return response()->json([
            'answer' => $answer,
            'citations' => $citationIds,
            'chunks' => $chunks->pluck('content'),
        ]);
    }
}
```

---

### Step 9: Add Route

**`routes/web.php`**

```php
<?php

use App\Http\Controllers\KnowledgeBotController;
use Illuminate\Support\Facades\Route;

Route::get('/ask', [KnowledgeBotController::class, 'ask']);
```

---

### Step 10: Test The Project

```bash
php artisan serve
```

**Browser / cURL:**

```bash
curl "http://127.0.0.1:8000/ask?query=How to return an item?"
```

**Expected Output:**

```json
{
    "answer": "You can return items within 30 days. Items must be unused.",
    "citations": [1, 2, 3],
    "chunks": [
        "You can return items within 30 days. Items must be unused.",
        "Refunds are processed within 5-7 business days after return approval.",
        "Free shipping on orders over $50. Delivery in 3-5 business days."
    ]
}
```

---

## 🎯 COMPLETE CHECKLIST

| Step | Status |
| :--- | :--- |
| Create Laravel Project | ✅ |
| Install Dependencies (`laravel/ai`, `pgvector`) | ✅ |
| `.env` Setup | ✅ |
| pgvector Extension Enabled | ✅ |
| Migrations Created + Run | ✅ |
| Models Created | ✅ |
| Seeder Command Created + Run | ✅ |
| Controller Created | ✅ |
| Route Added | ✅ |
| `curl` Test Passed | ✅ |
| Citations Returned | ✅ |

---

**Bhai, ab tu ek fully functional RAG pipeline (KnowledgeBot) bana chuka hai. Type `NEXT` ab main tujhe **Day 11 — Hybrid Search** ki theory aur code dunga.** 🐇🔥