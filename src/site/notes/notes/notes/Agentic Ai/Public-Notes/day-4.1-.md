---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/day-4-1/","dg-note-properties":{}}
---


## 🎯 DAY 4 PLAN

---

### 📚 Aaj Kya Seekhna Hai? (Theory - 1 hour)

Aaj sirf **2 concepts** seekhne hain:

|#|Concept|Kya Seekhna?|
|---|---|---|
|**1**|**Laravel AI SDK**|Official Laravel package jo 14 AI providers (OpenAI, Anthropic, Gemini, Ollama, etc.) ko **ek hi API** ke under laata hai[](https://laravel.com/blog/building-ai-agents-with-laravel-no-python-required?utm_campaign=laravelnews&utm_medium=link&utm_source=laravelnews). Agents, tools, structured output, streaming, vector search sab built-in hai[](https://laravel.com/blog/building-ai-agents-with-laravel-no-python-required?utm_campaign=laravelnews&utm_medium=link&utm_source=laravelnews).|
|**2**|**Agent Architecture**|Agent ek PHP class hai jo `Agent` interface implement karta hai. Isme instructions, conversation history, tools, aur output schema define karte hain.|

> **Interview Line:** _"I use Laravel's first-party AI SDK because it's maintained by the Laravel team, handles provider swapping, and reduces boilerplate—so I can focus on agent logic, not HTTP plumbing."_

---

### 📖 Kahan se Seekhna?

|Resource|Kya Padhna?|Time|
|---|---|---|
|**[Introducing the Laravel AI SDK](https://laravel.com/blog/introducing-the-laravel-ai-sdk)**|Agents kya hain, kaise banate hain. Example: `php artisan make:agent SalesCoach`[](https://laravel.com/blog/introducing-the-laravel-ai-sdk?ref=dailydev).|15 min|
|**[Laravel AI SDK Docs](https://laravel.com/docs/12.x/ai-sdk)**|Installation, configuration, provider setup[](https://laravel.com/docs/12.x/ai-sdk?trk=public_post_comment-text).|20 min|
|**[Ship AI with Laravel](https://laravel-news.com/ship-ai-with-laravel-building-your-first-agent-with-laravel-13s-ai-sdk)**|Fresh Laravel app se working support agent banane ka 10-min demo[](https://laravel-news.com/ship-ai-with-laravel-building-your-first-agent-with-laravel-13s-ai-sdk?utm_medium=feed&utm_source=feedpress.me&utm_campaign=Feed%3A+laravelnews).|20 min|
|**[Building AI Agents with Laravel](https://laravel.com/blog/building-ai-agents-with-laravel-no-python-required)**|SDK ka overview, agents kya kar sakte hain[](https://laravel.com/blog/building-ai-agents-with-laravel-no-python-required?utm_campaign=laravelnews&utm_medium=link&utm_source=laravelnews).|15 min|

> **Total Theory Time:** ~1 hour 10 minutes

---

### 🛠️ Aaj Kya Banana Hai? (Project)

**Project Name:** `HelloAgent`

**Flow:**

1. SDK install karo (`composer require laravel/ai`)
    
2. Configuration publish karo (`php artisan vendor:publish`)
    
3. Migrations run karo (`php artisan migrate`)
    
4. `.env` mein provider set karo (Ollama ya OpenAI)
    
5. Pehla agent banao (`php artisan make:agent HelloAgent`)
    
6. Agent ko prompt karo aur response dekho










Introducing the laravel AI SDK

To try the Laravel AI SDK, install it via Composer and add your API keys:

```
composer require laravel/ai
```

```
# .env
OPENAI_API_KEY=your-key-here
ANTHROPIC_API_KEY=your-key-here
```

# **Laravel AI SDK provider Support**

The Laravel AI SDK currently supports these providers:

Text Generation: OpenAI, Anthropic, Gemini, Groq, xAI

Image Generation: OpenAI, Gemini, xAI

Text-to-Speech: OpenAI, ElevenLabs

Speech-to-Text: OpenAI, ElevenLabs

Embeddings: OpenAI, Gemini, Cohere, Jina

Reranking: Cohere, Jina

Files: OpenAI, Anthropic, Gemini

Support for additional providers is planned.

# Building with agents and the Lravel AI SDK

Agents are the core of the Laravel AI SDK. Agents organize your AI interactions into clean, testable classes with clear responsibilities. Generate an agent with an [Artisan](https://laravel.com/docs/12.x/artisan) command:

```
php artisan make:agent SalesCoach
```

This creates a structured class where you define instructions, conversation context, available tools, and output schemas:

```
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Promptable;
use Stringable;

class SalesCoach implements Agent
{
    use Promptable;

    public function instructions(): Stringable|string
    {
        return 'You are a sales coach analyzing call transcripts and providing actionable feedback.';
    }
}
```

Once you've created an agent, you can prompt it using the `prompt` method:

$response = (new SalesCoach)->prompt('Analyze this sales transcript...');
return (string) $response;

Need to switch providers? Just pass the provider name:


$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: 'anthropic',
);