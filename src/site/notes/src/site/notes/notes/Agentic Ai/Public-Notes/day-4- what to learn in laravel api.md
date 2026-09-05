---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/day-4- what to learn in laravel api/","dg-note-properties":{}}
---

### 📝 What You MUST Learn (Sirf yeh 4 cheezein)

| #     | Topic                                 | Kyu?                                                         | Kitna Seekhna?                                                   |
| ----- | ------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------------- |
| **1** | **API Routes (`api.php`)**            | Agents ko mobile apps / frontends se connect karna.          | Bas `Route::post('/agent/run', [AgentController::class, 'run'])` |
| **2** | **Request Validation**                | User input ko sanitize karna (XSS, SQL injection se bachna). | Laravel `FormRequest` ka basic use.                              |
| **3** | **API Resources (Transforming data)** | Database se nikal kar JSON format mein convert karna.        | `UserResource::make($user)` jaise simple transformations.        |
| **4** | **Rate Limiting**                     | Agent ko baar-baar call karne se bachna (API bill bachana).  | `Route::middleware('throttle:60,1')`                             |

---

### 🚫 What to SKIP (Time Waste)

|Topic|Kyu Skip?|
|---|---|
|**RESTful Best Practices (HATEOAS, Hypermedia)**|Tera agent API internal hai, public website nahi.|
|**API Versioning (v1, v2)**|Jab tak product launch nahi hota, versioning waste hai.|
|**OAuth 2.0 / Socialite**|Jab tak user login nahi chahiye, skip.|
|**API Documentation (Swagger/OpenAPI)**|Baad mein karna, pehle product banao.|
|**Postman Collections**|Abhi nahi. Demo ke liye curl commands kaafi hain.|

---

### 🎯 The "Sagar" API Strategy

Tere agent ko bas **2 types** ke endpoints chahiye:

1. **Sync Endpoint:** User request → Agent sochta hai → Immediate JSON response. (Jab kaam 2 sec mein ho jaye).
    
2. **Async Endpoint:** User request → Queue mein job daalo → Return `job_id` → User baad mein result check kare. (Jab kaam 10-30 sec lage).
    

**Tune `POST /chat/stream` (Sync) already bana liya hai.**  
**Ab `POST /agent/run` (Async) banaunga Day 19-20 pe.**