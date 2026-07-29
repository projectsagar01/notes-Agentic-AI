---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/day-4-3/","dg-note-properties":{}}
---

Laravel Project Root
│
├── 📁 vendor/laravel/ai/          ← 🔥 PACKAGE (Kabhi mat touch karna)
│
├── 📁 config/
│   └── 📄 ai.php                  ← 🔥 CONFIG (Tune publish kiya)
│       ├── providers (Ollama, OpenAI, Gemini)
│       ├── models (gemini-2.0-flash-exp, gpt-4, llama3)
│       └── timeouts, retries, etc.
│
├── 📁 database/migrations/
│   └── 📄 ..._create_agent_tables.php  ← 🔥 DB TABLES (Tune migrate kiya)
│
├── 📁 app/Ai/Agents/              ← 🔥 TERI AGENTS (Tu yahan banayega)
│   └── 📄 HelloAgent.php          ← 🔥 TUNE BANAYA (Day 4 ka hero)
│
└── 📄 .env                        ← 🔥 PROVIDER SWITCH (Tune set kiya)
    ├── AI_PROVIDER=gemini
    ├── AI_MODEL=gemini-2.0-flash-exp
    └── GEMINI_API_KEY=...
    
    
    ## 📁 CHAPTER 3: PROJECT STRUCTURE (Kese banega?)

Tumhara Laravel folder kuch aisa dikhega:

text

app/
├── Ai/
│   ├── Agents/
│   │   └── SupportTriageAgent.php   <-- Yahan Agent ki Soch (Instructions) hai.
│   ├── Tools/
│   │   └── CreateTicketTool.php     <-- Yahan Agent ka Haath (Action) hai.
│   └── Data/
│       └── TicketData.php           <-- Yahan JSON ka Sancha (DTO) hai.
├── Http/
│   └── Controllers/
│       └── SupportController.php    <-- Web/API interface (Tum browser se test karoge)
└── routes/
    └── web.php                      <-- URL define karega

---
guys Want to tell you that there is a delay in this project and this is my 1st first real world project so i am facing lots of error and problem therefore i am also unable to upload vidoes