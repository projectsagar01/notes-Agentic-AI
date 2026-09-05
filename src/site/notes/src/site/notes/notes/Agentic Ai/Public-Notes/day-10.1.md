---
{"dg-publish":true,"permalink":"/src/site/notes/notes/Agentic Ai/Public-Notes/day-10.1/","dg-note-properties":{}}
---

**Bhai, yeh sabse basic aur important sawaal hai. Ek baar samajh gaya toh kabhi confuse nahi hoga.**

---

## 🧠 Docker vs Container — The Analogy

| Term | Analogy | Matlab |
| :--- | :--- | :--- |
| **Docker** | **Car Engine** | Software jo containers ko run karne ki ability deta hai. System pe install hota hai. |
| **Container** | **Car** | Engine ka use karke banayi gayi ek running instance. Docker engine ke andar chalti hai. |

**Rule:** Pehle Docker (Engine) install karo. Phir us engine ke andar Containers (Cars) banao.

---

## 📦 PHASE 1: DOCKER INSTALL (Naye System pe)

### 🐧 Linux (Ultramarine/Fedora/Ubuntu)

**Step 1: Purana Docker hatao (agar hai toh)**

```bash
sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

**Step 2: Dependencies install karo**

```bash
sudo dnf install -y dnf-plugins-core
```

**Step 3: Docker repository add karo**

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```

**Step 4: Docker Engine install karo**

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Step 5: Docker service start karo**

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

**Step 6: User ko docker group mein add karo**

```bash
sudo usermod -aG docker $USER
```

> ⚠️ **Restart** karo ya logout karke wapas login karo.

**Step 7: Verify**

```bash
docker --version
docker compose version
```

---

### 🐧 Agar Ubuntu/Debian hai toh:

```bash
# 1. Purana hatao
sudo apt remove docker docker-engine docker.io containerd runc

# 2. Update
sudo apt update

# 3. Dependencies
sudo apt install -y ca-certificates curl

# 4. Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 5. Repository add
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 6. Install
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## 🏗️ PHASE 2: CONTAINER BANANA (Pehla Hello World)

Docker install ho gaya. Ab container banao.

**Step 1: Test container run karo**

```bash
docker run hello-world
```

> Ye ek test container hai. Agar "Hello from Docker!" print hota hai, toh Docker sahi chal raha hai.

**Step 2: Nginx container run karo (Web server)**

```bash
docker run -d -p 8080:80 --name my-nginx nginx
```

- `-d` → Background mein chale
- `-p 8080:80` → Host port 8080 se container port 80 map
- `--name my-nginx` → Container ka naam

**Browser mein:** `http://localhost:8080` — Nginx welcome page dikhega.

**Step 3: Container stop/start/delete**

```bash
docker stop my-nginx    # Band karo
docker start my-nginx   # Phir se chalao
docker rm -f my-nginx   # Delete karo
```

---

## 🖥️ PHASE 3: PORTAINER CONTAINER (GUI ke liye)

Ye ek container hai jo browser-based GUI deta hai.

```bash
docker volume create portainer_data

docker run -d \
  -p 8000:8000 -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

**Browser:** `https://localhost:9443`

---

## 🐳 PHASE 4: LARAVEL SAIL CONTAINER (Tera Project)

Laravel Sail bhi ek container stack hai.

**Step 1: Project mein Sail add karo**

```bash
composer require laravel/sail --dev
php artisan sail:install   # MySQL/PostgreSQL choose karo
```

**Step 2: Sail run karo (Containers build honge)**

```bash
./vendor/bin/sail up -d
```

**Step 3: Check karo**

```bash
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan queue:work
```

---

## 📊 CHEAT SHEET (Yaad Rakho)

| Command | Kya Karta Hai? |
| :--- | :--- |
| `docker ps` | Chal rahe containers dikhao |
| `docker ps -a` | Saare containers dikhao (band wale bhi) |
| `docker images` | Downloaded images dikhao |
| `docker run <image>` | Naya container banao |
| `docker stop <name>` | Container band karo |
| `docker rm <name>` | Container delete karo |
| `docker rmi <image>` | Image delete karo |
| `docker logs <name>` | Container logs dekho |

---

## 🔥 Summary (Tere Sawaal ka Jawab)

| Sawaal | Jawab |
| :--- | :--- |
| *"New system mein Docker kaise install hoga?"* | Engine install karo (`dnf install docker-ce`) |
| *"Container kese banega?"* | `docker run` command se |
| *"Docker install karne ke liye container banate hain?"* | **Nahi!** Pehle Docker Engine (software), phir uske andar Containers. |
| *"Pehle docker install karte hain ya container?"* | **Pehle Docker Engine** → Phir Container |

---

**Bhai, ab tera system Docker-ready hai. Kya main tera Day 9 project **Laravel Sail** mein migrate karne ka step-by-step de doon?** 🐇🔥