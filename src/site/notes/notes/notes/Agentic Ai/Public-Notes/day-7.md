---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/day-7/","dg-note-properties":{}}
---

Welcome to Day 7. Today, we fix the user experience.

Up until now, all your AI responses have arrived as a single block after a 5-10 second wait. In production, that’s a dealbreaker. Your users see a spinner and assume the app is broken.

SSE (Server-Sent Events) streams the response word‑by‑word as the model generates it. The **perceived** latency drops from ~5 seconds to ~300ms. The UX improves dramatically.

Today we will:
1.  Understand how SSE works in Laravel.
2.  Build a streaming agent using `$agent->stream()`.
3.  Handle the streaming response in the browser with JavaScript.
4.  Implement rate limiting to prevent abuse.

---

## 📖 Day 7: SSE Streaming & Concurrency

### 1. What is SSE? (Mental Model)

SSE is a **one‑way** persistent HTTP connection. The server sends chunks of data (`data: ...\n\n`) as they become available. The browser listens for these chunks via `EventSource` or a `fetch` stream.

- **One‑way:** Server → Browser only (unlike WebSockets, which are bidirectional).
- **Persistent:** The connection stays open until the server finishes or the client closes it.
- **Text‑based:** Data is sent as plain text with a simple `data:` prefix and a blank line (`\n\n`) to separate messages.

**Why SSE and not WebSockets?** For AI chat, you only need to push tokens from the server to the browser. SSE is simpler, uses standard HTTP, and requires no extra infrastructure.

---

### 2. The Magic: `$agent->stream()` vs `$agent->prompt()`

The Laravel AI SDK turns streaming into a **one‑line change**.

| Method | Behaviour | Use Case |
| :--- | :--- | :--- |
| `$agent->prompt()` | Waits for the full response, then returns it | Background jobs, CLI, API responses |
| `$agent->stream()` | Yields tokens as they arrive | Real‑time chat, user‑facing interactions |

When you call `$agent->stream()`, the SDK internally sets `stream: true` on the provider request, reads the chunked response, and yields each token as a string. You can then forward these tokens to the browser via SSE.

**Tool Calling with Streaming:** Agents that use tools can still stream. The SDK handles the tool call loop and streams the final response to the client.

---

### 3. Building the Streaming Endpoint (Laravel Backend)

Your backend needs to:
1.  Accept the user message.
2.  Create the agent.
3.  Call `$agent->stream()`.
4.  Yield each token as a SSE message.

**Controller (using `response()->eventStream()`):**

```php
// app/Http/Controllers/ChatStreamController.php
use App\Ai\Agents\OrderAssistant;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class ChatStreamController
{
    public function __invoke(Request $request): Response
    {
        $request->validate(['message' => 'required|string']);

        // Release session lock to avoid blocking other requests
        $request->session()->save();

        $agent = new OrderAssistant(auth()->id() ?? 1);

        return response()->eventStream(function () use ($agent, $request) {
            $stream = $agent->stream(
                $request->input('message'),
                provider: 'ollama',
                model: 'llama3.1'
            );

            foreach ($stream as $token) {
                // Send each token as an SSE event
                yield $token;
            }

            // Optional: Send a final DONE event
            yield 'data: [DONE]' . "\n\n";
        }, 200, [
            'Content-Type' => 'text/event-stream',
            'Cache-Control' => 'no-cache',
            'X-Accel-Buffering' => 'no', // 🔥 Crucial for Nginx
        ]);
    }
}
```

**Explanation:**
- `$request->session()->save()` releases the session lock. Without this, a long‑running stream would block other requests from the same user.
- `response()->eventStream()` sets the correct SSE headers and handles the formatting.
- `X-Accel-Buffering: no` tells Nginx (if you use it) not to buffer the response, so chunks reach the browser immediately.

**Route:**
```php
// routes/web.php
use App\Http\Controllers\ChatStreamController;

Route::post('/chat/stream', ChatStreamController::class);
```

---

### 4. Consuming the Stream in JavaScript (Frontend)

The browser needs to connect to the streaming endpoint and append tokens as they arrive. You can use the native `fetch` API with a `ReadableStream`.

**Example JavaScript (Alpine.js or vanilla):**

```javascript
async function sendMessage() {
    const response = await fetch('/chat/stream', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
        },
        body: JSON.stringify({ message: userInput }),
    });

    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let buffer = '';

    while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        buffer += decoder.decode(value, { stream: true });

        // Split by newlines to extract SSE messages
        const lines = buffer.split('\n');
        buffer = lines.pop() || '';

        for (const line of lines) {
            if (line.startsWith('data: ')) {
                const data = line.slice(6);
                if (data === '[DONE]') {
                    // Stream finished
                    return;
                }
                // Append token to the UI
                appendToken(data);
            }
        }
    }
}
```

**What’s happening:**
1.  `fetch` opens a connection to the streaming endpoint.
2.  `response.body.getReader()` gives you a stream of chunks.
3.  The loop decodes each chunk and splits it into SSE messages.
4.  Each `data: ...` line is a token. You append it to the chat bubble.

**UI state:** You’ll want to manage a `message` variable that accumulates the tokens and displays them in real time (Livewire or Alpine.js works well for this).

---

### 5. Handling Concurrency: Redis Rate Limiting

Streaming responses are long‑lived. If 100 users are streaming at the same time, they can exhaust your PHP‑FPM workers or hit provider rate limits.

**System Focus:** Preventing 100 concurrent streams from exhausting your PHP‑FPM workers.

**Solution:** Implement rate limiting at two levels:

**A. Provider Rate Limiting (LLM API)**

Use Laravel’s `Redis::throttle` to limit how many requests your app sends to the provider per second.

```php
use Illuminate\Support\Facades\Redis;

// Inside your agent service/controller
Redis::throttle('ollama_requests')->allow(10)->every(1)->then(function () use ($agent, $message) {
    // Proceed with the AI request
    $stream = $agent->stream($message);
}, function () {
    // Rate limit exceeded - return a friendly error
    return response()->json(['error' => 'Too many requests. Please wait.'], 429);
});
```

**B. Per‑User Rate Limiting**

Use Laravel’s built‑in `throttle` middleware on the route.

```php
Route::post('/chat/stream', ChatStreamController::class)
    ->middleware('throttle:10,1'); // 10 requests per minute per user
```

**Why Redis?** It’s atomic and works across multiple servers — essential for distributed systems.

---

### 6. Common Mistakes & Edge Cases

| Mistake | Consequence | Fix |
| :--- | :--- | :--- |
| Forgetting `X-Accel-Buffering: no` | Nginx buffers the full response before sending — no streaming | Add the header to your response |
| Not releasing the session lock | All other requests from that user are blocked | Call `$request->session()->save()` |
| Not handling connection aborts | The agent continues generating even after the user closes the tab | Check `connection_aborted()` in the loop and break if `true` |
| Buffering in PHP (`ob_flush()` missing) | PHP’s output buffer holds the data | Use `ob_flush()` and `flush()` after each `echo` |

---

### 7. Your Task Today

1.  **Create a streaming endpoint** for your `OrderAssistant` agent.
2.  **Add a simple HTML form** with a `textarea` and a `div` for the streaming response.
3.  **Write the JavaScript** to consume the stream and append tokens.
4.  **Test the streaming UX** — you should see the response appear word‑by‑word.
5.  **Implement per‑user rate limiting** on the route.
6.  **Update your README** to include streaming.

---

### 8. Quick Recap

| Concept | What You Learned |
| :--- | :--- |
| SSE | One‑way persistent HTTP connection for streaming |
| `$agent->stream()` | Yields tokens as they arrive |
| `response()->eventStream()` | Laravel’s helper for SSE responses |
| Redis Throttle | Prevents provider rate limits from being hit |
| `X-Accel-Buffering: no` | Disables Nginx buffering |

---

## 📌 Interview Questions (Be Ready For)

**Q1: *“How does SSE differ from WebSockets?”***

**Your Answer:** SSE is one‑way (server to client) and works over standard HTTP, making it simpler and infrastructure‑free for chat streaming. WebSockets are bidirectional and require a persistent server like Reverb or Soketi. For AI chat completions, you only need the server to push tokens to the browser, so SSE is the better fit.

**Q2: *“How do you handle 100 concurrent streaming users without exhausting your PHP workers?”***

**Your Answer:** I implement two rate‑limiting layers. First, I use `Redis::throttle` to limit the number of requests sent to the LLM provider per second. Second, I apply Laravel’s `throttle` middleware to limit requests per user. This prevents a single user from flooding the system and ensures fair resource allocation.

**Q3: *“What happens if the user closes the browser tab mid‑stream?”***

**Your Answer:** I check `connection_aborted()` inside the streaming loop. If it returns `true`, I break out of the loop and stop the agent generation. This prevents wasted compute on abandoned streams.

---

**Your action for today:** Implement the streaming endpoint and frontend. Test it with your `OrderAssistant`. Once you see word‑by‑word responses, you’ll understand the power of SSE.

Type **`DONE`** when your streaming chat works, and we’ll move to the next phase.