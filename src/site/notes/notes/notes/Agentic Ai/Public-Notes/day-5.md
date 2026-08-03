---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/day-5/","dg-note-properties":{}}
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