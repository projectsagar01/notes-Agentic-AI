---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/day-6/","dg-note-properties":{}}
---

## Day 6: Tool Calling & DB Integration with DeepSeek

### What to Learn

|Concept|What It Means|
|---|---|
|**Tool Calling**|AI requests to execute a function in your backend (e.g., query database)|
|**DeepSeek Support**|DeepSeek uses the exact same **OpenAI-compatible tool call format**|
|**Laravel AI SDK**|The SDK handles the difference. You just write tool classes and agent logic.|
|**5-Step Flow**|User → AI decides tool → AI returns `tool_call` → PHP executes → Result → Final answer|

### Where to Learn

|Resource|What to Read|
|---|---|
|**[DeepSeek API Docs — Function Calling](https://api-docs.deepseek.com/guides/function_calling)**|Tool calling format (OpenAI-compatible)|
|**[Laravel AI SDK — Tools](https://laravel.com/docs/11.x/ai-sdk#tools)**|`HasTools` trait, `Tool` interface|
|**[OpenAI Function Calling Guide](https://platform.openai.com/docs/guides/function-calling)**|Industry standard format (same as DeepSeek)|

### Build: `OrderAssistant`

- Create a `SearchOrdersTool` that queries the database.
    
- Register it in `OrderAssistant` using `HasTools`.
    
- Use DeepSeek as the provider in `.env`.
    

**DeepSeek Setup in `.env`:**

env

AI_PROVIDER=deepseek
DEEPSEEK_API_KEY=your-api-key
AI_MODEL=deepseek-chat

The Laravel AI SDK automatically handles the format translation. You don't need to change your agent code.

---

## When Will We Work with Images?

|Day|Topic|What You'll Do|
|---|---|---|
|**Day 10**|**PDFIngestor (Queue Ingestion)**|Extract text from PDFs → Chunk → Embed → Store|
|**Day 11**|**KnowledgeBot (RAG)**|Answer questions from documents (text extracted from PDFs)|
|**Day 18+**|**Ultra Roadmap (Pillar 2: Eyes)**|OCR, Vision LLMs, Image analysis (later phase)|

**Current Focus (Week 1):** Text-based AI agents (structured outputs, tool calling, memory).  
**Week 2:** RAG & PDF/Text ingestion.  
**Later:** Images, audio, video.

supportflow-ai/
├── app/
│   ├── Ai/
│   │   ├── Agents/              # Day 5 & 6 → TicketExtractorAgent, OrderAssistant
│   │   ├── Tools/               # Day 6 → SearchOrdersTool, CreateTicketTool
│   │   └── DTOs/                # Day 5 → TicketDTO
│   ├── Events/                  # Day 4 → TicketCreated
│   ├── Listeners/               # Day 4 → HandleTicketCreated
│   ├── Jobs/                    # Day 4 → GenerateTicketPdf, SendTicketNotification
│   ├── Services/                # Day 4 → WhatsAppService
│   ├── Models/                  # Ticket, Order
│   └── Http/Controllers/        # TicketController, ExtractController
├── config/ai.php                # Provider config (Ollama/DeepSeek/Gemini)
├── database/migrations/         # Tickets table
├── .env                         # AI_PROVIDER, API keys, WAHA, Brevo
└── resources/views/             # Blade forms (triage_form, extract_form)