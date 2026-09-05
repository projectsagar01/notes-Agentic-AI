---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/reverse-engineering/supportflow-ai/","dg-note-properties":{}}
---

we have to thing of using llm like we can use with https request and with using laravel-sdk
and laravel-sdk is prebuild given by laravel compnay 
# Laravel AI SDK

## [Introduction](https://laravel.com/docs/13.x/ai-sdk#introduction)

The [Laravel AI SDK](https://github.com/laravel/ai) provides a unified, expressive API for interacting with AI providers such as OpenAI, Anthropic, Gemini, and more. With the AI SDK, you can build intelligent agents with tools and structured output, generate images, synthesize and transcribe audio, create vector embeddings, and much more — all using a consistent, Laravel-friendly interface.

## [Installation](https://laravel.com/docs/13.x/ai-sdk#installation)

You can install the Laravel AI SDK via Composer:

```
composer require laravel/ai
```

Next, you should publish the AI SDK configuration and migration files using the `vendor:publish` Artisan command:

```
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
```

Finally, you should run your application's database migrations. This will create a `agent_conversations` and `agent_conversation_messages` table that the AI SDK uses to power its conversation storage:

```
php artisan migrate
```

after changing the .env file always run php artisan config:clear

project structure:

app/Ai
	/agent
		SupportTriageAgent.php
	/Tools
		CreateTicketTool.php
	/Events
		TicketCreated.php
	/http/Controller.php
		Controller.php
		TicketController.php
	/jobs
		GenerateTicketPdf.php
		SendTicketEmail.php
		SendTicketNotification.php
	/Listeners
		HandleTicketCreated.php
	/Models
		Ticket.php
		User.php
	/Providers
		AppServiceProvider.php
		EventServiceProvider.php
	/Services
		WhatsAppService.php

config/ai.php
database/migration
	/create_tickets_table.php
resources/
	/css
		app.css
	/js
		index.js
	/views/
		emails
			ticket_form.blade.php
		pdfs
			traige_form.blade.php
			welcome.blade.php
routes
	console.php
	web.php