---
{"dg-publish":true,"permalink":"/agentic-ai/public-notes/day-3-1-project/","dg-note-properties":{}}
---

### 🎯 Project Name: **"Real-Time AI Streaming Chat"**

---

### 📌 Project Overview

Ek simple AI chat application jisme:

- User text input karega
    
- AI (Ollama/OpenAI) response **stream** hoga (word-by-word)
    
- Frontend par SSE (Server-Sent Events) use hoga
    
- Response complete hone ke baad full message database mein save hoga
    

---

### 🏗️ What You'll Build

|Component|Description|
|---|---|
|**Backend**|Laravel route + controller jo AI SDK ka `stream()` method use kare|
|**Frontend**|Simple HTML + JavaScript `EventSource` jo SSE events receive kare|
|**AI Integration**|Laravel AI SDK ka `agent()->stream()` ya `Ai::stream()`|
|**Persistence**|Stream complete hone ke baad full response DB mein save karo|