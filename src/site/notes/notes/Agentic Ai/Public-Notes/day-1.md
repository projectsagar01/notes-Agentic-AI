---
{"dg-publish":true,"permalink":"/notes/agentic-ai/public-notes/day-1/","dg-note-properties":{"permalink":"/agentic-ai/public-notes/day-1/"}}
---


### 🎯 The 3 Things You CARE About (Baaki sab bakwaas hai)

| Doc ne kya likha hai                                                               | Iska matlab kya hai? (Laravel language)                                                                                                  |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Endpoint URL**  <br>`http://localhost:11434/api/generate`                     | Ye tera **API URL** hai. Laravel HTTP Client isi pe `POST` request bhejega.                                                              |
| **2. Request Body**  <br>`{ "model": "gemma4", "prompt": "Why is the sky blue?" }` | Ye tera **Payload** hai. Laravel mein isko array mein daalna hai:  <br>`['model' => 'llama3.1', 'prompt' => 'Hello', 'stream' => false]` |
| **3. Response Field**  <br>`"response": "The sky is blue because..."`              | Ollama se jo **answer** aayega, wo is `response` key ke andar aayega.  <br>Laravel mein: `$response->json()['response']`                 |
### 🔍 1. Kaise pata karein ki konsa model install hai?

Ollama mein installed models dekhne ke liye yeh command use karein:

bash

ollama list

Ya fir:

bash

ollama ls

Is command se aapko local system par stored saare models ki list mil jaayegi, unke IDs, size, aur last modification time ke saath. Agar koi model install nahi hai toh list empty hogi.

### 🌐 2. Ollama server kis host/port par run kar raha hai?

- **Default:** Ollama server by default `127.0.0.1` (localhost) aur port `11434` par bind hota hai.
    
- **Custom:** Agar aapko change karna hai toh `OLLAMA_HOST` environment variable use karein. Jaise:
    
    bash
    
    export OLLAMA_HOST=0.0.0.0:11434
    
    (Note: `0.0.0.0` par bind karne se server aapke network par available ho jaata hai)
    

### 📝 3. Laravel `.env` file mein kaise configure karein?

Laravel project ki `.env` file mein aap yeh variables add kar sakte hain:

env

OLLAMA_URL=http://127.0.0.1:11434
OLLAMA_MODEL=llama3.1

Kuch Laravel packages isi format ko follow karte hain. Aap chahein toh `OLLAMA_HOST` bhi use kar sakte hain, lekin `OLLAMA_URL` zyada common hai.

**Note:** `.env` file mein changes karne ke baad Laravel cache clear karna na bhoolen:

bash

php artisan config:clear

### 🚀 4. Ollama server kaise run karein?

Ollama server ko chalane ke do tareeke hain:

**A. Direct run (foreground):**

bash

ollama serve

Ye command server ko foreground mein start karta hai. Agar aap terminal band karenge toh server band ho jaayega.

**B. Background mein run (recommended):**

- **Linux (systemd):** Ollama install script usually ek `ollama.service` systemd service create karta hai jo server ko background mein run karta hai. Aap ise manually manage kar sakte hain:
    
    bash
    
    sudo systemctl start ollama
    sudo systemctl status ollama
    
- **macOS:** Agar Ollama macOS app ke through install hai toh environment variables `launchctl` ke through set karne hote hain.
    
- **Manual background:** Aap is tarah bhi background mein daal sakte hain:
    
    bash
    
    ollama serve > ollama.log 2>&1 &
    
    Isse server background mein chalega aur logs `ollama.log` file mein store honge.
    

**Server status check:** Server chal raha hai ya nahi yeh check karne ke liye:

bash

curl http://localhost:11434/api/tags

Agar response aata hai toh server sahi chal raha hai.

### ✅ Summary Table

| Task                              | Command / Action                       |
| --------------------------------- | -------------------------------------- |
| Installed models dekhna           | `ollama list`                          |
| Server host/port check            | Default: `127.0.0.1:11434`             |
| Host change karna                 | `export OLLAMA_HOST=0.0.0.0:11434`     |
| `.env` mein add karna             | `OLLAMA_URL=http://127.0.0.1:11434`    |
| Server start (foreground)         | `ollama serve`/ollama run 'modelname'  |
| Server start (background - Linux) | `sudo systemctl start ollama`          |
| Server status check               | `curl http://localhost:11434/api/tags` |
