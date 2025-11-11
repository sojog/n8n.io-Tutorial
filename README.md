# n8n Workflow Automation Templates

Colecție completă de template-uri și tutoriale pentru automatizări n8n self-hosted.

## Despre n8n

n8n este o platformă open-source de automatizare workflow bazată pe noduri care permite conectarea aplicațiilor și automatizarea sarcinilor. Spre deosebire de alte platforme, n8n oferă self-hosting complet și control total asupra datelor.

### De ce n8n?

- 🎯 **Visual Workflow Builder**: Interfață bazată pe noduri intuitivă
- 🔓 **Open Source**: Fair-code license, self-hosting gratuit
- 🔗 **300+ Integrări**: Node-uri pre-construite + custom nodes
- 💻 **Self-Hosted**: Control complet, date pe serverul tău
- 🤖 **AI Native**: Integrare profundă cu OpenAI, Anthropic, etc.
- 💰 **Cost-Effective**: Gratuit pentru self-hosting, cloud de la $20/lună
- 🔒 **Securitate**: Date encrypted, conformitate GDPR

## Structura Proiectului

```
N8n.io/
├── docker-compose.yml              # Setup Docker pentru n8n
├── .env.example                    # Variabile de mediu template
├── README.md                       # Acest fișier
├── INSTRUCTIONS_EN.md             # Ghid complet în engleză
├── INSTRUCTIONS_RO.md             # Ghid complet în română
├── PROJECT_OVERVIEW.md            # Overview proiect
├── QUICK_START.md                 # Ghid rapid 10 minute
└── Workflows/                     # Template-uri workflow
    ├── 01-webhook-to-email.json
    ├── 02-google-sheets-sync.json
    ├── 03-slack-bot.json
    ├── 04-ai-content-assistant.json
    ├── 05-database-backup.json
    ├── 06-api-integration.json
    └── README.md
```

## Quick Start

### Cerințe Minime

- ✅ Docker & Docker Compose instalat
- ✅ 2GB RAM minim (4GB recomandat)
- ✅ 10GB spațiu disk
- ✅ Linux, macOS, sau Windows (cu WSL2)

### Instalare Rapidă (5 minute)

```bash
# 1. Clonează sau descarcă folderul
cd N8n.io

# 2. Copiază .env.example la .env
cp .env.example .env

# 3. Editează .env și setează parola
nano .env  # sau orice editor

# 4. Pornește n8n
docker-compose up -d

# 5. Deschide în browser
# http://localhost:5678
```

**🎉 Gata!** n8n rulează acum local.

## Configurare Docker Compose

### docker-compose.yml Minimal

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - WEBHOOK_URL=${WEBHOOK_URL}
      - GENERIC_TIMEZONE=${TIMEZONE}
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/workflows
      - ./backups:/home/node/backups

volumes:
  n8n_data:
    driver: local
```

### Cu PostgreSQL (Recomandat pentru Producție)

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: n8n_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -h localhost -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 10

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - WEBHOOK_URL=${WEBHOOK_URL}
      - GENERIC_TIMEZONE=${TIMEZONE}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/workflows
      - ./backups:/home/node/backups
    depends_on:
      postgres:
        condition: service_healthy
    links:
      - postgres

volumes:
  n8n_data:
    driver: local
  postgres_data:
    driver: local
```

## Variabile de Mediu (.env)

Creează fișierul `.env`:

```bash
# === Autentificare n8n ===
N8N_USER=admin
N8N_PASSWORD=schimba-parola-asta

# === Configurare Host ===
N8N_HOST=localhost
N8N_PORT=5678
WEBHOOK_URL=http://localhost:5678/

# === Timezone ===
TIMEZONE=Europe/Bucharest

# === PostgreSQL (dacă folosești) ===
POSTGRES_USER=n8n
POSTGRES_PASSWORD=parola-postgres-sigura
POSTGRES_DB=n8n

# === Encryption Key (Generează random!) ===
# Folosește: openssl rand -base64 32
N8N_ENCRYPTION_KEY=
```

## Comenzi Docker Utile

```bash
# Pornește n8n
docker-compose up -d

# Vezi logs
docker-compose logs -f n8n

# Oprește n8n
docker-compose down

# Restart
docker-compose restart n8n

# Update la ultima versiune
docker-compose pull
docker-compose up -d

# Backup date
docker-compose exec n8n n8n export:workflow --all --output=/home/node/backups/

# Curăță tot (ATENȚIE!)
docker-compose down -v
```

## Workflow-uri Template Disponibile

### 1. 🎣 Webhook to Email (Începător)
Primește date prin webhook și trimite notificări prin email.
- **Noduri**: Webhook, Send Email
- **Timp setup**: 5 minute

### 2. 📊 Google Sheets Sync (Intermediar)
Sincronizează date între Google Sheets și alte servicii.
- **Noduri**: Google Sheets Trigger, HTTP Request, Set
- **Timp setup**: 15 minute

### 3. 💬 Slack Bot (Intermediar)
Bot Slack pentru automatizări și comenzi.
- **Noduri**: Slack Trigger, IF, HTTP Request, Slack
- **Timp setup**: 20 minute

### 4. 🤖 AI Content Assistant (Avansat)
Asistent AI pentru generare și procesare conținut.
- **Noduri**: Schedule, OpenAI, Google Docs, Edit Fields
- **Timp setup**: 30 minute

### 5. 💾 Database Backup (Intermediar)
Backup automat baze de date cu notificări.
- **Noduri**: Schedule, Postgres, Google Drive, Slack
- **Timp setup**: 25 minute

### 6. 🔌 API Integration (Avansat)
Template generic pentru integrări API custom.
- **Noduri**: Webhook, HTTP Request, Code, Merge
- **Timp setup**: 40 minute

## Concepte Fundamentale n8n

### Noduri (Nodes)

#### 🎯 Trigger Nodes
- **Webhook**: Primește HTTP requests
- **Schedule**: Rulează la intervale (cron)
- **Email Trigger**: Monitorizează inbox
- **Execute Workflow Trigger**: Apelat de alt workflow

#### ⚙️ Action Nodes
- **HTTP Request**: Apelează orice API
- **Send Email**: Trimite email-uri
- **Google Sheets**: Citește/scrie în sheets
- **Slack**: Interacționează cu Slack
- **Postgres/MySQL**: Operații database

#### 🔀 Core Nodes
- **IF**: Logică condițională (branch)
- **Switch**: Multiple branch-uri
- **Merge**: Combină două flow-uri
- **Split Out**: Împarte items în multiple
- **Code**: JavaScript custom
- **Edit Fields (Set)**: Manipulează date

#### 🤖 AI Nodes
- **OpenAI**: ChatGPT, GPT-4, embeddings
- **Anthropic**: Claude AI
- **AI Agent**: Agent autonom cu tools
- **Vector Store**: Stocare embeddings

### Data Flow și Expressions

```javascript
// Accesează date din nodul curent
{{ $json.email }}

// Accesează date din nod specific
{{ $node["Webhook"].json["data"]["name"] }}

// Variabile sistem
{{ $now }}              // Timestamp curent
{{ $today }}            // Data curentă
{{ $workflow.id }}      // ID workflow

// Funcții
{{ $json.text.toLowerCase() }}
{{ $json.price * 1.19 }}  // Adaugă TVA
{{ new Date($json.timestamp).toISOString() }}
```

### Code Node JavaScript

```javascript
// Procesează toate items
for (const item of $input.all()) {
  item.json.processed = true;
  item.json.timestamp = new Date().toISOString();
}

return $input.all();
```

## Integrări Populare

### Google Services
- ✅ Google Sheets
- ✅ Google Drive
- ✅ Gmail
- ✅ Google Calendar
- ✅ Google Docs

### Communication
- ✅ Slack
- ✅ Discord
- ✅ Telegram
- ✅ Microsoft Teams
- ✅ Email (SMTP/IMAP)

### Databases
- ✅ PostgreSQL
- ✅ MySQL/MariaDB
- ✅ MongoDB
- ✅ Redis
- ✅ Supabase

### AI & ML
- ✅ OpenAI (GPT-4, DALL-E)
- ✅ Anthropic (Claude)
- ✅ Hugging Face
- ✅ Pinecone
- ✅ LangChain

### Development
- ✅ GitHub
- ✅ GitLab
- ✅ Docker
- ✅ SSH
- ✅ FTP/SFTP

## Best Practices

### 1. ✅ Structură Workflow

```
Trigger → Validare → Procesare → Error Handling → Output
```

### 2. 🛡️ Error Handling

- Activează "Continue On Fail" pe noduri critice
- Folosește Error Trigger Workflow separat
- Adaugă Try-Catch în Code nodes
- Logează erorile în Slack/Email

### 3. 🔄 Testing

- Testează cu "Test Workflow"
- Folosește Execute Once pentru debug
- Verifică fiecare nod individual
- Testează cu date reale

### 4. 📊 Monitoring

- Verifică Execution History regulat
- Setup Error Workflows
- Monitorizează resource usage
- Activează webhook errors notifications

### 5. 💰 Performance

```javascript
// ❌ Rău - procesează individual
for (item of items) {
  await httpRequest(item)  // 100 requests
}

// ✅ Bine - batch processing
await httpRequest(items)   // 1 request
```

### 6. 🔒 Securitate

- Folosește credentials pentru API keys
- Nu hardcoda parole în workflows
- Activează HTTPS pentru producție
- Backup regulat credențiale
- Rotație regulată parole

## Deployment Producție

### Cu Nginx Reverse Proxy + SSL

```yaml
# docker-compose.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - n8n

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    environment:
      - N8N_HOST=${DOMAIN}
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://${DOMAIN}/
    volumes:
      - n8n_data:/home/node/.n8n
```

### Configurare nginx.conf

```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    location / {
        proxy_pass http://n8n:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

## Backup și Restore

### Backup Automat

```bash
#!/bin/bash
# backup-n8n.sh

BACKUP_DIR="./backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Export workflows
docker-compose exec n8n n8n export:workflow --all --output=/home/node/backups/workflows_${DATE}.json

# Export credentials
docker-compose exec n8n n8n export:credentials --all --output=/home/node/backups/credentials_${DATE}.json

# Backup database (PostgreSQL)
docker-compose exec postgres pg_dump -U n8n n8n > ${BACKUP_DIR}/db_${DATE}.sql

echo "Backup completed: ${DATE}"
```

### Restore

```bash
# Restore workflows
docker-compose exec n8n n8n import:workflow --input=/home/node/backups/workflows_20240101.json

# Restore database
docker-compose exec -T postgres psql -U n8n n8n < ./backups/db_20240101.sql
```

## Troubleshooting

### 🔴 n8n nu pornește

```bash
# Verifică logs
docker-compose logs n8n

# Verifică resurse
docker stats

# Verifică porturi
netstat -tulpn | grep 5678
```

### 🔴 Database connection failed

```bash
# Verifică dacă Postgres rulează
docker-compose ps

# Test conexiune
docker-compose exec postgres psql -U n8n -d n8n -c "SELECT 1"
```

### 🔴 Workflows nu se salvează

- Verifică permissions pe volume: `chmod -R 777 n8n_data`
- Verifică spațiu disk: `df -h`
- Verifică logs: `docker-compose logs n8n`

## Resurse

### Documentație Oficială
- 🌐 [n8n Documentation](https://docs.n8n.io)
- 🎓 [n8n Academy](https://docs.n8n.io/courses/)
- 📘 [API Reference](https://docs.n8n.io/api/)

### Comunitate
- 💬 [n8n Community Forum](https://community.n8n.io)
- 💬 [Discord Server](https://discord.gg/n8n)
- 🐙 [GitHub](https://github.com/n8n-io/n8n)

### Video Tutorials
- 📺 [n8n YouTube Channel](https://www.youtube.com/@n8n-io)
- 📺 [Nate Herk - AI Automation](https://www.youtube.com/@nateherk)

### Tools
- 🔧 [Webhook.site](https://webhook.site) - Test webhooks
- 🔧 [JSONLint](https://jsonlint.com) - Validate JSON
- 🔧 [Cron Expression Generator](https://crontab.guru)

## Pricing

### Self-Hosted (Docker)
- ✅ **Gratuit complet**
- ✅ Execuții nelimitate
- ✅ Workflows nelimitate
- ✅ Toate feature-urile
- ⚠️ Necesită hosting propriu

### n8n Cloud
- 💰 **Starter**: $20/lună (2,500 executions)
- 💰 **Pro**: $50/lună (10,000 executions)
- 💰 **Enterprise**: Custom pricing

**Recomandare**: Self-hosted pentru control total și costuri reduse.

## Contribuții

Ai creat un workflow util? Contribuie!

1. Exportă workflow-ul (JSON)
2. Adaugă documentație
3. Trimite pull request

## Licență

n8n este fair-code licensed. Acest tutorial este creat cu scop educațional.

---

**Automatizează tot ce se poate automatiza! 🚀**

*Open-source. Self-hosted. Under your control.*

