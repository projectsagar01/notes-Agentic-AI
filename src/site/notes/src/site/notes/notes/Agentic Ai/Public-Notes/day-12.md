---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/day-12/","dg-note-properties":{}}
---

The best api


---

# 🏗️ PULSEAI — THE COMPLETE ARCHITECTURE BLUEPRINT

---

## 📊 GRAPH 1: High-Level System Architecture

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          PULSEAI — HIGH LEVEL ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│   │   WhatsApp   │  │  Instagram   │  │   Facebook   │  │    Email     │     │
│   │  Cloud API   │  │  Graph API   │  │  Messenger   │  │  Brevo API   │     │
│   └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│          │                  │                  │                  │             │
│          └──────────────────┼──────────────────┼──────────────────┘             │
│                             ↓                  ↓                                │
│                    ┌────────────────────────────────────┐                      │
│                    │      WEBHOOK ROUTER (Laravel)      │                      │
│                    │   /webhook/whatsapp                │                      │
│                    │   /webhook/instagram               │                      │
│                    │   /webhook/facebook                │                      │
│                    │   /webhook/email                   │                      │
│                    └────────────────┬───────────────────┘                      │
│                                     ↓                                          │
│                    ┌────────────────────────────────────┐                      │
│                    │      EVENT: MessageReceived        │                      │
│                    │   Fires Event → Listeners          │                      │
│                    └────────────────┬───────────────────┘                      │
│                                     ↓                                          │
│          ┌──────────────────────────┼──────────────────────────┐              │
│          ↓                          ↓                          ↓              │
│   ┌──────────────┐         ┌──────────────┐          ┌──────────────┐        │
│   │ Save to DB   │         │ Queue Job    │          │ Log to AI    │        │
│   │ (Message)    │         │ (Process)    │          │ Observability │        │
│   └──────────────┘         └──────┬───────┘          └──────────────┘        │
│                                    ↓                                          │
│                    ┌────────────────────────────────────┐                      │
│                    │      AI AGENT (SupportAgent)       │                      │
│                    │   ┌─────────────────────────────┐  │                      │
│                    │   │ 1. RAG (Product Catalog)    │  │                      │
│                    │   │ 2. Hybrid Search (SKU+Text) │  │                      │
│                    │   │ 3. Tool Calling (Orders)    │  │                      │
│                    │   │ 4. Memory (Past Chats)      │  │                      │
│                    │   │ 5. Confidence Check         │  │                      │
│                    │   └─────────────────────────────┘  │                      │
│                    └────────────────┬───────────────────┘                      │
│                                     ↓                                          │
│                    ┌────────────────────────────────────┐                      │
│                    │      REPLY GENERATOR               │                      │
│                    │   - If confidence > 0.6: Reply     │                      │
│                    │   - If confidence < 0.6: Escalate  │                      │
│                    └────────────────┬───────────────────┘                      │
│                                     ↓                                          │
│                    ┌────────────────────────────────────┐                      │
│                    │      REPLY SENDER                  │                      │
│                    │   Send back via same channel       │                      │
│                    └────────────────────────────────────┘                      │
│                                                                                 │
│   ┌──────────────────────────────────────────────────────────────────────┐    │
│   │                     ADMIN DASHBOARD (SSE)                            │    │
│   │   - Live incoming messages streaming                                 │    │
│   │   - AI thinking in real-time                                         │    │
│   │   - Analytics: messages/day, AI replies, escalations                 │    │
│   └──────────────────────────────────────────────────────────────────────┘    │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 2: Database ER Diagram

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          PULSEAI — DATABASE SCHEMA                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐         ┌─────────────────────┐                          │
│  │   businesses    │         │      channels       │                          │
│  ├─────────────────┤         ├─────────────────────┤                          │
│  │ id (PK)         │◄────────│ id (PK)             │                          │
│  │ name            │         │ business_id (FK)    │                          │
│  │ email           │         │ type (enum)         │                          │
│  │ phone           │         │   whatsapp          │                          │
│  │ api_key         │         │   instagram         │                          │
│  │ created_at      │         │   facebook          │                          │
│  └────────┬────────┘         │   email             │                          │
│           │                   │ credentials (json)  │                          │
│           │                   │ is_active           │                          │
│           │                   └──────────┬──────────┘                          │
│           │                              │                                      │
│           │                              │                                      │
│           ↓                              ↓                                      │
│  ┌─────────────────┐         ┌─────────────────────┐                          │
│  │    products     │         │   conversations     │                          │
│  ├─────────────────┤         ├─────────────────────┤                          │
│  │ id (PK)         │         │ id (PK)             │                          │
│  │ business_id(FK) │         │ business_id (FK)    │                          │
│  │ name            │         │ channel_id (FK)     │                          │
│  │ sku             │         │ customer_id (FK)    │                          │
│  │ price           │         │ status (enum)       │                          │
│  │ stock           │         │   active            │                          │
│  │ description     │         │   escalated         │                          │
│  │ category        │         │   closed            │                          │
│  │ created_at      │         │ last_message_at     │                          │
│  └────────┬────────┘         └──────────┬──────────┘                          │
│           │                              │                                      │
│           │                              │                                      │
│           ↓                              ↓                                      │
│  ┌─────────────────┐         ┌─────────────────────┐                          │
│  │ product_chunks  │         │      messages       │                          │
│  ├─────────────────┤         ├─────────────────────┤                          │
│  │ id (PK)         │         │ id (PK)             │                          │
│  │ product_id (FK) │         │ conversation_id(FK) │                          │
│  │ chunk_index     │         │ role (enum)         │                          │
│  │ content         │         │   user              │                          │
│  │ embedding       │         │   assistant         │                          │
│  │   vector(1024)  │         │   system            │                          │
│  │ search_vector   │         │ content             │                          │
│  │   tsvector      │         │ citations (json)    │                          │
│  │ created_at      │         │ confidence (float)  │                          │
│  └─────────────────┘         │ created_at          │                          │
│                               └─────────────────────┘                          │
│                                                                                 │
│  ┌─────────────────┐         ┌─────────────────────┐                          │
│  │   customers     │         │   subscriptions     │                          │
│  ├─────────────────┤         ├─────────────────────┤                          │
│  │ id (PK)         │         │ id (PK)             │                          │
│  │ business_id(FK) │         │ customer_id (FK)    │                          │
│  │ name            │         │ business_id (FK)    │                          │
│  │ phone           │         │ channel_id (FK)     │                          │
│  │ email           │         │ subscribed (bool)   │                          │
│  │ channel_type    │         │ subscribed_at       │                          │
│  │ external_id     │         │ unsubscribed_at     │                          │
│  │ created_at      │         │ source (string)     │                          │
│  └─────────────────┘         └─────────────────────┘                          │
│                                                                                 │
│  ┌─────────────────┐         ┌─────────────────────┐                          │
│  │     orders      │         │    ai_logs          │                          │
│  ├─────────────────┤         ├─────────────────────┤                          │
│  │ id (PK)         │         │ id (PK)             │                          │
│  │ business_id(FK) │         │ business_id (FK)    │                          │
│  │ customer_id(FK) │         │ channel             │                          │
│  │ product_id (FK) │         │ prompt              │                          │
│  │ quantity        │         │ response            │                          │
│  │ total_price     │         │ tokens_used         │                          │
│  │ status          │         │ latency_ms          │                          │
│  │ created_at      │         │ confidence          │                          │
│  └─────────────────┘         │ created_at          │                          │
│                               └─────────────────────┘                          │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 3: Message Lifecycle (WhatsApp Example)

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                       MESSAGE LIFECYCLE — WHATSAPP                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  T+0ms                                                                          │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  Customer sends: "Bhaiya blue shirt hai kya? Size M"                    │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+50ms                                                                         │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  Meta WhatsApp API → POST /webhook/whatsapp                              │  │
│  │  Body: {from: "919876...", text: "Bhaiya blue shirt..."}                │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+100ms                                                                        │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  Laravel WebhookController → Validates signature → Fires Event           │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+150ms                                                                        │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  Event: MessageReceived → Listeners:                                      │  │
│  │    • SaveMessageListener → DB mein save                                   │  │
│  │    • LogListener → ai_logs mein log                                       │  │
│  │    • AnalyticsListener → count update                                     │  │
│  │    • ProcessMessageJob → Queue mein daalo                                │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+200ms                                                                        │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  HTTP Response 200 OK → Meta ko bhej do                                  │  │
│  │  (Meta ko 200 chahiye 5 seconds mein, warna webhook disable)            │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  ══════════════════════ BACKGROUND PROCESSING ══════════════════════════════   │
│                                     ↓                                          │
│  T+500ms                                                                        │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  Queue Worker picks ProcessMessageJob                                    │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+800ms                                                                        │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  AI Agent:                                                                │  │
│  │    1. Generate query embedding (Ollama)                                   │  │
│  │    2. Hybrid Search (pgvector + tsvector)                                │  │
│  │    3. Retrieve top 3 chunks                                              │  │
│  │    4. Load conversation memory (last 5 messages)                        │  │
│  │    5. Detect intent: "check_stock"                                       │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+1500ms                                                                       │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  Tool Call: check_inventory(product="blue shirt", size="M")              │  │
│  │  Result: {stock: 5, price: 799, sku: "BS-M-001"}                        │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+2500ms                                                                       │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  LLM Generation (llama3.1):                                              │  │
│  │  "Yes bhaiya! Blue shirt (M) available. ₹799. Free delivery.            │  │
│  │   7-day return policy. Order karna hai?"                                 │  │
│  │  Confidence: 0.92 (High)                                                 │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+3000ms                                                                       │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  Reply Sender: POST to Meta WhatsApp API                                 │  │
│  └──────────────────────────────────┬───────────────────────────────────────┘  │
│                                     ↓                                          │
│  T+3500ms                                                                       │
│  ┌──────────────────────────────────────────────────────────────────────────┐  │
│  │  Customer receives WhatsApp reply 🎉                                     │  │
│  └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  Total Latency: 3.5 seconds (User perceived: ~3 seconds)                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 4: AI Agent Decision Tree

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                       AI AGENT — DECISION TREE                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                    ┌─────────────────────────────┐                             │
│                    │   Incoming Message          │                             │
│                    │   "Blue shirt hai kya?"     │                             │
│                    └──────────────┬──────────────┘                             │
│                                   ↓                                            │
│                    ┌─────────────────────────────┐                             │
│                    │   Step 1: Intent Detection  │                             │
│                    │   (Structured Output JSON)  │                             │
│                    └──────────────┬──────────────┘                             │
│                                   ↓                                            │
│                    ┌─────────────────────────────┐                             │
│                    │   {                          │                             │
│                    │     "intent": "check_stock", │                             │
│                    │     "product": "blue shirt", │                             │
│                    │     "size": "M",             │                             │
│                    │     "confidence": 0.95       │                             │
│                    │   }                          │                             │
│                    └──────────────┬──────────────┘                             │
│                                   ↓                                            │
│              ┌────────────────────┼────────────────────┐                       │
│              ↓                    ↓                    ↓                       │
│     ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│     │   Intent:    │    │   Intent:    │    │   Intent:    │                  │
│     │ check_stock  │    │ create_order │    │    other     │                  │
│     └──────┬───────┘    └──────┬───────┘    └──────┬───────┘                  │
│            ↓                    ↓                    ↓                         │
│     ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│     │ Tool:        │    │ Tool:        │    │ RAG Search   │                  │
│     │ check_       │    │ create_      │    │ + LLM Reply  │                  │
│     │ inventory    │    │ order        │    │              │                  │
│     └──────┬───────┘    └──────┬───────┘    └──────┬───────┘                  │
│            ↓                    ↓                    ↓                         │
│     ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│     │ Result:      │    │ Result:      │    │ Result:      │                  │
│     │ stock: 5     │    │ order_id: 42 │    │ Generic      │                  │
│     │ price: 799   │    │ total: 799   │    │ Reply        │                  │
│     └──────┬───────┘    └──────┬───────┘    └──────┬───────┘                  │
│            └────────────────────┼────────────────────┘                         │
│                                 ↓                                              │
│                    ┌─────────────────────────────┐                             │
│                    │   Generate Final Reply      │                             │
│                    │   (Natural Language)        │                             │
│                    └──────────────┬──────────────┘                             │
│                                   ↓                                            │
│                    ┌─────────────────────────────┐                             │
│                    │   Confidence Check          │                             │
│                    └──────────────┬──────────────┘                             │
│                                   ↓                                            │
│              ┌────────────────────┼────────────────────┐                       │
│              ↓                    ↓                    ↓                       │
│     ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│     │ Confidence   │    │ Confidence   │    │ Confidence   │                  │
│     │  > 0.8       │    │  0.6 - 0.8   │    │  < 0.6       │                  │
│     │  (High)      │    │  (Medium)    │    │  (Low)       │                  │
│     └──────┬───────┘    └──────┬───────┘    └──────┬───────┘                  │
│            ↓                    ↓                    ↓                         │
│     ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│     │ Send Reply   │    │ Send Reply   │    │ Escalate to  │                  │
│     │ Immediately  │    │ + Log for    │    │ Human (HITL) │                  │
│     │              │    │  Review      │    │              │                  │
│     └──────────────┘    └──────────────┘    └──────────────┘                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 5: Hybrid Search Flow (Day 11)

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                       HYBRID SEARCH — TWO LANES MERGE                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  User Query: "BS-M-001 blue shirt"                                              │
│                                                                                 │
│              ┌─────────────────────────┴─────────────────────────┐             │
│              ↓                                                     ↓             │
│  ┌───────────────────────────┐                     ┌───────────────────────────┐│
│  │   LANE 1: VECTOR SEARCH   │                     │  LANE 2: FULL-TEXT SEARCH ││
│  ├───────────────────────────┤                     ├───────────────────────────┤│
│  │ 1. Generate Embedding     │                     │ 1. to_tsvector('english') ││
│  │    (mxbai-embed-large)    │                     │    → 'bs-m-001':1 'blue':2││
│  │                           │                     │    'shirt':3              ││
│  │ 2. WHERE embedding <=> ?  │                     │                           ││
│  │    < 0.5                  │                     │ 2. plainto_tsquery(       ││
│  │                           │                     │      'english', query)    ││
│  │ 3. Results:               │                     │                           ││
│  │    • Blue Shirt (0.85)    │                     │ 3. Results:               ││
│  │    • Blue T-Shirt (0.82)  │                     │    • BS-M-001 (rank 0.99) ││
│  │    • Sky Blue Shirt (0.78)│                     │    • Blue Shirt (rank 0.4)││
│  │    • Navy Shirt (0.71)    │                     │                           ││
│  └───────────────┬───────────┘                     └───────────────┬───────────┘│
│                  │                                                 │             │
│                  └─────────────────────┬───────────────────────────┘             │
│                                        ↓                                        │
│                        ┌───────────────────────────────┐                       │
│                        │   RRF MERGE (Reciprocal Rank  │                       │
│                        │   Fusion)                     │                       │
│                        │                               │                       │
│                        │   Score = 1/(60+rank_v) +     │                       │
│                        │           1/(60+rank_f)       │                       │
│                        └───────────────┬───────────────┘                       │
│                                        ↓                                        │
│                        ┌───────────────────────────────┐                       │
│                        │   FINAL RANKING               │                       │
│                        │   1. BS-M-001 (v:3, f:1)      │  ← Winner!           │
│                        │   2. Blue Shirt (v:1, f:2)    │                       │
│                        │   3. Blue T-Shirt (v:2, f:?)  │                       │
│                        └───────────────┬───────────────┘                       │
│                                        ↓                                        │
│                        ┌───────────────────────────────┐                       │
│                        │   TOP 3 → AI CONTEXT          │                       │
│                        │   → LLM → Reply with citation │                       │
│                        └───────────────────────────────┘                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 6: RAG Pipeline (Product Catalog)

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    RAG PIPELINE — PRODUCT CATALOG                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PHASE 1: INGESTION (One-time per product)                                     │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                                                                          │   │
│  │  Product PDF/CSV → Upload → Queue Job → ProcessProductJob               │   │
│  │                                            ↓                             │   │
│  │                                    ┌───────────────┐                    │   │
│  │                                    │  Chunker      │                    │   │
│  │                                    │  (500 chars)  │                    │   │
│  │                                    └───────┬───────┘                    │   │
│  │                                            ↓                             │   │
│  │                                    ┌───────────────┐                    │   │
│  │                                    │  Ollama       │                    │   │
│  │                                    │  /api/embed   │                    │   │
│  │                                    │  (1024 dims)  │                    │   │
│  │                                    └───────┬───────┘                    │   │
│  │                                            ↓                             │   │
│  │                                    ┌───────────────┐                    │   │
│  │                                    │  Save to DB   │                    │   │
│  │                                    │  product_     │                    │   │
│  │                                    │  chunks       │                    │   │
│  │                                    │  + tsvector   │                    │   │
│  │                                    └───────────────┘                    │   │
│  │                                                                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  PHASE 2: RETRIEVAL (Every user query)                                        │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                                                                          │   │
│  │  User: "Blue shirt ka price kya hai?"                                   │   │
│  │         ↓                                                                │   │
│  │  Generate Embedding → Hybrid Search → Top 3 Chunks                     │   │
│  │         ↓                                                                │   │
│  │  Build Prompt:                                                          │   │
│  │  "Context: [chunks] Question: [query] Answer based ONLY on context."   │   │
│  │         ↓                                                                │   │
│  │  LLM (llama3.1) → Answer with citations                                │   │
│  │         ↓                                                                │   │
│  │  "Blue shirt ₹799 hai. SKU: BS-M-001. [Citation: 5, 7]"               │   │
│  │                                                                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 7: Folder Structure (Complete)

```text
pulseai/
│
├── app/
│   ├── Ai/
│   │   ├── Agents/
│   │   │   ├── SupportAgent.php              # Main AI agent
│   │   │   ├── ProductSearchAgent.php        # RAG-based product queries
│   │   │   └── EscalationAgent.php           # Human handoff
│   │   ├── Tools/
│   │   │   ├── CheckInventoryTool.php
│   │   │   ├── GetPriceTool.php
│   │   │   ├── CreateOrderTool.php
│   │   │   └── EscalateToHumanTool.php
│   │   ├── Data/
│   │   │   ├── MessageData.php               # DTO
│   │   │   ├── OrderData.php
│   │   │   └── ProductData.php
│   │   └── Prompts/
│   │       └── SupportPrompt.php             # Versioned prompts
│   │
│   ├── Console/
│   │   └── Commands/
│   │       ├── SeedProductsCommand.php
│   │       └── TestAgentCommand.php          # CLI debugging
│   │
│   ├── Events/
│   │   └── MessageReceived.php
│   │
│   ├── Listeners/
│   │   ├── SaveMessageListener.php
│   │   ├── LogToAiLogsListener.php
│   │   └── DispatchProcessingJobListener.php
│   │
│   ├── Jobs/
│   │   ├── ProcessIncomingMessage.php        # Main AI processing
│   │   ├── GenerateOrderPdf.php
│   │   ├── SendDailyUpdate.php               # Scheduled broadcast
│   │   └── SyncChannelWebhook.php            # Channel setup
│   │
│   ├── Models/
│   │   ├── Business.php
│   │   ├── Channel.php
│   │   ├── Customer.php
│   │   ├── Conversation.php
│   │   ├── Message.php
│   │   ├── Product.php
│   │   ├── ProductChunk.php
│   │   ├── Order.php
│   │   ├── Subscription.php
│   │   └── AiLog.php
│   │
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   ├── WebhookController.php      # Receive messages
│   │   │   │   ├── ProductController.php
│   │   │   │   ├── OrderController.php
│   │   │   │   └── AnalyticsController.php
│   │   │   ├── DashboardController.php        # SSE streaming
│   │   │   └── AuthController.php             # Sanctum
│   │   ├── Middleware/
│   │   │   ├── VerifyWebhookSignature.php
│   │   │   └── EnsureBusinessIsActive.php
│   │   └── Resources/
│   │       ├── MessageResource.php
│   │       └── ProductResource.php
│   │
│   ├── Services/
│   │   ├── Channels/
│   │   │   ├── ChannelInterface.php
│   │   │   ├── WhatsAppChannel.php
│   │   │   ├── InstagramChannel.php
│   │   │   ├── FacebookChannel.php
│   │   │   └── EmailChannel.php
│   │   ├── WhatsAppService.php
│   │   ├── InstagramService.php
│   │   ├── FacebookService.php
│   │   ├── EmailService.php
│   │   ├── HybridSearchService.php
│   │   ├── EmbeddingService.php
│   │   └── AnalyticsService.php
│   │
│   └── Providers/
│       └── ChannelServiceProvider.php
│
├── database/
│   ├── migrations/
│   │   ├── 2026_01_01_000001_create_businesses_table.php
│   │   ├── 2026_01_01_000002_create_channels_table.php
│   │   ├── 2026_01_01_000003_create_customers_table.php
│   │   ├── 2026_01_01_000004_create_conversations_table.php
│   │   ├── 2026_01_01_000005_create_messages_table.php
│   │   ├── 2026_01_01_000006_create_products_table.php
│   │   ├── 2026_01_01_000007_create_product_chunks_table.php
│   │   ├── 2026_01_01_000008_create_orders_table.php
│   │   ├── 2026_01_01_000009_create_subscriptions_table.php
│   │   └── 2026_01_01_000010_create_ai_logs_table.php
│   │
│   └── seeders/
│       ├── DatabaseSeeder.php
│       ├── BusinessSeeder.php
│       └── ProductSeeder.php
│
├── resources/
│   └── views/
│       ├── dashboard.blade.php                # Admin panel
│       └── live-stream.blade.php              # SSE streaming view
│
├── routes/
│   ├── web.php
│   ├── api.php                                # API routes
│   └── channels.php                           # Broadcasting
│
├── tests/
│   ├── Feature/
│   │   ├── WebhookTest.php
│   │   ├── RagSearchTest.php
│   │   ├── ToolCallingTest.php
│   │   └── AgentMemoryTest.php
│   └── Unit/
│       ├── ChunkerTest.php
│       └── HybridSearchTest.php
│
├── docker/
│   └── compose.yaml                           # Sail + WAHA + Ollama
│
├── .env
├── .env.example
├── composer.json
├── README.md
└── docker-compose.yml
```

---

## 📊 GRAPH 8: Build Timeline (14 Days)

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                       PULSEAI — 14 DAY BUILD PLAN                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  DAY 1-2: FOUNDATION                                                            │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Database Design + Migrations                  │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Business + Channel Models                     │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Product + Customer Models                     │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Multi-tenant Setup                             │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  DAY 3-4: WHATSAPP CHANNEL                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  WhatsApp Cloud API Setup                       │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Webhook Endpoint (Receive)                     │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Reply Sender (Send)                            │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Message Storage                                 │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  DAY 5-6: AI AGENT (CORE)                                                       │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  SupportAgent Class                             │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  RAG Pipeline (Product Catalog)                 │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Hybrid Search (Day 11)                         │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Tool Calling (Check Inventory, Create Order)   │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Conversation Memory (Day 8)                    │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  DAY 7-8: INSTAGRAM + FACEBOOK                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Meta Graph API Setup                           │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Instagram Channel                              │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Facebook Messenger Channel                     │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Unified Webhook                                │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  DAY 9-10: EMAIL + SUBSCRIPTIONS                                                │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Brevo API Integration                          │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Email Channel                                  │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Opt-in/Opt-out System                          │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Daily Update Scheduler                        │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  DAY 11-12: DASHBOARD + ANALYTICS                                               │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Admin Dashboard (Blade)                        │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  SSE Live Streaming (Day 3, 7)                  │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Analytics: messages/day, AI replies, escalations│   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Export CSV Reports                             │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  DAY 13: TESTING + DOCKER                                                       │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Pest Tests (Webhook, RAG, Tools)               │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Docker Compose Setup                           │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  CI/CD (GitHub Actions)                         │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  DAY 14: POLISH + SHIP                                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  README with Architecture Diagram              │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Demo Video (5 min walkthrough)                 │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  Cold Email Template                            │   │
│  │  [■■■■■■■■■■■■■■■■■■■■]  GitHub Release v1.0                           │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 9: Learning Outcome Mapping (Day 1-12 → PulseAI)

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    LEARNING → PROJECT MAPPING                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  DAY 1: HTTP + Ollama + MySQL                                                   │
│  └─→ SupportAgent.php (AI core) + AiLog.php (observability)                    │
│                                                                                 │
│  DAY 2: JSON Extraction + DTO                                                   │
│  └─→ MessageData.php + OrderData.php + Structured output prompts               │
│                                                                                 │
│  DAY 3: SSE Streaming                                                           │
│  └─→ live-stream.blade.php + DashboardController@stream                        │
│                                                                                 │
│  DAY 4: Queues + Events + WAHA + Email + PDF                                   │
│  └─→ ProcessIncomingMessage Job + MessageReceived Event + WhatsAppChannel      │
│      + EmailChannel + GenerateOrderPdf Job                                     │
│                                                                                 │
│  DAY 5: Structured Output + Guardrails                                         │
│  └─→ Confidence check → HITL escalation if < 0.6                              │
│                                                                                 │
│  DAY 6: Tool Calling                                                            │
│  └─→ CheckInventoryTool + CreateOrderTool + GetPriceTool                      │
│                                                                                 │
│  DAY 7: SDK Streaming + Memory                                                  │
│  └─→ Real-time dashboard updates + Conversation memory                       │
│                                                                                 │
│  DAY 8: Conversation Memory                                                     │
│  └─→ Conversation + Message models + last 5 messages injection                │
│                                                                                 │
│  DAY 9: Async Ingestion                                                         │
│  └─→ ProcessProductJob + Chunker + ProductChunk model                          │
│                                                                                 │
│  DAY 10: RAG Pipeline                                                           │
│  └─→ ProductSearchAgent + citations + minSimilarity guardrail                 │
│                                                                                 │
│  DAY 11: Hybrid Search                                                          │
│  └─→ HybridSearchService (SKU exact + semantic)                               │
│                                                                                 │
│  DAY 12: Testing                                                                │
│  └─→ Pest tests for Webhook, RAG, Tools, Memory                               │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 10: Free Services Cost Breakdown

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                     PULSEAI — MONTHLY COST (ALL FREE)                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  SERVICE              FREE TIER              LIMIT                  COST       │
│  ─────────────────────────────────────────────────────────────────────────    │
│  WhatsApp Cloud API   1,000 conversations    per month              $0         │
│  Instagram Graph API  Unlimited (inbound)    rate-limited           $0         │
│  Facebook Messenger   Unlimited (inbound)    rate-limited           $0         │
│  Brevo Email          300 emails             per day                $0         │
│  Ollama (Local)       Unlimited              local GPU              $0         │
│  PostgreSQL           Unlimited              self-hosted            $0         │
│  pgvector             Unlimited              extension              $0         │
│  Redis (Cache)        Unlimited              self-hosted            $0         │
│  Docker / Sail        Unlimited              local machine          $0         │
│  Laravel              Unlimited              framework              $0         │
│  GitHub               Unlimited public repos free                   $0         │
│  ─────────────────────────────────────────────────────────────────────────    │
│                                                                                 │
│  TOTAL MONTHLY COST:                                                $0         │
│                                                                                 │
│  SCALING (When you get clients):                                               │
│  • WhatsApp over 1,000 conversations: ~₹0.78-3.50 per conversation             │
│  • Brevo over 300 emails/day: $25/month for 20,000 emails                      │
│  • VPS (DigitalOcean/Railway): $6-12/month                                     │
│  • Domain: $10/year                                                            │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 11: API Endpoints (For Client Integration)

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                       PULSEAI — API ENDPOINTS                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  AUTHENTICATION (Sanctum)                                                       │
│  ─────────────────────────────────────────────────────────────────────────     │
│  POST   /api/auth/register          → Business register                        │
│  POST   /api/auth/login             → Get API token                            │
│                                                                                 │
│  CHANNELS                                                                       │
│  ─────────────────────────────────────────────────────────────────────────     │
│  GET    /api/channels               → List connected channels                  │
│  POST   /api/channels/whatsapp      → Connect WhatsApp                         │
│  POST   /api/channels/instagram     → Connect Instagram                        │
│  POST   /api/channels/facebook      → Connect Facebook                         │
│  POST   /api/channels/email         → Connect Email                            │
│                                                                                 │
│  WEBHOOKS (Meta / WAHA will call these)                                        │
│  ─────────────────────────────────────────────────────────────────────────     │
│  POST   /webhook/whatsapp           → Receive WhatsApp message                 │
│  POST   /webhook/instagram          → Receive Instagram DM                     │
│  POST   /webhook/facebook           → Receive FB Message                       │
│  POST   /webhook/email              → Receive Email                            │
│                                                                                 │
│  PRODUCTS                                                                       │
│  ─────────────────────────────────────────────────────────────────────────     │
│  GET    /api/products               → List products                            │
│  POST   /api/products               → Add product                              │
│  POST   /api/products/ingest        → Upload catalog (PDF/CSV)                 │
│  GET    /api/products/search?q=...  → Hybrid search                            │
│                                                                                 │
│  CONVERSATIONS                                                                  │
│  ─────────────────────────────────────────────────────────────────────────     │
│  GET    /api/conversations          → List all conversations                   │
│  GET    /api/conversations/{id}     → Get conversation details                 │
│  POST   /api/conversations/{id}/reply → Human manual reply                     │
│  POST   /api/conversations/{id}/close → Close conversation                     │
│                                                                                 │
│  ORDERS                                                                         │
│  ─────────────────────────────────────────────────────────────────────────     │
│  GET    /api/orders                 → List orders                              │
│  GET    /api/orders/{id}            → Order details                            │
│  GET    /api/orders/{id}/pdf        → Download PDF                             │
│                                                                                 │
│  ANALYTICS                                                                      │
│  ─────────────────────────────────────────────────────────────────────────     │
│  GET    /api/analytics/overview     → Messages, replies, escalations           │
│  GET    /api/analytics/daily        → Daily message count                      │
│  GET    /api/analytics/export       → Download CSV                             │
│                                                                                 │
│  DASHBOARD (SSE)                                                                │
│  ─────────────────────────────────────────────────────────────────────────     │
│  GET    /dashboard                  → Admin dashboard (Blade)                  │
│  GET    /dashboard/stream           → SSE live stream endpoint                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 GRAPH 12: Queue Job Flow

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          QUEUE JOB FLOW                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  QUEUE: high                                                            │   │
│  │  Worker: 4 processes                                                    │   │
│  ├─────────────────────────────────────────────────────────────────────────┤   │
│  │  • ProcessIncomingMessage (AI Agent - critical, fast)                  │   │
│  │  • SendReply (Reply to user - time-sensitive)                          │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  QUEUE: default                                                         │   │
│  │  Worker: 2 processes                                                    │   │
│  ├─────────────────────────────────────────────────────────────────────────┤   │
│  │  • GenerateOrderPdf (Background, can wait)                            │   │
│  │  • SendEmailReceipt (Transactional)                                    │   │
│  │  • UpdateAnalytics (Counter increments)                               │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  QUEUE: low                                                             │   │
│  │  Worker: 1 process                                                      │   │
│  ├─────────────────────────────────────────────────────────────────────────┤   │
│  │  • SendDailyUpdate (Scheduled broadcast)                              │   │
│  │  • SyncChannelWebhook (Setup)                                         │   │
│  │  • CleanupOldMessages (Maintenance)                                   │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  Command: php artisan queue:work --queue=high,default,low                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 SUMMARY — Ek Nazar Mein PulseAI

| Aspect | Detail |
| :--- | :--- |
| **Project Name** | PulseAI |
| **Type** | Omnichannel AI Support Agent |
| **Channels** | WhatsApp, Instagram, Facebook, Email |
| **Core AI** | RAG + Hybrid Search + Tool Calling + Memory |
| **Stack** | Laravel, Ollama, PostgreSQL+pgvector, Redis, Docker |
| **Cost** | $0 (Free tier + Local LLM) |
| **Build Time** | 14 Days |
| **Unique Selling Point** | "Autonomous AI agent that reads your catalog, checks inventory, creates orders, and escalates to humans" |
| **Recruiter Pitch** | "I built a production-ready Omnichannel AI Agent in pure Laravel. No Python. No paid APIs." |
| **Legal** | 100% compliant (Consent-based, Opt-in only) |

---

**Bhai, ab tujhe complete blueprint mil gaya. Bata, "START DAY 12" bol, toh main pehla step (Database Design + Migrations) shuru karwa deta hoon. 🐇🔥**