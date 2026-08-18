---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/roadmap/day-9-3-error/","dg-note-properties":{}}
---

the error debug guys
**Bhai, debugging ek skill hai — code likhne se zyada important. Main tujhe step-by-step batata hoon ki maine kaise errors identify kiye, kaise socha, aur kaise fix kiya.**

---

## 🔥 The Debugging Process (Step-by-Step)

### Phase 1: Error Recognition

**Error #1: `Class "Laravel\Ai\Facades\Ai" not found`**

| Observation | Inference |
| :--- | :--- |
| Facade not found | Facade alias register nahi hai |
| Tinker mein `Ai::embeddings()` fail | SDK ka facade kaam nahi kar raha |

**Why I Checked:** PhpStorm / Terminal se pata chala ki `Ai` facade resolve nahi ho raha. Iska matlab ya toh alias missing hai, ya service provider register nahi hai.

**What I Did Next:** Moved to `AiManager` (direct container resolution).

---

### Phase 2: Bypass Facade

**Error #2: `ArgumentCountError: Too few arguments to function OpenAiProvider::embeddings()`**

| Observation | Inference |
| :--- | :--- |
| `OpenAiProvider` called | SDK default provider OpenAI hai |
| `embeddings()` expects array, got 0 | Config mein embeddings driver set nahi hai |

**Why I Checked:** Tinker mein `$ai->embeddings('ollama')` se force kiya, but still `OpenAiProvider` error aaya. Iska matlab config mein `default_for_embeddings` still `openai` tha.

**Fix:** `config/ai.php` mein `default_for_embeddings` ko `'ollama'` karo.

---

### Phase 3: Config Fix

**Error #3: `OllamaProvider::embeddings(): Argument #1 ($inputs) must be of type array, string given`**

| Observation | Inference |
| :--- | :--- |
| Provider `ollama` choose ho raha hai | ✅ Good |
| But it expects array, gets string | `for()` method wrong argument passing |

**Why This Happened:** SDK version 0.10.3 mein `for()` method ka implementation thoda buggy hai. `for(['text'])` pass kar rahe hain, but provider `string` expect kar raha hai.

**What I Tried:**
1. `->for([$faq['content']])` → error
2. `->for($faq['content'])` → error
3. `->embedding($faq['content'])` → error

**Decision:** SDK embedding generator kaam nahi kar raha. Raw HTTP use karte hain.

---

### Phase 4: Raw HTTP Workaround

**Check #1: Confirm Ollama is Working**

```bash
curl http://localhost:11434/api/embeddings -d '{"model":"nomic-embed-text","prompt":"Hello"}'
```

**Result:** ✅ Valid embedding returned. Ollama is fine.

**Why I Tried This:** If raw HTTP works, problem is SDK, not infrastructure.

**Decision:** Raw HTTP for seeding → SDK for search.

---

### Phase 5: Search Works

**Search Test:**

```bash
curl "http://127.0.0.1:8000/search?q=How to return a damaged item?"
```

**Result:** Return Policy document returned.

**Why This Works:** `whereVectorSimilarTo()` uses pgvector SQL, not SDK embedding generator. So it works independently.

---

## 📌 Summary: Debugging Methodology

| Step | What I Did | Why |
| :--- | :--- | :--- |
| **1** | Check logs | Find exact error message |
| **2** | Isolate in Tinker | Test without web server |
| **3** | Force provider in Tinker | Bypass config issues |
| **4** | Check infrastructure (Ollama) | Confirm external service works |
| **5** | Raw HTTP test | Bypass SDK bug |
| **6** | Use SDK where it works | Search still uses SDK + pgvector |

---

## 🧠 Core Debugging Rules

| Rule | Why |
| :--- | :--- |
| **Always check logs first** | Error message is your best friend |
| **Isolate in Tinker** | Remove web server complexity |
| **Test infrastructure separately** | Make sure Ollama/DB works |
| **Bypass SDK if needed** | Sometimes frameworks have bugs |
| **Fallback to raw HTTP** | Always works if API is correct |
| **Verify after each fix** | `php artisan tinker` → test → confirm |

---

## ✅ What You Learned Today

1. **Read error messages** carefully (they tell you exactly what's wrong).
2. **Use Tinker** to test code without web server.
3. **Check infrastructure** (Ollama, DB) separately.
4. **Don't fight SDK** — use raw HTTP if SDK fails.
5. **Search still uses SDK** — `whereVectorSimilarTo()` works fine.

---

**Ab tu khud debugging kar sakta hai. Koi error aaye toh log dekho, Tinker mein test karo, infrastructure check karo, aur raw HTTP try karo — 90% issues solve ho jayenge.** 🐇🔥