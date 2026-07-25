---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/roadmap/final-roadmap/","dg-note-properties":{}}
---

## 🎯 THE "REMOTE JOB CRACK" ROADMAP (Day 4 to Day 30)

### 🟢 PHASE 1: Production-Ready Foundation (Days 4-10)

|Day|Project|Topic|Interview Question Target|Kya Seekhna?|
|---|---|---|---|---|
|**4**|**Project: SDK Migrator**|**Switch from Raw HTTP to Laravel AI SDK**|_"Why did you choose Laravel AI SDK over raw HTTP?"_|`composer require laravel/ai`. Provider swapping, built-in tools, schema generation.|
|**5**|**Project: Multi-Provider Agent**|**Provider Swapping (Ollama ↔ OpenAI)**|_"How do you handle different LLM providers in production?"_|`.env` se model switch. Same code, different providers.|
|**6**|**Project: Ticket Extractor**|**Structured Outputs (JSON Schema)**|_"How do you ensure AI returns valid JSON?"_|`withStructuredOutput()` + PHP DTOs. Validation + retry.|
|**7**|**Project: Order Assistant**|**Tool Calling (Database Read)**|_"How does your agent interact with the database?"_|`HasTools` trait. Tool schema auto-generation.|
|**8**|**Project: StreamChat (SDK)**|**SSE Streaming (SDK Way)**|_"How do you handle long-running AI responses?"_|SDK's built-in `stream()` method. Word-by-word.|
|**9**|**Project: MemoryBot**|**Conversation Memory**|_"How does your agent remember previous chats?"_|`remember()` method. Redis + Database persistence.|
|**10**|**Project: VectorStore**|**pgvector + Embeddings**|_"How do you implement semantic search?"_|`vector` column. `whereVectorSimilarTo()`. HNSW index.|

---

### 🟡 PHASE 2: Agentic AI Core (Days 11-17)

|Day|Project|Topic|Interview Question Target|Kya Seekhna?|
|---|---|---|---|---|
|**11**|**Project: PDFIngestor**|**Document Ingestion + RAG**|_"How does your RAG pipeline work?"_|Upload → chunk → embed → store (Queue).|
|**12**|**Project: KnowledgeBot**|**RAG with Citations**|_"How do you prevent hallucinations?"_|Similarity search + citations. Refuse if no context.|
|**13**|**Project: ResilientAgent**|**Queues + Retries**|_"How do you handle API failures?"_|`$tries`, `$backoff`, timeout. Horizon dashboard.|
|**14**|**Project: ReActMath**|**ReAct Pattern (Loop)**|_"Explain the ReAct pattern."_|Thought → Action → Observation → Final Answer.|
|**15**|**Project: ResearchTeam**|**Multi-Agent Orchestration**|_"How do you coordinate multiple agents?"_|Supervisor pattern. ResearchAgent + WriterAgent + ReviewerAgent.|
|**16**|**Project: ApprovalGate**|**Human-in-the-Loop (HITL)**|_"How do you prevent AI from taking harmful actions?"_|Approval gate before refund/delete. Notifications.|
|**17**|**Project: TraceMe**|**Observability (Langfuse)**|_"How do you debug AI agents?"_|Langfuse self-host. Traces, spans, token usage.|

---

### 🔴 PHASE 3: Production Skills (Days 18-24)

|Day|Project|Topic|Interview Question Target|Kya Seekhna?|
|---|---|---|---|---|
|**18**|**Project: ParallelFetch**|**HTTP Client Pool**|_"How do you speed up agent tool calls?"_|3 APIs ek saath call. Parallel execution.|
|**19**|**Project: TestSuite**|**Testing (Pest)**|_"How do you test AI systems?"_|Mock LLM responses. Assert tool calls.|
|**20**|**Project: DockerStack**|**Docker Compose**|_"How do you deploy your AI stack?"_|Laravel + Postgres(pgvector) + Redis + Ollama.|
|**21**|**Project: AutoDeploy**|**CI/CD (GitHub Actions)**|_"How do you ensure code quality?"_|Tests + linting auto-run on push.|
|**22**|**Project: InboxOps (Init)**|**Project 1: Support Agent (Tables + Ingestion)**|_"Walk me through your project architecture."_|`inboxes`, `tickets`, `messages` tables. Ingestion API.|
|**23**|**Project: InboxOps (AI)**|**Project 1: Triage + Draft + Tools**|_"How does your support agent work?"_|Triage agent (structured output) + RAG draft + CreateTicketTool.|
|**24**|**Project: InboxOps (Polish)**|**Project 1: SSE + Tests + README**|_"How do you stream progress to users?"_|SSE stream (Triage → Draft → Ticket Created). CI passing.|

---

### ⚫ PHASE 4: Portfolio & Job Hunt (Days 25-30)

|Day|Project|Topic|Interview Question Target|Kya Seekhna?|
|---|---|---|---|---|
|**25**|**Project: ContractGuard**|**Project 2: Legal Extraction + Policy Engine**|_"How do you handle structured extraction?"_|Strict JSON Schema. Policy rules (non-AI).|
|**26**|**Project: ContractGuard (HITL)**|**Project 2: Human Approval + Audit**|_"How do you handle sensitive data?"_|Approval screen. Audit log.|
|**27**|**Project: RunbookPilot**|**Project 3: Incident + Multi-Agent**|_"How do you handle complex workflows?"_|Alert ingestion. DiagnosticsAgent + CommsAgent.|
|**28**|**Project: RunbookPilot (Polish)**|**Project 3: SSE Timeline + Export**|_"How do you show progress to users?"_|SSE timeline. Exportable run log.|
|**29**|**Project: Portfolio Polish**|**3 Projects: README + Diagrams + Demo Video**|_"Show me your best project."_|Architecture diagram. Demo video. README.|
|**30**|**Project: Cold Outreach**|**50 US SaaS Founders Cold Emails**|_"How do you market yourself?"_|Cold email script. GitHub links. Demo video.|

---

## 🧠 7 SKILLS THAT CRACK THE REMOTE JOB

|#|Skill|Interview Question|
|---|---|---|
|**1**|**Laravel AI SDK**|_"Why SDK over raw HTTP?"_|
|**2**|**Tool Calling**|_"How does your agent interact with DB/APIs?"_|
|**3**|**RAG + pgvector**|_"How do you prevent hallucinations?"_|
|**4**|**Multi-Agent Orchestration**|_"How do you coordinate multiple agents?"_|
|**5**|**Queues + Horizon**|_"How do you handle long-running AI tasks?"_|
|**6**|**Observability (Langfuse)**|_"How do you debug AI agents?"_|
|**7**|**Human-in-the-Loop**|_"How do you prevent harmful actions?"_|

---

## 📅 DAILY ROUTINE (Roz ka 8-hour shift)

|Time|Task|
|---|---|
|**8:00 - 9:00**|Theory (Read SDK docs, watch 1 video)|
|**9:00 - 12:00**|Code (Build the day's project)|
|**12:00 - 1:00**|Lunch|
|**1:00 - 4:00**|Code + Debug (Solve errors)|
|**4:00 - 5:00**|Write Tests + README|
|**5:00 - 6:00**|GitHub Push + 1 Tweet/LinkedIn post about what you built|
|**6:00 - 8:00**|Record 10-min video for YouTube|

---

## 🚫 WHAT TO SKIP (Time Waste)

|Topic|Why Skip?|
|---|---|
|**Python (for now)**|Pehle Laravel Agent master karo. Python baad mein.|
|**React / Vue**|Blade + Alpine.js enough for demos.|
|**TypeScript**|Overkill. Vanilla JS + Alpine.js kaafi hai.|
|**Fine-tuning**|Startup wants RAG, not fine-tuning.|
|**LangChain (Python)**|Laravel AI SDK is your stack.|