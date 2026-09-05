---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/dad-10.3/","dg-note-properties":{}}
---

Haan, ab problem clear hai. Actually .env mein day10_project nahi tha — hum jo assume kar rahe the woh galat tha.

Aapke earlier output mein clearly ye tha:

.env:26:DB_DATABASE=laravel
.env:33:DB_DATABASE=laravel


Aur container ke andar bhi:

DB_DATABASE=laravel
DB_DATABASE=laravel


Isliye Laravel laravel database use kar raha tha.

Exactly kya hua?

Aapke project mein .env mein 2 DB_DATABASE entries thi:

DB_DATABASE=laravel
...
DB_DATABASE=laravel


Aapne shayad .env mein manually ek jagah day10_project dekha/change kiya tha, but actual saved .env file still had laravel.

Humne ye command chalaya:

sed -i 's/^DB_DATABASE=laravel$/DB_DATABASE=day10_project/g' .env


Isne dono entries ko change kar diya:

DB_DATABASE=day10_project
DB_DATABASE=day10_project


Phir:

./vendor/bin/sail artisan config:clear


ke baad Laravel ne finally:

DB::connection()->getDatabaseName();


par:

"day10_project"


return kiya. ✅

Ab ek cleanup karo

.env mein DB_DATABASE sirf ek baar rakho:

DB_CONNECTION=pgsql
DB_HOST=pgsql
DB_PORT=5432
DB_DATABASE=day10_project
DB_USERNAME=sail
DB_PASSWORD=password


Bas. Ab tumhara Laravel → Sail → PostgreSQL → day10_project connection correctly working hai. 🎉

Aur haan, config/database.php mein default 'laravel' dikhna problem nahi tha — env('DB_DATABASE', 'laravel') ka matlab hai: .env mein value ho toh wahi use karo.