---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/day-5/","dg-note-properties":{}}
---

## Project Structure

text

ticket-extraction-agent/
├── app/
│   ├── Ai/
│   │   ├── Agents/
│   │   │   └── TicketExtractorAgent.php          # Agent class with instructions & output() method
│   │   └── DTOs/
│   │       └── TicketDTO.php                     # DTO defining the output schema
│   ├── Http/
│   │   └── Controllers/
│   │       └── TicketController.php              # Controller: validates input, calls agent, returns DTO
│   └── Services/
│       └── TicketService.php (optional)          # Business logic layer (extract→store→notify)
├── tests/
│   ├── Feature/
│   │   └── TicketExtractionTest.php              # Tests hallucination handling, missing fields, schema compliance
│   └── Unit/
│       └── TicketDTOTest.php                     # DTO validation & type safety
├── config/
│   └── ai.php                                     # Provider config (OpenAI/Gemini API keys, model, defaults)
├── routes/
│   └── api.php                                    # POST /api/tickets/extract route
├── database/
│   └── migrations/
│       └── create_tickets_table.php               # Stores extracted ticket data (optional persistence layer)
├── app/Console/
│   └── Commands/
│       └── TestTicketExtractor.php                # Artisan command for CLI testing
├── .env                                            # OPENAI_API_KEY, GEMINI_API_KEY
└── composer.json                                   # Requires laravel/ai

---

## File-by-File Breakdown

|File|Responsibility|
|---|---|
|`TicketExtractorAgent.php`|Implements `Agent` and `HasStructuredOutput`. Defines system prompt (`instructions()`) and returns `TicketDTO::class` via `output()`. Uses `Promptable` trait[](https://laravel.com/blog/introducing-the-laravel-ai-sdk?ref=dailydev).|
|`TicketDTO.php`|Constructor with typed properties: `string $ticketId`, `int $priority`, `string $category`, `?string $summary`. Uses `#[Required]` for mandatory fields.|
|`TicketController.php`|Validates incoming request (`message` required). Instantiates agent → `$agent->prompt($request->message)`. Returns DTO as JSON. Adds manual guardrail: checks `$dto->ticketId` exists; logs hallucination if missing.|
|`TicketService.php`|Optional. Orchestrates: (1) call agent, (2) store to DB, (3) dispatch notification job. Keeps controller thin.|
|`TicketExtractionTest.php`|Tests: valid extraction, missing `ticketId` → exception, extra fields rejected, type mismatches caught. Uses `Agent::fake()` for deterministic testing.|
|`ai.php`|Sets default provider (`openai`), model (`gpt-4o-mini`), `max_tokens`, `temperature`.|
|`create_tickets_table.php`|Schema: `id`, `ticket_id` (unique), `priority` (tinyint), `category` (string), `summary` (text, nullable), `raw_message` (text), `created_at`.|
|`TestTicketExtractor.php`|`php artisan test:ticket "My payment failed"` → outputs structured JSON to terminal for rapid iteration.|

---

## Execution Flow

text

User Request (POST /api/tickets/extract)
    ↓
TicketController::extract()
    ↓ validates 'message' present
    ↓
TicketExtractorAgent->prompt($message)
    ↓ SDK reflects TicketDTO → generates JSON Schema
    ↓ sends to OpenAI with response_format
    ↓ provider returns constrained JSON
    ↓ SDK validates → maps to TicketDTO
    ↓
Controller guardrail: if $dto->ticketId === null → log hallucination → throw
    ↓
(Optional) TicketService::handle($dto) → store to DB → dispatch job
    ↓
Return JSON response with DTO fields

# Understanding the System — Mental Model

You are building a **structured extraction pipeline**:

1. **User submits raw text** (e.g., support ticket description).
    
2. **LLM extracts** specific fields (ticketId, priority, category, summary) and returns **only valid JSON** matching a predefined schema.
    
3. **Your application validates** the JSON before using it, logs failures, and optionally stores the result.
    

This pattern is fundamental for any AI integration that feeds data into your database, UI, or business logic. Once you internalize this structure, you can reuse it for any extraction task (invoices, contact forms, emails, etc.).

---

## High‑Level Flow

text

[Web Form] → [Controller] → [Agent] → [LLM Provider] → [JSON] → [DTO] → [DB/Response]
      │              │           │                            │         │
      │              │           └─ enforces schema          │         │
      │              └─ validates input & handles errors     │         │
      └─ user interacts                                          └─ API response or HTML

---

## File‑by‑File Responsibility & Dependencies

|File|Purpose|Why It Exists|What It Depends On|
|---|---|---|---|
|**`app/Ai/DTOs/TicketDTO.php`**|Defines the **contract** – what fields must be returned, their types, and which are required.|The SDK uses PHP reflection to generate JSON Schema from this DTO. This is the single source of truth for output shape.|None (pure data object).|
|**`app/Ai/Agents/TicketExtractorAgent.php`**|The **orchestrator** – holds system prompt, tells the SDK which DTO to use, and calls the provider.|Separates AI‑specific logic (prompt, model selection) from HTTP layer. Makes it testable and reusable.|Depends on `TicketDTO` (via `output()`).|
|**`app/Http/Controllers/TicketController.php`**|**HTTP entry point** – validates user input, calls the agent, applies guardrails, returns response.|Handles web concerns (request/response) and orchestrates the business flow.|Depends on `TicketExtractorAgent`.|
|**`routes/api.php`**|Maps URL `/api/tickets/extract` to controller method.|Standard Laravel routing; separates endpoint definitions.|None (router).|
|**`app/Models/Ticket.php`**|**Persistence** – stores extracted data in database.|Optional, but typical for production systems to record all extractions for auditing.|Depends on DB table.|
|**`database/migrations/...`**|Defines database schema.|Keeps DB schema versioned and reproducible.|None.|
|**`app/Console/Commands/TestTicketExtractor.php`**|CLI tool to test extraction without HTTP.|Great for development, debugging, and one‑off tests.|Depends on `TicketExtractorAgent`.|
|**`tests/Feature/TicketExtractionTest.php`**|Automated tests covering happy path and failures.|Prevents regressions; ensures guardrails work.|Uses Laravel testing helpers and `AI::fake()`.|
|**`config/ai.php`**, `.env`|Configuration – API keys, provider, model, timeouts.|Decouples environment from code; allows switching providers.|None.|

---

## Why Each Piece Exists (The Engineering Justification)

- **DTO + `output()`** → Forces the LLM to return structured JSON. Without this, you'd manually `json_decode()` and pray. With this, the SDK validates **before** your code sees the data.
    
- **Agent class** → Encapsulates the prompt, model, and output schema. If you later need to extract different entities (e.g., `InvoiceDTO`), you create a new agent without touching the old one.
    
- **Controller guardrail** → Even with structured outputs, the LLM might return `ticketId` as an empty string or `priority` as a negative number. The guardrail is your **final safety net** before data enters your system.
    
- **Logging hallucinations** → Critical for monitoring. If the agent repeatedly fails to extract required fields, you need to know so you can improve the prompt or adjust the schema.
    
- **Database** → Stores the raw message and extracted data. This enables auditing, retraining, and debugging. Without it, you lose forensic evidence when something goes wrong.
    
- **Tests** → The `AI::fake()` method lets you simulate the LLM response so you can test your guardrails, error handling, and business logic **without calling the real API** (which costs money and is slow).
    

---

## Mental Checklist to Re‑create This from Scratch

When facing a new extraction task, you will **always** follow these 6 steps:

1. **Define DTO** – What fields do you want? Type‑hint them. Mark required ones with `#[Required]` (or use nullable for optional).
    
2. **Create Agent** – Extend `Agent`, use `HasStructuredOutput`, `Promptable`, define `instructions()` (system prompt) and `output()` returning your DTO class.
    
3. **Write Controller** – Validate input, instantiate agent, call `prompt()`. Add at least one guardrail (e.g., check for non‑empty critical fields). Wrap in try‑catch for `StructuredOutputException`.
    
4. **Add Route** – Usually a `POST` endpoint.
    
5. **Optionally persist** – Create migration and model, save the DTO and raw input.
    
6. **Add tests** – Fake the provider, test valid and invalid scenarios.
    

That is the **entire pattern**. Once memorized, you can implement any AI extraction feature in 15 minutes.

---

## View File (Web Page)

Since you need a web interface to test, create a simple Blade view with a form and JavaScript to display the result.

### Create Route

Add to `routes/web.php`:

php

use App\Http\Controllers\TicketController;
Route::get('/tickets/extract', function () {
    return view('tickets.extract');
});

### Create Blade View

`resources/views/tickets/extract.blade.php`:

blade

<!DOCTYPE html>
<html>
<head>
    <title>Ticket Extractor</title>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <style>
        body { font-family: sans-serif; max-width: 600px; margin: 2rem auto; }
        textarea { width: 100%; height: 100px; padding: 0.5rem; }
        button { margin-top: 0.5rem; padding: 0.5rem 2rem; }
        #result { margin-top: 1rem; background: #f5f5f5; padding: 1rem; border-radius: 4px; white-space: pre-wrap; }
        .error { color: #d32f2f; background: #ffebee; }
    </style>
</head>
<body>
    <h1>Ticket Extraction</h1>
    <form id="extractForm">
        @csrf
        <div>
            <label for="message">Describe your issue:</label>
            <textarea id="message" name="message" required>My payment was charged twice</textarea>
        </div>
        <button type="submit">Extract</button>
    </form>
    <div id="result"></div>
    <script>
        document.getElementById('extractForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            const message = document.getElementById('message').value;
            const resultDiv = document.getElementById('result');
            resultDiv.innerHTML = 'Extracting...';
            resultDiv.className = '';
            try {
                const response = await fetch('/api/tickets/extract', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
                    },
                    body: JSON.stringify({ message }),
                });
                const data = await response.json();
                if (!response.ok) {
                    resultDiv.className = 'error';
                    resultDiv.innerHTML = `Error: ${data.error || 'Unknown error'}`;
                    return;
                }
                resultDiv.innerHTML = `
                    <strong>Ticket ID:</strong> ${data.ticketId}<br>
                    <strong>Priority:</strong> ${data.priority}/5<br>
                    <strong>Category:</strong> ${data.category}<br>
                    <strong>Summary:</strong> ${data.summary || '(none)'}
                `;
            } catch (error) {
                resultDiv.className = 'error';
                resultDiv.innerHTML = `Request failed: ${error.message}`;
            }
        });
    </script>
</body>
</html>

### Test in Browser

- Start server: `php artisan serve`
    
- Visit `http://localhost:8000/tickets/extract`
    
- Submit a message and see the extracted fields displayed.
    

---

## Recap: Where Each Piece Fits

text

User sees HTML form
    ↓ submits via fetch to /api/tickets/extract (API route)
TicketController::extract()
    ↓ validates message
    ↓ calls TicketExtractorAgent->prompt()
Agent sends prompt + schema to OpenAI
    ↓ returns structured JSON
    ↓ SDK maps to TicketDTO
Controller guardrail checks and logs
    ↓ (optionally) saves to tickets table
    ↓ returns JSON response
JavaScript displays fields in the page

---