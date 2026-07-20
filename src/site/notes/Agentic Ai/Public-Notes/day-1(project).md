---
{"dg-publish":true,"permalink":"/agentic-ai/public-notes/day-1-project/","dg-note-properties":{}}
---


+-------------------+       +-----------------------+       +-------------------+
|   TERMINAL        |       |   LARAVEL (Code)      |       |   OLLAMA          |
|   (Tu)            |       |                       |       |   (Local Server)  |
+-------------------+       +-----------------------+       +-------------------+
| 1. php artisan     | ---> | 2. Command (Waiter)    |       |                   |
|    ai:chat "Hi"    |      |    reads input         |       |                   |
+-------------------+       +-----------------------+       +-------------------+
                            | 3. Command calls       |       |                   |
                            |    OllamaService       |       |                   |
                            +-----------+-----------+       +-------------------+
                                        |                       |
                                        | 4. Http::post()       |
                                        +---------------------> | 5. Generates    |
                                        |                       |    Response      |
                                        | 6. Gets JSON back     |                   |
                                        | <---------------------+                   |
                            +-----------+-----------+       +-------------------+
                            | 7. Service returns     |
                            |    clean text to       |
                            |    Command             |
                            +-----------+-----------+
                                        |
                            +-----------v-----------+
                            | 8. Command prints to   |
                            |    Terminal            |
                            +-----------------------+
                            | 9. Command saves to    |
                            |    MySQL Database      |
                            +-----------------------+