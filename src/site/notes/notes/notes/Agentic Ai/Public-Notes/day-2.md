---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/day-2/","dg-note-properties":{"permalink":"/notes/agentic-ai/public-notes/day-2/"}}
---



### 📖 Aaj (Day 2) ki SIRF 3 CHEEZEIN seekhni hain:

**(Baaki sab ignore karo, even the [Dev.to](https://dev.to/) blog)**

|#|Kya Seekhna Hai?|Kahan se Seekhna? (Exact short link)|Kitna Padhna?|
|---|---|---|---|
|**1**|**JSON Schema for Prompts**  <br>(AI ko JSON ka format kaise batayein)|Bas samajh lo ki prompt mein likhna hai:  <br>_"Respond ONLY in valid JSON with keys: name, issue, urgency"_|Sirf yeh line yaad rakhni hai.  <br>**Skip:** `required`, `properties`, `$schema` keywords.|
|**2**|**JSON Parsing in PHP**  <br>(Raw string ko array mein kaise badle)|PHP Manual: `json_decode()` function.  <br>Aur `json_last_error()` se error catch karna.|Sirf yeh 2 functions dekhne hain.  <br>5 minute.|
|**3**|**Laravel DTO (Data Transfer Object)**  <br>(Array ko strongly-typed object mein kaise badle)|Laravel Docs ya PHP Manual: Bas class banani hai jisme `public function __construct()` ho.  <br>**Spatie/Laravel-Data package mat padhna abhi.**|Sirf simple PHP class ka constructor dekhna hai.  <br>10 minute.|

---

### 🔥 "Theory" ka Exact Summary (Tere kaam ki baat)

1. **JSON Force kaise karein?**  
    Prompt mein `"Respond ONLY in valid JSON. No markdown, no extra text."` daal do. Itna hi hai. Agar AI ne galat JSON bheja, toh hum `preg_replace` se markdown hata denge (code already upar diya hai).
    
2. **json_decode kaise use karein?**
    
    php
    
    $array = json_decode($jsonString, true); // true = array mein convert karo
    if (json_last_error() !== JSON_ERROR_NONE) {
        // Galat JSON hai, fallback do
    }
    
3. **Simple DTO kya hota hai?**  
    Bas ek normal PHP class jisme saare properties `public` ho aur constructor ho:
    
    php
    
    class CustomerIssue {
        public function __construct(public string $name, public int $orderId) {}
    }
    
    _(Isse fayda yeh hai ki Controller mein `$dto->name` type-safe tarike se use kar sakte ho)_
    

---

### 🛠️ Ab Project kaise banayein? (Code)

Maine upar jo **`AiExtractCommand`** aur **`CustomerIssueDTO`** ka code diya hai, wo **already exact inhi 3 cheezon** ko implement karta hai.

- **JSON Schema:** Prompt ke andar likha hai (`"Respond ONLY in valid JSON with keys..."`).
    
- **JSON Parsing:** `json_decode()` aur `json_last_error()` use kiya hai.
    
- **DTO Mapping:** `CustomerIssueDTO::fromArray()` use kiya hai.
    

**Tumhe bas wo code copy karke run karna hai aur terminal mein output dekhna hai.** Jab wo chal jaaye, toh har line ko padhna ki kya ho raha hai. Yehi "Learning + Building" ek saath ho gaya.

---

### ❌ Aaj Time Waste Mat Karna (Skip these)

| Topic                              | Kyu nahi?                                               |
| ---------------------------------- | ------------------------------------------------------- |
| **Spatie Laravel-Data Package**    | Bahut advanced hai. 30 din baad kaam aayega, abhi nahi. |
| **Official JSON Schema Validator** | Tumhe AI se data nikalna hai, validate nahi karna.      |
| **Complex Nested JSON**            | Aaj bas flat JSON (single level) seekhna hai.           |