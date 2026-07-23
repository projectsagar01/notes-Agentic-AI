---
{"dg-publish":true,"permalink":"/notes/agentic-ai/public-notes/day-1-1-kya-padhna-h/","dg-note-properties":{"permalink":"/agentic-ai/public-notes/day-1-1-kya-padhna-h/"}}
---



### 📖 DOC 1: Laravel HTTP Client

**Link:** [https://laravel.com/docs/11.x/http-client](https://laravel.com/docs/11.x/http-client)

**Sirf yeh sections padhne hain (baaki sab skip):**

|Section|Kya padhna hai?|
|---|---|
|**Introduction**|Bas pehla paragraph - Laravel ne Guzzle ka wrapper banaya hai outgoing requests ke liye[](https://laravel.com/docs/7.x/http-client#introduction)|
|**Making Requests**|`Http::post()` kaise use karna hai[](https://laravel.com/docs/7.x/http-client#introduction)|
|**Response Methods**|`$response->json()`, `$response->body()`, `$response->status()` - yeh 3 methods[](https://laravel.com/docs/7.x/http-client#introduction)|
|**Request Data**|POST request mein data array kaise bhejte hain[](https://laravel.com/docs/7.x/http-client#introduction)|

**Skip:** Retry, Middleware, Macros, Testing, Events - inme se kuch mat padhna aaj.

---

### 📖 DOC 2: Laravel Artisan Console

**Link:** [https://laravel.com/docs/11.x/artisan](https://laravel.com/docs/11.x/artisan)

**Sirf yeh sections padhne hain:**

|Section|Kya padhna hai?|
|---|---|
|**Introduction**|Artisan kya hai - `php artisan` command line tool[](https://laravel.com/docs/12.x/artisan.)|
|**Writing Commands**|Custom command kaise banate hain[](https://laravel.com/docs/12.x/artisan.)|
|**Generating Commands**|`php artisan make:command` command|
|**Command Structure**|`$signature`, `$description`, `handle()` method - yeh teen cheezein|

**Skip:** Tinker, Command Allow List, All other sections.

---

### 📖 DOC 3: Ollama API (Generate Endpoint)

**Link:** [https://docs.ollama.com/api/generate](https://docs.ollama.com/api/generate)

**Sirf yeh dekho:**

|Section|Kya padhna hai?|
|---|---|
|**Endpoint URL**|`POST http://localhost:11434/api/generate`[](https://docs.ollama.com/api/generate)|
|**Request Body**|`model`, `prompt`, `stream` fields[](https://docs.ollama.com/api/generate)|
|**Response**|`response` field mein actual reply aata hai[](https://docs.ollama.com/api/generate)|

**Example request** jo docs mein hai[](https://docs.ollama.com/api/generate):

bash

curl http://localhost:11434/api/generate -d '{
  "model": "llama3.1",
  "prompt": "Why is the sky blue?",
  "stream": false
}'

**Skip:** Streaming, Images, Logprobs, Thinking output - inme se kuch mat padhna.

---

### ⏱️ Time Allocation (Day 1)

|Task|Time|
|---|---|
|HTTP Client docs padhna|20 min|
|Artisan docs padhna|15 min|
|Ollama API docs padhna|10 min|
|**Total reading time**|**45 min**|

Baaki time code likhne mein laga - OllamaService, Command, Migration.


**You need to create exactly 3 files:**

1. **Service Class** (`app/Services/OllamaService.php`)
    
    - Makes the `Http::post()` call to Ollama.
        
2. **Migration** (`database/migrations/...create_ai_logs_table.php`)
    
    - Columns: `user_message`, `ai_response`, `tokens_used`.
        
3. **Artisan Command** (`app/Console/Commands/AiChatCommand.php`)
    
    - Runs `php artisan ai:chat "Your message"`.
        
    - Calls the Service, prints the reply in the terminal, and saves it to the database.

