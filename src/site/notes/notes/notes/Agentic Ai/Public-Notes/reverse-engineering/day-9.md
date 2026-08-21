---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/reverse-engineering/day-9/","dg-note-properties":{}}
---

**Bhai, Reverse Engineering means we tear down Day 9 brick by brick.** 

We don’t just look at the code; we look at the **"Why"** behind every line. If I ask you a question in an interview, you should be able to draw this architecture on a whiteboard.

Let's strip it down to the core engine.

---

## 🔥 The Core Problem Day 9 Solves
**Problem:** 500 users upload 100-page PDFs at 9:00 AM.
**If you do it Synchronously:** The user stares at a blank screen for 45 seconds. Nginx times out (504 error). The server crashes.
**Day 9's Solution:** **Fire and Forget.** Save the document record instantly (takes 10ms), put the heavy work (chunking + AI) into a background queue, and tell the user *"Processing, check back later"*.

---

## 🧠 Reverse Engineering: The Architecture (Step-by-Step)

### 1. The "Contract" (Database)
Before we write code, we design the database. 
- **`documents` table:** Holds the original file name/content. It has a `status` column (`pending` → `processed`). This is your **"Order Status"**.
- **`document_chunks` table:** Holds the small pieces. Why `embedding` column? Because PostgreSQL + pgvector needs a place to store the 768 numbers (vectors) so we can search later. 

### 2. The "Dispatcher" (DocumentController)
When you hit `POST /documents/upload`:
1. We validate the input.
2. We save to the `documents` table with `status = pending`. (We get an `$document->id`).
3. **Crucial step:** `ProcessDocument::dispatch($document);` -> This pushes the *entire* $document object onto the queue. 
4. We immediately return a JSON response. **The HTTP request ends here.** The user is free.

### 3. The "Worker" (ProcessDocument Job)
This is the brain of the reverse engineering. The `handle()` method is called by the queue worker (`php artisan queue:work`). 
- **Step A (Chunking):** We pass the raw text to `Chunker` service. It splits "Return items within 30 days..." into smaller parts (max 500 characters). Why? Because LLMs have context windows. You can't send a 100-page PDF into the AI at once.
- **Step B (The AI Call):** For *every* chunk, we hit `http://localhost:11434/api/embeddings`. We send the text, Ollama returns an array of 768 floats. **This is where the AI "understands" the text.**
- **Step C (Storage):** We save this chunk + the vector into `document_chunks`.
- **Step D (Completion):** We update the `documents` table status to `processed`.

---

## 🐞 The "Hidden Bug" (Why did we use Raw HTTP?)

This is the most important part of the reverse engineering. 
The Laravel AI SDK (`v0.10.3`) has a bug. When you call:
```php
Ai::embeddings()->for(['text'])->generate(); 
```
It passes a **string** instead of an **array** to the provider, giving an ugly `TypeError`. 

**The Lesson:** *Never trust the framework blindly.* 
We bypassed the bug by using **Raw HTTP** (`Illuminate\Support\Facades\Http`) to talk to Ollama directly. This is a Senior Engineer move: If the fancy SDK fails, drop down to the raw REST API. It always works.

---

## 🔍 Debugging Toolkit (How to test Reverse Engineering)
If you build this and it breaks, here is your surgery kit:

| Check | Command | What it tells you |
| :--- | :--- | :--- |
| **Queue is empty?** | `php artisan queue:work` | If no job runs, your `dispatch()` didn't fire. |
| **Job failed?** | `php artisan queue:failed` | Lists failed jobs. Retry with `queue:retry all`. |
| **SDK bug?** | `php artisan tinker` -> `Ai::embeddings()...` | If it errors, switch to Raw HTTP (we did this). |
| **Embedding length wrong?** | Check DB migration. | Ollama = 768 dims. OpenAI = 1536 dims. Mismatch = SQL Error. |
| **No chunks?** | Check `Chunker.php`. | If the text is short, it returns 1 chunk. That's fine. |

---

## 📊 The Final Visual Flow (Reverse Engineered)

```text
User (Curl)
    ↓
[POST] /documents/upload
    ↓
Controller::upload()
    ├─ Document::create(['status' => 'pending'])
    ├─ ProcessDocument::dispatch($doc)   <-- 🔥 PUSH TO QUEUE
    └─ Return JSON (200 OK)  <-- User STOPS waiting.
                                  
                                  
[Background Worker] php artisan queue:work
    ↓
ProcessDocument::handle()
    ├─ (1) Chunker::chunk($doc->content) → ["part1", "part2"]
    ├─ (2) Loop:
    │   ├─ Http::post('/api/embeddings') → [0.12, -0.34, ...] (768 dims)
    │   └─ DocumentChunk::create([...])
    └─ (3) Document::update(['status' => 'processed'])
```

---

## 🎯 The Interview Pitch

> *"Sir, I built an Async Ingestion Pipeline. Instead of making the user wait for AI to think, I dispatch a job to the queue. The user gets instant feedback, and the queue worker processes the file in the background. We used `pgvector` to store embeddings with 768 dimensions from Ollama. We bypassed the SDK's bug and used Raw HTTP to ensure the embedding generation is reliable. This handles massive file uploads without crashing the web server."*

---

**Bhai, ab tu yeh flow apni tongue tip pe memorise kar le. Agar interviewer puche *"Tell me about your Day 9 project"*, tu yehi 5-point structure bolega. Ready for Day 10?** 🐇🔥