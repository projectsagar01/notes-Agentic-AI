---
{"dg-publish":true,"permalink":"/agentic-ai/day-1/","dg-note-properties":{}}
---


[Laravel](https://laravel.com/) ==provides an expressive, minimal API wrapper around the **Guzzle HTTP client**==. It allows your application to seamlessly make outgoing HTTP requests to interact with third-party web services and external APIs. [[1](https://laravel.com/docs/7.x/http-client), [2](https://laracasts.com/discuss/channels/laravel/http-client-1)]

Making Requests

You can leverage the `Illuminate\Support\\Facades\Http` facade to quickly dispatch basic HTTP requests: [[1](https://laravel.com/docs/7.x/http-client), [2](https://laravel-news.com/laravel-http-client)]

php

```
use Illuminate\Support\Facades\Http;

// GET Request
$response = Http::get('https://example.com');

// POST Request with Data
$response = Http::post('https://example.com', [
    'name' => 'Alex',
    'role' => 'Developer',
```

Inspecting Responses

The response facade returns an instance of `Illuminate\Http\Client\Response`, which contains convenient methods to verify status and retrieve payloads: [[1](https://laravel.com/docs/7.x/http-client), [2](https://www.youtube.com/watch?v=T1irYrYkTPM)]

- **`$response->body()`**: Returns the raw response body as a string.

- **`$response->json()`**: Decodes the response directly into a readable PHP array.

- **`$response->status()`**: Returns the exact HTTP status code integer.

- **`$response->successful()`**: Evaluates to true if the status code is between `200` and `299`.

- **`$response->failed()`**: Evaluates to true if the status code is `400` or higher. [[1](https://laravel.com/docs/7.x/http-client), [2](https://laravel-news.com/laravel-http-client), [3](https://eminentcoders.com/laravel-http-client-api/)]

php

```
if ($response->successful()) {
    $data = $re
```
Advanced Request Configurations

You can chain configuration methods directly onto the `Http` facade before dispatching the final verb: [[1](https://www.desarrollolibre.net/blog/laravel/laravel-http-client-first-connections-and-basic-exception-handling), [2](https://laravel-zero.com/docs/http-client)]

php

```
$response = Http::withHeaders([
    'X-Custom-Header' => 'Value'
])
->withToken('your-api-bearer-token') // Adds Authorization Bearer header
->timeout(5)                         // Terminates after 5 seconds of hanging
->retry(3, 100)                      // Retries 3 times, waiting 100ms apart
->get('https://example.com');
```

Handling Failures Gracefully

Instead of manually evaluating every condition, you can explicitly direct Laravel to throw a `RequestException` if a client error (`4xx`) or server error (`5xx`) occurs: [[1](https://laravel.com/docs/13.x/http-client), [2](https://github.com/JosephSilber/laravel-docs/blob/8.x/http-client.md), [3](https://devdocs-fr.github.io/laravel/7.x/http-client.html)]

php

```
$response = Http::get('https://example.com');

// Throws an exception if the request fails
$response->throw(); 

// Throws conditional exceptions
$response->throwIf($condition);
```

Quick Debugging

Laravel features on-the-fly debugging methods to inspect outbound parameters and returned diagnostics directly in execution: [[1](https://laravel-news.com/http-client-debugging), [2](https://www.youtube.com/watch?v=T1irYrYkTPM)]

php

```
// Dump the request/response details and continue execution
Http::get($url)->dump();

// Die and Dump the response layout immediately
Http::get($url)->dd();
```

Testing and Mocking

You can isolate your applications from network reliance during testing cycles by invoking the `Http::fake()` wrapper to simulate external responses without breaking actual integrations: [[1](https://laravel.com/docs/13.x/http-client), [2](https://devtalk94.medium.com/laravel-http-client-best-practices-features-e4f7e670b057), [3](https://laravel.com/docs/7.x/http-client), [4](https://laravel-news.com/laravel-http-client), [5](https://msaied.com/articles/laravel-contextual-http-clients-per-service-config-retries-and-middleware-stacks)]

php

```
use Illuminate\Support\Facades\Http;

Http::fake([
    // Stub a JSON response for GitHub endpoints
    'github.com/*' => Http::response(['data' => 'mocked-profile'], 200, ['Headers']),

    // Stub a string response for Google endpoints
    '://google.com*' => Http::response('Hello World', 200),
]);
```
