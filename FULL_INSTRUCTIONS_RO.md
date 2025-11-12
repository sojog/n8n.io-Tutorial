# Ghid Complet n8n - Self-Hosted cu Docker

Un ghid cuprinzător pentru instalarea și utilizarea n8n self-hosted pe calculatorul local.

## Cuprins

1. [Introducere](#introducere)
2. [Instalare Docker](#instalare-docker)
3. [Setup n8n cu Docker Compose](#setup-n8n-cu-docker-compose)
4. [Configurare Inițială](#configurare-inițială)
5. [Înțelegerea Interfeței](#înțelegerea-interfeței)
6. [Primul Tău Workflow](#primul-tău-workflow)
7. [Lucrul cu Noduri](#lucrul-cu-noduri)
8. [Manipularea Datelor](#manipularea-datelor)
9. [Funcții Avansate](#funcții-avansate)
10. [AI și Agenți](#ai-și-agenți)
11. [Best Practices](#best-practices)
12. [Backup și Mentenanță](#backup-și-mentenanță)
13. [Depanare](#depanare)

## Introducere

### Ce este n8n?

n8n este o platformă open-source de automatizare workflow care permite conectarea aplicațiilor și automatizarea sarcinilor printr-o interfață vizuală bazată pe noduri. Este similară cu Zapier sau Make.com, dar oferă avantajul major al self-hosting-ului complet.

### Beneficii Cheie

- 🔓 **Open Source**: Fair-code license, cod sursă disponibil
- 💻 **Self-Hosted**: Rulează pe serverul/calculatorul tău
- 🔒 **Securitate și Privacy**: Datele tale rămân la tine
- 💰 **Cost Zero**: Gratuit pentru self-hosting (plătești doar infrastructura)
- 🎯 **Control Total**: Customizare completă
- 🔗 **300+ Integrări**: Node-uri pre-construite + community nodes
- 🤖 **AI Native**: Suport profund pentru OpenAI, Anthropic, LangChain
- 🛠️ **Extensibil**: Creează node-uri custom în JavaScript/TypeScript

### n8n vs Alte Platforme

| Feature | n8n (Self-hosted) | Make.com | Zapier |
|---------|-------------------|----------|--------|
| Cost | Gratuit | $9+/lună | $20+/lună |
| Execuții | Nelimitate | 10K/lună | Limitate |
| Data Privacy | 100% control | Cloud | Cloud |
| Customizare | Completă | Limitată | Limitată |
| Code Support | Da | Limitat | Limitat |
| AI Integration | Nativi | Extern | Extern |
| Self-hosting | Da | Nu | Nu |

## Instalare Docker

Docker este necesar pentru a rula n8n într-un container izolat.

### Windows

#### Pasul 1: Instalează WSL2

```powershell
# Deschide PowerShell ca Administrator
wsl --install

# Restart calculator
```

#### Pasul 2: Instalează Docker Desktop

1. Descarcă [Docker Desktop pentru Windows](https://www.docker.com/products/docker-desktop/)
2. Rulează installer-ul
3. La primul pornire, asigură-te că WSL2 backend este activat
4. Restart dacă este necesar

#### Verificare Instalare

```powershell
docker --version
docker-compose --version
```

### macOS

#### Cu Homebrew (Recomandat)

```bash
# Instalează Homebrew dacă nu există
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Instalează Docker Desktop
brew install --cask docker

# Pornește Docker Desktop din Applications
```

#### Manual

1. Descarcă [Docker Desktop pentru Mac](https://www.docker.com/products/docker-desktop/)
2. Deschide DMG și trage în Applications
3. Pornește Docker.app

#### Verificare Instalare

```bash
docker --version
docker-compose --version
```

### Linux (Ubuntu/Debian)

```bash
# Update pachete existente
sudo apt update
sudo apt upgrade -y

# Instalează dependențe
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Adaugă Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Adaugă repository Docker
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalează Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Adaugă user-ul la grupul docker
sudo usermod -aG docker $USER

# Logout și login din nou pentru a aplica permisiunile
```

#### Verificare Instalare

```bash
docker --version
docker compose version
```

## Setup n8n cu Docker Compose

### Structură Directoare

```bash
# Creează structura de foldere
mkdir -p ~/n8n-docker
cd ~/n8n-docker

mkdir -p workflows backups
```

### Configurare 1: Setup Basic (Începători)

Ideal pentru testare și învățare pe calculatorul local.

#### docker-compose.yml

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
      # Autentificare
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=schimba-parola-asta
      
      # Configurare generală
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - WEBHOOK_URL=http://localhost:5678/
      
      # Timezone
      - GENERIC_TIMEZONE=Europe/Bucharest
      
      # Debugging (opțional)
      - N8N_LOG_LEVEL=info
      
    volumes:
      # Date persistente n8n
      - n8n_data:/home/node/.n8n
      
      # Workflows locale (opțional)
      - ./workflows:/home/node/workflows
      
      # Backups
      - ./backups:/home/node/backups

volumes:
  n8n_data:
    driver: local
```

#### Pornire Rapidă

```bash
# Creează docker-compose.yml cu conținutul de mai sus

# Pornește n8n
docker-compose up -d

# Verifică logs
docker-compose logs -f n8n

# Oprește (Ctrl+C pentru logs)
# Deschide browser: http://localhost:5678
```

### Configurare 2: Cu PostgreSQL (Recomandat pentru Producție)

PostgreSQL oferă performanță mai bună și fiabilitate sporită față de SQLite (default).

#### docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: n8n_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=parola-postgres-sigura-aici
      - POSTGRES_DB=n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -h localhost -U n8n -d n8n']
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
      # Database PostgreSQL
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=parola-postgres-sigura-aici
      
      # Autentificare
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=parola-n8n-sigura-aici
      
      # Configurare
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - WEBHOOK_URL=http://localhost:5678/
      - GENERIC_TIMEZONE=Europe/Bucharest
      
      # Encryption key pentru credentials
      - N8N_ENCRYPTION_KEY=generează-un-key-random-aici
      
      # Log level
      - N8N_LOG_LEVEL=info
      
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

#### Generare Encryption Key

```bash
# Linux/macOS
openssl rand -base64 32

# Windows (PowerShell)
$bytes = New-Object byte[] 32
[System.Security.Cryptography.RandomNumberGenerator]::Create().GetBytes($bytes)
[Convert]::ToBase64String($bytes)
```

#### Pornire

```bash
docker-compose up -d

# Verifică că ambele servicii rulează
docker-compose ps

# Vezi logs
docker-compose logs -f
```

### Configurare 3: Setup Complet cu Nginx + SSL

Pentru deployment în producție cu domeniu propriu și HTTPS.

#### Structură Foldere

```bash
mkdir -p ~/n8n-production/{nginx,ssl}
cd ~/n8n-production
```

#### docker-compose.yml

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
    networks:
      - n8n_network
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -h localhost -U ${POSTGRES_USER} -d ${POSTGRES_DB}']
      interval: 5s
      timeout: 5s
      retries: 10

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
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
      - N8N_HOST=${DOMAIN}
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://${DOMAIN}/
      - GENERIC_TIMEZONE=${TIMEZONE}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - NODE_ENV=production
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/workflows
      - ./backups:/home/node/backups
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - n8n_network

  nginx:
    image: nginx:alpine
    container_name: n8n_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - n8n
    networks:
      - n8n_network

networks:
  n8n_network:
    driver: bridge

volumes:
  n8n_data:
    driver: local
  postgres_data:
    driver: local
```

#### .env

```bash
# PostgreSQL
POSTGRES_USER=n8n
POSTGRES_PASSWORD=parola-postgres-foarte-sigura
POSTGRES_DB=n8n

# n8n
N8N_USER=admin
N8N_PASSWORD=parola-admin-foarte-sigura
N8N_ENCRYPTION_KEY=generează-cu-openssl-rand-base64-32

# Domain & SSL
DOMAIN=n8n.yourdomain.com
TIMEZONE=Europe/Bucharest
```

#### nginx/nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {
    # Redirectează HTTP -> HTTPS
    server {
        listen 80;
        server_name n8n.yourdomain.com;
        return 301 https://$server_name$request_uri;
    }

    # HTTPS
    server {
        listen 443 ssl http2;
        server_name n8n.yourdomain.com;

        # SSL Certificates
        ssl_certificate /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        # SSL Configuration
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Logging
        access_log /var/log/nginx/n8n_access.log;
        error_log /var/log/nginx/n8n_error.log;

        # Max upload size
        client_max_body_size 50M;

        location / {
            proxy_pass http://n8n:5678;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # WebSocket support pentru n8n
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            
            # Timeouts
            proxy_connect_timeout 90s;
            proxy_send_timeout 90s;
            proxy_read_timeout 90s;
        }
    }
}
```

#### Obținere Certificat SSL cu Let's Encrypt

```bash
# Instalează certbot
sudo apt install certbot

# Obține certificat (asigură-te că domeniul pointează la server-ul tău)
sudo certbot certonly --standalone -d n8n.yourdomain.com

# Copiază certificatele
sudo cp /etc/letsencrypt/live/n8n.yourdomain.com/fullchain.pem ./ssl/
sudo cp /etc/letsencrypt/live/n8n.yourdomain.com/privkey.pem ./ssl/

# Setează permissions
sudo chown $USER:$USER ./ssl/*.pem
chmod 600 ./ssl/*.pem
```

## Configurare Inițială

### Prima Accesare

1. Deschide browser și navighează la `http://localhost:5678`
2. Vei vedea ecranul de autentificare Basic Auth
3. Introdu credențialele setate în docker-compose.yml:
   - User: `admin` (sau ce ai setat)
   - Password: parola ta

### Setup Wizard

La prima autentificare, n8n va afișa un wizard de configurare:

#### Pasul 1: Creează Owner Account
- **Email**: Adresa ta de email
- **First Name**: Prenumele tău
- **Last Name**: Numele tău
- **Password**: Parolă pentru contul n8n (diferită de Basic Auth)

**Important**: Acesta este contul tău principal n8n, diferit de Basic Auth.

#### Pasul 2: Setări Inițiale
- **Usage**: Personal / Company
- **Role**: Developer / Operations / etc.
- **Company Size**: Opțional

#### Pasul 3: Personalizare
- Alege template-uri recomandate (opțional)
- Skip sau explorează template-urile

### Dezactivare Basic Auth (Opțional)

După ce ai cont n8n creat, poți dezactiva Basic Auth:

```yaml
# În docker-compose.yml
environment:
  - N8N_BASIC_AUTH_ACTIVE=false  # Schimbă la false
  # Păstrează restul setărilor
```

```bash
# Restart container
docker-compose restart n8n
```

## Înțelegerea Interfeței

### Layout Principal

```
┌─────────────────────────────────────────────────────────┐
│  [n8n Logo]    Workflows  Credentials  Executions       │ ← Header
├─────────────────────────────────────────────────────────┤
│ ┌─────────┐                                             │
│ │  Nodes  │  [Canvas Workflow]                          │
│ │  Panel  │  Aici construiești workflow-urile           │
│ │         │                                             │
│ │ Trigger │  Trage noduri din stânga pe canvas          │
│ │ Actions │                                             │
│ │ Core    │                                             │
│ │ AI      │                                             │
│ └─────────┘                                             │
├─────────────────────────────────────────────────────────┤
│  [Test] [Execute] [Save]                                │ ← Footer
└─────────────────────────────────────────────────────────┘
```

### Componente Principale

#### 1. Header Navigation

- **Workflows**: Lista tuturor workflow-urilor tale
- **Credentials**: Gestionează conexiuni (API keys, OAuth, etc.)
- **Executions**: Istoric rulări workflow-uri
- **Settings**: Setări globale n8n
- **Help**: Documentație și suport

#### 2. Nodes Panel (Stânga)

Căutare și browse noduri:

```
📁 Trigger Nodes
  ├── Webhook
  ├── Schedule (Cron)
  ├── Email Trigger (IMAP)
  └── Execute Workflow Trigger

📁 Action Nodes
  ├── HTTP Request
  ├── Send Email
  ├── Google Sheets
  ├── Slack
  └── 300+ altele...

📁 Core Nodes
  ├── IF (Conditional)
  ├── Switch
  ├── Merge
  ├── Code (JavaScript)
  └── Edit Fields (Set)

📁 AI Nodes
  ├── OpenAI
  ├── Anthropic (Claude)
  ├── AI Agent
  └── Vector Store
```

#### 3. Canvas (Centru)

Zona principală unde construiești workflow-uri prin drag & drop și conectare noduri.

**Comenzi Canvas:**
- **Drag noduri**: Click și trage din Nodes Panel
- **Conectare**: Trage de pe punctul din dreapta nodului către alt nod
- **Selectare**: Click pe nod
- **Ștergere**: Select nod + Delete/Backspace
- **Zoom**: Scroll mouse sau pinch pe touchpad
- **Pan**: Ține Space + drag mouse

#### 4. Node Editor (Dreapta)

Când selectezi un nod, panoul din dreapta arată:
- **Parameters**: Setări nod
- **Settings**: Opțiuni avansate (timeout, retry, etc.)
- **Data**: Output-ul nodului după execuție

#### 5. Footer

- **Test Workflow**: Rulează workflow manual (pentru development)
- **Execute Workflow**: Execută o singură dată
- **Save**: Salvează workflow-ul
- **Active Toggle**: Activează/Dezactivează workflow (pentru producție)

### Workflows List

```
┌─────────────────────────────────────────────────────┐
│  [+ New Workflow]                    [Search...]    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  📋 Email to Slack Notifier          ● Active      │
│     Last run: 2 min ago | Success                  │
│                                                     │
│  📋 Daily Database Backup            ○ Inactive    │
│     Last run: 1 day ago | Success                  │
│                                                     │
│  📋 AI Content Generator             ● Active      │
│     Last run: Error | 5 min ago                    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Credentials Management

Gestionare centralizată a conexiunilor:

```
Credentials → [+ Add Credential]

Tipuri Populare:
├── Google OAuth2
├── Slack OAuth2
├── OpenAI API
├── PostgreSQL
├── HTTP Basic Auth
└── Header Auth
```

**Best Practice**: Folosește întotdeauna Credentials, niciodată hardcode API keys în noduri.

### Execution History

```
Executions → Filters: [All | Success | Error | Waiting]

┌────────────────────────────────────────────────────┐
│ Workflow: Email Notifier                          │
│ Status: ✓ Success | Duration: 1.2s                │
│ Started: 2024-01-15 10:30:45                      │
│ [View Details]                                     │
├────────────────────────────────────────────────────┤
│ Workflow: API Integration                         │
│ Status: ✗ Error | Duration: 0.5s                  │
│ Started: 2024-01-15 10:25:12                      │
│ Error: Connection timeout                         │
│ [View Details] [Retry]                            │
└────────────────────────────────────────────────────┘
```

## Primul Tău Workflow

Să creăm un workflow simplu: **Webhook → Procesare → Email**

### Pasul 1: Creează Workflow Nou

1. Click pe **"+ New Workflow"** (sau Ctrl/Cmd + N)
2. Vei vedea un canvas gol
3. Denumește workflow-ul: Click pe "My workflow" sus → "Webhook to Email"

### Pasul 2: Adaugă Trigger (Webhook)

1. Click pe **"+ Add first step"** sau click pe Nodes Panel
2. Caută **"Webhook"**
3. Click pe **"Webhook"** node
4. În configurare:
   - **HTTP Method**: POST
   - **Path**: lasă gol (se generează automat)
   - **Response Mode**: Respond Immediately
5. Click **"Listen for Test Event"**

Vei vedea un URL generat, ceva de genul:
```
http://localhost:5678/webhook-test/abc123def456
```

### Pasul 3: Testează Webhook-ul

Deschide un nou terminal și trimite un test:

```bash
curl -X POST http://localhost:5678/webhook-test/abc123def456 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Ion Popescu",
    "email": "ion@example.com",
    "message": "Vreau să aflu mai multe despre produsul vostru"
  }'
```

În n8n, ar trebui să vezi datele apărând în panoul **Data** al nodului Webhook.

### Pasul 4: Adaugă Node de Procesare (Edit Fields / Set)

1. Click pe mânerul din dreapta nodului Webhook (buline mici)
2. Caută **"Edit Fields"** sau **"Set"**
3. Adaugă câmpuri noi:
   - Click **"Add Field"** → **String**
   - Name: `processedAt`
   - Value: `{{ $now.toISO() }}`
   
   - Click **"Add Field"** → **String**
   - Name: `subject`
   - Value: `Mesaj nou de la {{ $json.name }}`
   
   - Click **"Add Field"** → **String**
   - Name: `fullMessage`
   - Value:
   ```
   Salut,

   Ai primit un mesaj nou:
   
   De la: {{ $json.name }}
   Email: {{ $json.email }}
   Mesaj: {{ $json.message }}
   
   Data primirii: {{ $json.processedAt }}
   ```

### Pasul 5: Adaugă Node Email

1. Conectează nodul Set la un nou nod
2. Caută **"Send Email"**
3. Configurare:
   - Click **"Create New Credential"**
   - Alege tipul (Gmail OAuth2 sau SMTP)

#### Pentru Gmail:

1. Selectează **"Google OAuth2 API"**
2. **Client ID** și **Client Secret**: Obține de la [Google Cloud Console](https://console.cloud.google.com)
   - Creează project nou
   - Activează Gmail API
   - Credentials → OAuth 2.0 Client IDs
   - Authorized redirect URIs: `http://localhost:5678/rest/oauth2-credential/callback`
3. Click **"Connect my account"**
4. Autorizează access

#### Pentru SMTP Generic:

1. Selectează **"SMTP"**
2. Configurare:
   ```
   Host: smtp.gmail.com  (sau alt SMTP)
   Port: 587
   User: tau-email@gmail.com
   Password: parola-app-sau-parola (Gmail: folosește App Password)
   Secure: TLS
   ```

#### Completează Email Node:

- **From Email**: `tau-email@gmail.com`
- **To Email**: `{{ $json.email }}` (emailul din webhook)
- **Subject**: `{{ $json.subject }}`
- **Email Type**: Text
- **Text**: `{{ $json.fullMessage }}`

### Pasul 6: Testează Workflow Complet

1. Click **"Test Workflow"** în footer
2. Trimite din nou request cu curl (vezi Pasul 3)
3. Observă execuția în timp real:
   - Webhook se colorează verde → date primite
   - Set se colorează verde → date procesate
   - Email se colorează verde → email trimis

4. Verifică inbox-ul pentru email

### Pasul 7: Salvează și Activează

1. Click **"Save"** (Ctrl/Cmd + S)
2. Activează toggle-ul **"Active"** din stânga jos
3. Webhook-ul tău URL permanent:
   ```
   http://localhost:5678/webhook/abc123def456
   ```

**Notă**: URL-ul se schimbă de la `/webhook-test/` la `/webhook/` când activezi workflow-ul.

### Pasul 8: Test Final

```bash
# Folosește noul URL permanent
curl -X POST http://localhost:5678/webhook/abc123def456 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Maria Ionescu",
    "email": "maria@example.com",
    "message": "Test workflow activ"
  }'
```

🎉 **Felicitări!** Ai creat primul tău workflow n8n funcțional!

## Lucrul cu Noduri

### Tipuri de Noduri

#### Trigger Nodes (Pornire Workflow)

##### 1. Webhook
Primește HTTP requests instant.

```yaml
Configurare:
  HTTP Method: POST/GET/PUT/DELETE/PATCH
  Path: /custom-path (opțional)
  Authentication: None/Basic/Header
  Response: 
    - Respond Immediately
    - Respond When Last Node Finishes
    - Respond Using Webhook Response Node
```

**Use Cases:**
- Formular contact de pe website
- Webhook-uri externe (GitHub, Stripe, etc.)
- API endpoints custom

##### 2. Schedule (Cron)
Rulează la intervale regulate.

```yaml
Trigger Rules:
  - Every 5 minutes
  - Every hour at :15
  - Daily at 9:00 AM
  - Custom Cron: */15 * * * * (fiecare 15 min)
```

**Use Cases:**
- Backup-uri periodice
- Sincronizare date
- Rapoarte zilnice
- Monitorizare status

##### 3. Email Trigger (IMAP)
Monitorizează inbox pentru email-uri noi.

```yaml
Configurare:
  IMAP Host: imap.gmail.com
  Port: 993
  SSL/TLS: true
  Mailbox: INBOX
  Criteria:
    - UNSEEN (necitite)
    - FROM: email@example.com
    - SUBJECT: "keyword"
```

**Use Cases:**
- Procesare comenzi prin email
- Ticketing system
- Automated replies

##### 4. Execute Workflow Trigger
Apelat de alt workflow.

```yaml
Configurare:
  - Minimă configurare
  - Primește date de la workflow apelant
```

**Use Cases:**
- Workflow modular
- Sub-workflows refolosibile
- Complex workflow orchestration

#### Action Nodes

##### HTTP Request
Apelează orice API REST.

```javascript
Configurare:
  Method: GET/POST/PUT/PATCH/DELETE
  URL: https://api.example.com/endpoint
  Authentication:
    - None
    - Basic Auth
    - Header Auth
    - OAuth2
  Headers:
    Content-Type: application/json
    Authorization: Bearer {{ $credentials.apiKey }}
  Body (pentru POST/PUT):
    {
      "data": "{{ $json.inputData }}"
    }
```

**Exemplu: Apel API REST**

```yaml
Method: POST
URL: https://api.openai.com/v1/chat/completions
Authentication: Header Auth
  Name: Authorization
  Value: Bearer sk-...
Body:
{
  "model": "gpt-4",
  "messages": [
    {
      "role": "user",
      "content": "{{ $json.question }}"
    }
  ]
}
```

##### Send Email

```yaml
From: tau@example.com
To: {{ $json.recipientEmail }}
CC: optional@example.com
Subject: {{ $json.emailSubject }}
Email Type: 
  - Text
  - HTML
Text/HTML:
  Salut {{ $json.name }},
  
  {{ $json.messageBody }}
  
  Cu respect,
  Echipa Ta
Attachments:
  - binary:{{ $binary.file }}
```

#### Core Nodes (Procesare și Control)

##### IF Node (Conditional Logic)

```yaml
Conditii:
  Condition 1:
    - {{ $json.amount }} > 1000
  Condition 2:
    - {{ $json.status }} === "approved"
    - AND {{ $json.verified }} === true

Outputs:
  - true → Ramura True
  - false → Ramura False
```

**Exemplu:**

```
Webhook → IF Node → [True] → Send Email "Approved"
                   → [False] → Send Email "Rejected"
```

##### Switch Node (Multiple Branches)

```yaml
Mode: Rules / Expression

Rules Mode:
  Rule 1: {{ $json.type }} === "order"    → Output 1
  Rule 2: {{ $json.type }} === "refund"   → Output 2
  Rule 3: {{ $json.type }} === "inquiry"  → Output 3
  Fallback: Output 4
```

**Exemplu:**

```
Webhook → Switch → [order] → Process Order
                 → [refund] → Process Refund
                 → [inquiry] → Forward to Support
                 → [fallback] → Log Unknown
```

##### Merge Node

Combină date din două flow-uri diferite.

```yaml
Mode:
  - Append: Combină toate items
  - Keep Key Matches: Merge by key
  - Merge By Position: Merge by index
  - Multiplex: Combine pairwise
```

**Exemplu:**

```
├─ Get User Data (DB) ─┐
│                       ├→ Merge → Process
└─ Get Orders (API) ───┘
```

##### Split Out Node

Împarte un item în multiple items (opus lui Merge).

```yaml
Split Out: {{ $json.items }}

Input:
{
  "order": "12345",
  "items": [
    {"name": "Product A", "qty": 2},
    {"name": "Product B", "qty": 1}
  ]
}

Output: 2 items separate
Item 1: {"name": "Product A", "qty": 2}
Item 2: {"name": "Product B", "qty": 1}
```

##### Code Node (JavaScript)

```javascript
// Acces la date
for (const item of $input.all()) {
  // Modifică item curent
  item.json.processed = true;
  item.json.timestamp = new Date().toISOString();
  
  // Calcule
  item.json.total = item.json.price * item.json.quantity;
  
  // Transformări
  item.json.email = item.json.email.toLowerCase();
}

// Return toate items
return $input.all();
```

**Funcții Disponibile:**

```javascript
// Acces items
$input.all()           // Toate items
$input.first()         // Primul item
$input.last()          // Ultimul item
$input.item            // Item curent (în loop)

// Helpers
$json                  // Date JSON item curent
$binary                // Date binare
$node                  // Acces date alte noduri
$workflow              // Info workflow
$execution             // Info execuție

// Node Methods
$('Node Name').all()   // Toate items din nod specific
$('Node Name').first() // Primul item
```

##### Edit Fields (Set) Node

```yaml
Add Field:
  
  # String
  Field: fullName
  Value: {{ $json.firstName }} {{ $json.lastName }}
  
  # Number
  Field: totalPrice
  Value: {{ $json.price * 1.19 }}  # TVA 19%
  
  # Boolean
  Field: isPremium
  Value: {{ $json.subscription === 'pro' }}
  
  # Date
  Field: processedAt
  Value: {{ $now.toISO() }}
  
  # Object (JSON)
  Field: metadata
  Value: { "source": "webhook", "ip": "{{ $json.ip }}" }
  
  # Array
  Field: tags
  Value: {{ ["new", "unprocessed"] }}
```

### Node Settings Avansate

Fiecare nod are un meniu **Settings** (iconița gear):

```yaml
Settings:
  
  # Notes & Display
  Notes: Descriere ce face nodul
  Display Name: Nume custom pentru nod
  
  # Error Handling
  Continue On Fail: true/false
    - true: Continuă workflow-ul la eroare
    - false: Oprește workflow-ul la eroare
  
  # Retry Logic
  Retry On Fail: true/false
  Max Tries: 3
  Wait Between Tries: 1000ms
  
  # Execution
  Always Output Data: true/false
  Execute Once: true/false (rulează o singură dată, nu pentru fiecare item)
  
  # Timeout
  Timeout: 30000ms (30 secunde)
```

## Manipularea Datelor

### Înțelegerea Structurii de Date

n8n lucrează cu **items** - fiecare item este un obiect JSON.

```json
[
  {
    "json": {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "orders": [
        {"id": "A1", "total": 100},
        {"id": "A2", "total": 250}
      ]
    },
    "binary": {}
  },
  {
    "json": {
      "id": 2,
      "name": "Jane Smith",
      "email": "jane@example.com"
    },
    "binary": {}
  }
]
```

### Expressions și Variabile

#### Sintaxă de Bază

```javascript
// Acces date simple
{{ $json.fieldName }}
{{ $json.email }}

// Acces date nested
{{ $json.user.name }}
{{ $json.address.city }}

// Acces array
{{ $json.items[0].name }}      // Primul element
{{ $json.items[1].price }}     // Al doilea element

// Acces toate elementele array (pentru Split Out)
{{ $json.items }}
```

#### Variabile Sistem

```javascript
// Timp
{{ $now }}                        // Timestamp curent (Luxon DateTime)
{{ $now.toISO() }}               // ISO string: 2024-01-15T10:30:00.000Z
{{ $now.toFormat('yyyy-MM-dd') }} // 2024-01-15
{{ $today }}                      // Data curentă fără oră

// Workflow info
{{ $workflow.id }}                // ID workflow
{{ $workflow.name }}              // Nume workflow
{{ $workflow.active }}            // true/false

// Execution info
{{ $execution.id }}               // ID execuție
{{ $execution.mode }}             // manual/trigger/webhook/etc.
{{ $execution.resumeUrl }}        // URL pentru resume (wait nodes)

// Item info
{{ $itemIndex }}                  // Index item curent (0, 1, 2...)
{{ $nodeId }}                     // ID nod curent

// Environment (dacă setat în docker-compose)
{{ $env.API_KEY }}
{{ $env.DATABASE_URL }}
```

#### Acces Date din Alte Noduri

```javascript
// Nodul anterior (relativ)
{{ $json.field }}

// Nod specific (absolut)
{{ $('Node Name').item.json.field }}
{{ $('Webhook').item.json.email }}
{{ $('HTTP Request').item.json.response.data }}

// Toate items dintr-un nod
{{ $('Node Name').all() }}        // Array cu toate items

// Primul/Ultimul item
{{ $('Node Name').first() }}
{{ $('Node Name').last() }}

// Item la index specific
{{ $('Node Name').all()[0].json.field }}
```

### Funcții pentru Manipulare Date

#### String Functions

```javascript
// Upper/Lower case
{{ $json.email.toLowerCase() }}              // john@example.com
{{ $json.name.toUpperCase() }}               // JOHN DOE

// Trim
{{ $json.text.trim() }}                      // Elimină spații

// Substring
{{ $json.text.substring(0, 10) }}            // Primele 10 caractere
{{ $json.text.slice(0, -3) }}                // Fără ultimele 3

// Replace
{{ $json.text.replace('old', 'new') }}       // Înlocuire
{{ $json.text.replaceAll('old', 'new') }}    // Toate apariții

// Split
{{ $json.text.split(',') }}                  // Split la virgulă
{{ $json.text.split(' ')[0] }}               // Primul cuvânt

// Includes
{{ $json.email.includes('@gmail.com') }}     // true/false

// Template strings
{{ `Hello ${$json.name}!` }}                 // Hello John!
{{ `Total: ${$json.price * $json.qty} RON` }}
```

#### Number Functions

```javascript
// Operații matematice
{{ $json.price + $json.tax }}                // Adunare
{{ $json.price * 1.19 }}                     // Înmulțire (TVA)
{{ $json.total - $json.discount }}           // Scădere
{{ $json.price / $json.quantity }}           // Împărțire
{{ $json.quantity % 10 }}                    // Modulo (rest)

// Rotunjire
{{ Math.round($json.value) }}                // Rotunjire standard
{{ Math.floor($json.value) }}                // Rotunjire jos
{{ Math.ceil($json.value) }}                 // Rotunjire sus
{{ $json.price.toFixed(2) }}                 // 2 zecimale: "19.99"

// Min/Max
{{ Math.min($json.a, $json.b, $json.c) }}
{{ Math.max($json.a, $json.b, $json.c) }}

// Parse
{{ parseInt($json.stringNumber) }}           // String → Int
{{ parseFloat($json.stringDecimal) }}        // String → Float
```

#### Date/Time Functions (Luxon)

```javascript
// Format date
{{ $now.toFormat('dd/MM/yyyy') }}            // 15/01/2024
{{ $now.toFormat('HH:mm:ss') }}              // 14:30:45
{{ $now.toFormat('MMMM dd, yyyy') }}         // January 15, 2024
{{ $now.toFormat('dd.MM.yyyy HH:mm') }}      // 15.01.2024 14:30

// ISO format
{{ $now.toISO() }}                           // 2024-01-15T14:30:00.000+02:00
{{ $now.toUTC().toISO() }}                   // UTC time

// Operații date
{{ $now.plus({ days: 7 }) }}                 // +7 zile
{{ $now.minus({ hours: 2 }) }}               // -2 ore
{{ $now.plus({ months: 1, days: 5 }) }}      // +1 lună +5 zile

// Parse din string
{{ DateTime.fromISO($json.dateString) }}
{{ DateTime.fromFormat($json.date, 'dd/MM/yyyy') }}

// Diferență între date
{{ $now.diff(DateTime.fromISO($json.startDate), 'days').days }}
```

#### Array Functions

```javascript
// Length
{{ $json.items.length }}

// Accesare
{{ $json.items[0] }}                         // Primul
{{ $json.items[$json.items.length - 1] }}    // Ultimul

// Join
{{ $json.tags.join(', ') }}                  // "tag1, tag2, tag3"

// Map (în Code node)
$json.items.map(item => item.name)

// Filter (în Code node)
$json.items.filter(item => item.price > 100)

// Find
$json.items.find(item => item.id === 'ABC')

// Some/Every
$json.items.some(item => item.active)        // true dacă măcar unul
$json.items.every(item => item.verified)     // true dacă toate
```

#### Object Functions

```javascript
// Keys/Values
{{ Object.keys($json) }}                     // ["name", "email", ...]
{{ Object.values($json) }}                   // Toate valorile

// Verificare existență
{{ $json.hasOwnProperty('email') }}          // true/false
{{ 'email' in $json }}                       // true/false

// Merge objects (în Code node)
Object.assign({}, $json, { newField: 'value' })
{ ...$json, newField: 'value' }              // Spread operator
```

#### Conditional Logic

```javascript
// Ternary operator
{{ $json.status === 'active' ? 'Activ' : 'Inactiv' }}
{{ $json.price > 100 ? 'Scump' : 'Ieftin' }}

// Nullish coalescing
{{ $json.optionalField ?? 'default value' }}

// Logical operators
{{ $json.verified && $json.active }}         // AND
{{ $json.status === 'new' || $json.status === 'pending' }}  // OR
{{ !$json.deleted }}                         // NOT

// If în template
{{ $json.isPremium ? `Premium: ${$json.name}` : $json.name }}
```

### Code Node - JavaScript Avansat

#### Template Complet

```javascript
// ============================================
// Procesare toate items
// ============================================

// Iterează prin toate items
for (const item of $input.all()) {
  
  // 1. Validare date
  if (!item.json.email || !item.json.email.includes('@')) {
    item.json.valid = false;
    item.json.error = 'Invalid email';
    continue;
  }
  
  // 2. Transformări
  item.json.email = item.json.email.toLowerCase().trim();
  item.json.name = item.json.firstName + ' ' + item.json.lastName;
  
  // 3. Calcule
  item.json.totalPrice = item.json.price * item.json.quantity;
  item.json.priceWithVAT = item.json.totalPrice * 1.19;
  
  // 4. Date și timp
  item.json.processedAt = new Date().toISOString();
  item.json.expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString();
  
  // 5. Logică condițională
  if (item.json.totalPrice > 1000) {
    item.json.discount = 0.1;  // 10% discount
    item.json.finalPrice = item.json.priceWithVAT * 0.9;
  } else {
    item.json.discount = 0;
    item.json.finalPrice = item.json.priceWithVAT;
  }
  
  // 6. Array manipulation
  item.json.tags = item.json.tags || [];
  item.json.tags.push('processed');
  
  if (item.json.totalPrice > 500) {
    item.json.tags.push('high-value');
  }
  
  // 7. Object manipulation
  item.json.metadata = {
    source: 'api',
    processed: true,
    version: '2.0'
  };
  
  // 8. Acces date din alte noduri
  const webhookData = $('Webhook').first().json;
  item.json.originalSource = webhookData.source;
  
  // 9. Validare finală
  item.json.valid = true;
}

// Return toate items procesate
return $input.all();
```

#### Exemple Specifice

**Filtrare Items:**

```javascript
// Păstrează doar items care îndeplinesc condiția
return $input.all().filter(item => {
  return item.json.status === 'active' && item.json.amount > 100;
});
```

**Transformare Items:**

```javascript
// Transformă fiecare item
return $input.all().map(item => {
  return {
    json: {
      id: item.json.id,
      fullName: `${item.json.firstName} ${item.json.lastName}`,
      email: item.json.email.toLowerCase(),
      registered: new Date(item.json.createdAt).toLocaleDateString('ro-RO')
    }
  };
});
```

**Agregare Date:**

```javascript
// Calculează total dintr-un array
const total = $input.all().reduce((sum, item) => {
  return sum + item.json.amount;
}, 0);

// Return single item cu rezultat
return [{
  json: {
    totalAmount: total,
    itemCount: $input.all().length,
    averageAmount: total / $input.all().length
  }
}];
```

**API Calls din Code:**

```javascript
// Apel HTTP cu fetch
const items = [];

for (const item of $input.all()) {
  const response = await fetch('https://api.example.com/data', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${item.json.apiKey}`
    },
    body: JSON.stringify({
      id: item.json.id,
      data: item.json.data
    })
  });
  
  const result = await response.json();
  
  items.push({
    json: {
      ...item.json,
      apiResponse: result
    }
  });
}

return items;
```

**Error Handling:**

```javascript
const results = [];

for (const item of $input.all()) {
  try {
    // Procesare care poate genera eroare
    const processed = riskyOperation(item.json.data);
    
    results.push({
      json: {
        ...item.json,
        processed: processed,
        success: true
      }
    });
  } catch (error) {
    // Capturează eroarea și continuă
    results.push({
      json: {
        ...item.json,
        error: error.message,
        success: false
      }
    });
  }
}

return results;
```

## Funcții Avansate

### Error Workflows

Creează workflow-uri dedicate pentru gestionarea erorilor.

#### Setup Error Workflow

**Pasul 1: Creează Error Workflow**

```
Workflow nou: "Error Handler"

Noduri:
1. Error Trigger
2. Edit Fields (construiește mesaj)
3. Send Email / Slack
```

**Error Trigger Node:**

```yaml
Configurare minimă:
  - Se declanșează automat când orice workflow are eroare
  - Primește date despre eroare:
    - $json.workflow (workflow cu eroare)
    - $json.execution (detalii execuție)
    - $json.error (mesaj eroare)
```

**Edit Fields - Construiește Mesaj:**

```javascript
Subject: 🚨 Eroare în Workflow: {{ $json.workflow.name }}

Body:
Workflow: {{ $json.workflow.name }}
Execution ID: {{ $json.execution.id }}
Data/Ora: {{ $json.execution.startedAt }}

Eroare:
{{ $json.error.message }}

Stack Trace:
{{ $json.error.stack }}

Node cu eroare: {{ $json.error.node.name }}

Link execuție:
http://localhost:5678/execution/{{ $json.execution.id }}
```

**Send Email/Slack:**

Trimite notificarea către tine sau echipă.

**Pasul 2: Activează Error Workflow în Alt Workflow**

```
Workflow original:
  Settings (gear icon sus dreapta) →
  Error Workflow →
  Selectează: "Error Handler"
```

Acum, orice eroare în workflow-ul original va declanșa Error Handler.

### Wait Node

Pauzează execuția workflow-ului până la un eveniment.

```yaml
Wait Node Options:

1. After Time Delay:
   - 5 seconds
   - 10 minutes
   - 1 hour
   - Custom: 2 days, 3 hours, 15 minutes

2. At Specified Time:
   - La o dată/oră specifică
   - Exemplu: 2024-12-25 09:00:00

3. On Webhook Call:
   - Așteaptă până primește un webhook
   - Generează resume URL
   - Folosit pentru: human approval, async API callbacks

4. On Form Submission:
   - Afișează formular web
   - Așteaptă completare
   - Continuă cu datele din formular
```

**Exemplu: Approval Workflow**

```
Webhook (primește request)
  ↓
Send Email cu link aprobare
  ↓
Wait (webhook call)  ← Resume URL în email
  ↓
[Approved] → Procesează
[Rejected] → Notifică
```

### Sticky Note

Organizează și documentează workflow-uri complexe.

```
Right-click pe Canvas → Add Sticky Note

Uses:
- Documentație: Explică secțiuni complexe
- TODOs: Ce trebuie îmbunătățit
- Warnings: Chestii importante de știut
- Structură: Grupează noduri logic
```

**Best Practice:**

```
┌──────────────────────────────────────────┐
│  📝 VALIDARE INPUT                       │
│  Verifică și curăță datele primite      │
│  din webhook înainte de procesare        │
└──────────────────────────────────────────┘
    │
    ↓
  [Noduri validare]
```

### Sub-Workflows (Execute Workflow)

Modulează workflow-uri mari în componente refolosibile.

**Main Workflow:**

```
Webhook
  ↓
Execute Workflow: "Process User"
  ↓
Execute Workflow: "Send Notifications"
  ↓
Response
```

**Sub-Workflow: "Process User"**

```
Execute Workflow Trigger
  ↓
Validate Data
  ↓
Database Insert
  ↓
Return processed data
```

**Execute Workflow Node Configuration:**

```yaml
Source: Database / Current Project
Workflow: Select "Process User"

Pass Data:
  Input Data: {{ $json }}
  
Wait for Workflow to Complete: Yes/No
```

### Looping și Batch Processing

#### Loop Over Items Node

```yaml
Mode: 
  - Run Once for All Items: Procesează toate odată
  - Run Once for Each Item: Loop prin fiecare

Uses:
  Când trebuie să procesezi items secvențial
  cu dependențe între iterații
```

#### Split in Batches Node

```yaml
Batch Size: 10
Options:
  - Reset: După finalizare
```

**Exemplu: Procesează 1000 înregistrări în batches de 10:**

```
Get 1000 Records
  ↓
Split in Batches (10)
  ↓
Process Batch (rulează 100 ori)
  ↓
Loop back până totul e procesat
```

## AI și Agenți

### OpenAI Node

Integrare directă cu GPT-4, DALL-E, Whisper, embeddings.

#### Chat Completion (GPT-4)

```yaml
Credential: OpenAI API Key
Model: gpt-4-turbo / gpt-3.5-turbo
Operation: Message a Model

Messages:
  - Role: System
    Content: "Ești un asistent util care răspunde în română."
  
  - Role: User
    Content: {{ $json.question }}

Options:
  Temperature: 0.7 (0 = deterministă, 1 = creativă)
  Max Tokens: 500
  Top P: 1
```

**Exemplu Workflow: Q&A Bot**

```
Webhook (primește întrebare)
  ↓
OpenAI Chat (generează răspuns)
  ↓
Edit Fields (formatează)
  ↓
Response Webhook / Send Email
```

#### Image Generation (DALL-E)

```yaml
Operation: Create Image
Prompt: {{ $json.imageDescription }}
Size: 1024x1024 / 1792x1024 / 1024x1792
Quality: standard / hd
Style: vivid / natural
Number of Images: 1
```

#### Embeddings

```yaml
Operation: Create Embeddings
Model: text-embedding-3-small
Input: {{ $json.text }}

Output: Array de numere (vector)
Use: Similarity search, RAG systems
```

### AI Agent Node

Agent autonom care poate folosi tools pentru a rezolva sarcini complexe.

```yaml
Agent:
  Type: 
    - Conversational Agent
    - OpenAI Functions Agent
    - ReAct Agent
  
  Model: OpenAI GPT-4
  
  Prompt:
    """
    Ești un agent util care ajută utilizatorii cu sarcinile lor.
    Ai acces la următoarele tools:
    - Calculator pentru calcule matematice
    - HTTP Request pentru API calls
    - Database pentru queries
    
    Folosește aceste tools când este necesar.
    """

Tools (pot fi atașate):
  - Calculator
  - HTTP Request (definit ca tool)
  - Custom Code tool
  - Database query tool
  - Vector Store search

Memory (opțional):
  - Conversational Buffer Memory
  - Session-based memory pentru context

Output Parser:
  - JSON
  - Structured Output
```

**Exemplu: Research Agent**

```
1. Webhook (primește topic)
2. AI Agent
   - Tool 1: HTTP Request (caută pe web)
   - Tool 2: Summarize cu OpenAI
   - Tool 3: Save to Database
3. Return structured rezultat
```

### Vector Store

Stochează embeddings pentru semantic search.

```yaml
Vector Stores suportate:
  - Pinecone
  - Supabase (pgvector)
  - Qdrant
  - Weaviate
  - In-Memory Vector Store (pentru dev)

Operations:
  1. Insert: Adaugă documente cu embeddings
  2. Retrieve: Caută similar documents
  3. Delete: Șterge documente

Insert Workflow:
  Get Documents
    ↓
  OpenAI Embeddings (generează vectors)
    ↓
  Vector Store Insert

Retrieve Workflow:
  User Query
    ↓
  OpenAI Embeddings (query vector)
    ↓
  Vector Store Retrieve (similarity search)
    ↓
  OpenAI Chat (answer cu context)
```

### RAG (Retrieval Augmented Generation)

Combină search cu generation pentru răspunsuri bazate pe date proprii.

**Architecture:**

```
User Question
  ↓
Generate Query Embedding (OpenAI)
  ↓
Search Vector Store (top 5 similar docs)
  ↓
Build Context (concatenează docs)
  ↓
OpenAI Chat cu Context
  ↓
Return Answer
```

**Exemplu Workflow Complet:**

```yaml
Workflow: "RAG Q&A System"

1. Webhook (primește întrebare)

2. OpenAI Embeddings
   Input: {{ $json.question }}
   Output: query_vector

3. Pinecone Retrieve
   Vector: {{ $json.query_vector }}
   Top K: 5
   Output: similar_docs

4. Code Node (construiește context)
   const context = $('Pinecone').all()
     .map(item => item.json.text)
     .join('\n\n');
   
   return [{
     json: {
       question: $('Webhook').json.question,
       context: context
     }
   }];

5. OpenAI Chat
   System Prompt:
   """
   Răspunde la întrebarea utilizatorului bazându-te DOAR
   pe contextul furnizat mai jos. Dacă răspunsul nu se
   găsește în context, spune că nu știi.
   
   Context:
   {{ $json.context }}
   """
   
   User Prompt:
   {{ $json.question }}

6. Response
```

## Best Practices

### 1. Organizare Workflow-uri

```
Naming Convention:
  ✅ Bine: "CRM - Sync Contacts to HubSpot"
  ❌ Rău: "workflow 1"

Structură:
  [Category] - [Action] - [Destination]
  
  Examples:
  - Email - Parse Attachments - Save to Drive
  - API - Sync Orders - Shopify to Database
  - Backup - Daily Database - PostgreSQL to S3

Folosește Sticky Notes:
  - Grupează noduri logic
  - Documentează decizii importante
  - Explică logică complexă

Tags/Folders (dacă n8n cloud):
  - Production
  - Development
  - Testing
  - Archived
```

### 2. Error Handling

```yaml
La Nivel de Nod:
  Settings → Continue On Fail: true
  Settings → Retry On Fail: true
  Settings → Max Tries: 3

La Nivel de Workflow:
  Settings → Error Workflow: "Global Error Handler"

În Code Nodes:
  try {
    // risky operation
  } catch (error) {
    item.json.error = error.message;
    item.json.failed = true;
  }

Logging:
  Trimite erori la:
  - Slack channel #errors
  - Email către echipă
  - Logging service (Sentry, LogRocket)
```

### 3. Securitate

```yaml
Credentials:
  ✅ Folosește Credentials Manager
  ❌ Nu hardcoda API keys în noduri

  Credentials Types:
  - OAuth2 (recomandat pentru Google, etc.)
  - API Key
  - Header Auth
  - Basic Auth

Environment Variables:
  În docker-compose.yml:
    environment:
      - API_KEY=${API_KEY}
      - DB_PASSWORD=${DB_PASSWORD}
  
  În workflow:
    {{ $env.API_KEY }}

SSL/HTTPS:
  Activează pentru producție:
  - N8N_PROTOCOL=https
  - Folosește nginx cu Let's Encrypt
  - Forțează HTTPS pentru webhooks

Backup Credentials:
  # Export credentials (encrypted)
  docker-compose exec n8n n8n export:credentials \
    --all --output=/home/node/backups/creds.json
```

### 4. Performance

```yaml
Batch Processing:
  ✅ Procesează 100 items odată
  ❌ 100 HTTP requests separate

  Use: Split in Batches node
  Batch size: 10-50 items

Database Queries:
  ✅ Single query cu WHERE IN (ids)
  ❌ Loop cu query per item

HTTP Requests:
  - Setează timeout rezonabil (30s)
  - Retry cu exponential backoff
  - Rate limiting dacă API cere

Caching:
  - Folosește n8n variables pentru date temp
  - PostgreSQL pentru cache persistent
  - Redis pentru high-performance cache

Async Operations:
  Workflows grele:
  - Split în sub-workflows
  - Execute Workflow async (don't wait)
  - Procesează în background
```

### 5. Testing

```yaml
Development:
  1. Test Workflow button
  2. Verifică fiecare nod individual
  3. Testează cu date reale
  4. Testează edge cases

Staging:
  - Duplicate workflow pentru testing
  - Folosește date de test
  - Webhook URLs separate pentru test

Production:
  - Monitor Execution History
  - Setup alerts pentru errors
  - Test după fiecare deploy
  - Rollback plan

Test Data:
  Webhook test payload:
  {
    "test": true,
    "data": {...}
  }
  
  IF node:
  {{ $json.test !== true }} → Production branch
```

### 6. Monitoring

```yaml
Built-in:
  - Executions History
  - Filter: Errors only
  - Search by date/status

Error Workflows:
  Main Workflow → Settings → Error Workflow

  Error Workflow:
    Error Trigger
      ↓
    Edit Fields (format error message)
      ↓
    Slack/Email notification

External Monitoring:
  - Uptime Robot (ping webhooks)
  - Healthchecks.io (cron monitoring)
  - Custom monitoring endpoint:
  
    Schedule (every 5min)
      ↓
    HTTP Request (check critical workflows)
      ↓
    IF (failed) → Alert

Metrics:
  Track în Database:
  - Execution time
  - Success/fail rate
  - Items processed
  - API costs
```

## Backup și Mentenanță

### Backup Automat

#### Script Bash

```bash
#!/bin/bash
# backup-n8n.sh

# Configurare
BACKUP_DIR="/path/to/backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Creează director backup
mkdir -p ${BACKUP_DIR}

echo "Starting n8n backup: ${DATE}"

# 1. Export Workflows
echo "Exporting workflows..."
docker-compose exec -T n8n n8n export:workflow \
  --all \
  --output=/home/node/backups/workflows_${DATE}.json

# 2. Export Credentials (encrypted)
echo "Exporting credentials..."
docker-compose exec -T n8n n8n export:credentials \
  --all \
  --output=/home/node/backups/credentials_${DATE}.json

# 3. Backup PostgreSQL Database
echo "Backing up database..."
docker-compose exec -T postgres pg_dump -U n8n n8n \
  > ${BACKUP_DIR}/db_${DATE}.sql

# 4. Backup n8n data volume
echo "Backing up n8n data..."
docker run --rm \
  --volumes-from n8n \
  -v ${BACKUP_DIR}:/backup \
  alpine \
  tar czf /backup/n8n_data_${DATE}.tar.gz /home/node/.n8n

# 5. Copiază backups din container
docker cp n8n:/home/node/backups/workflows_${DATE}.json ${BACKUP_DIR}/
docker cp n8n:/home/node/backups/credentials_${DATE}.json ${BACKUP_DIR}/

# 6. Curăță backups vechi
echo "Cleaning old backups (older than ${RETENTION_DAYS} days)..."
find ${BACKUP_DIR} -name "*.json" -mtime +${RETENTION_DAYS} -delete
find ${BACKUP_DIR} -name "*.sql" -mtime +${RETENTION_DAYS} -delete
find ${BACKUP_DIR} -name "*.tar.gz" -mtime +${RETENTION_DAYS} -delete

echo "Backup completed: ${DATE}"
echo "Files created:"
ls -lh ${BACKUP_DIR}/*${DATE}*
```

#### Automatizare cu Cron

```bash
# Editează crontab
crontab -e

# Adaugă backup zilnic la 2 AM
0 2 * * * /path/to/backup-n8n.sh >> /var/log/n8n-backup.log 2>&1

# Sau cu n8n Schedule Node:
Schedule (Daily 2 AM)
  ↓
Execute Command (bash backup script)
  ↓
IF (success) → Send success email
IF (failed) → Send alert
```

### Restore din Backup

```bash
#!/bin/bash
# restore-n8n.sh

BACKUP_FILE=$1

if [ -z "$BACKUP_FILE" ]; then
  echo "Usage: ./restore-n8n.sh <backup_date>"
  echo "Example: ./restore-n8n.sh 20240115_020000"
  exit 1
fi

BACKUP_DIR="/path/to/backups"

echo "Stopping n8n..."
docker-compose stop n8n

# 1. Restore Database
echo "Restoring database..."
docker-compose exec -T postgres psql -U n8n -d n8n \
  < ${BACKUP_DIR}/db_${BACKUP_FILE}.sql

# 2. Restore Workflows
echo "Restoring workflows..."
docker cp ${BACKUP_DIR}/workflows_${BACKUP_FILE}.json n8n:/home/node/backups/
docker-compose exec n8n n8n import:workflow \
  --input=/home/node/backups/workflows_${BACKUP_FILE}.json

# 3. Restore Credentials
echo "Restoring credentials..."
docker cp ${BACKUP_DIR}/credentials_${BACKUP_FILE}.json n8n:/home/node/backups/
docker-compose exec n8n n8n import:credentials \
  --input=/home/node/backups/credentials_${BACKUP_FILE}.json

# 4. Restart n8n
echo "Restarting n8n..."
docker-compose start n8n

echo "Restore completed!"
```

### Update n8n

```bash
#!/bin/bash
# update-n8n.sh

echo "Backing up before update..."
./backup-n8n.sh

echo "Pulling latest n8n image..."
docker-compose pull n8n

echo "Stopping n8n..."
docker-compose down

echo "Starting with new image..."
docker-compose up -d

echo "Checking status..."
docker-compose ps

echo "Checking logs..."
docker-compose logs -f n8n
```

### Mentenanță PostgreSQL

```bash
# Conectare la PostgreSQL
docker-compose exec postgres psql -U n8n -d n8n

-- Verifică mărime database
SELECT pg_size_pretty(pg_database_size('n8n'));

-- Verifică mărime tabele
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Curăță execuții vechi (opțional)
DELETE FROM execution_entity
WHERE "startedAt" < NOW() - INTERVAL '30 days'
AND finished = true;

-- VACUUM pentru reclaim space
VACUUM ANALYZE;
```

### Monitoring Disk Space

```bash
# Verifică spațiu volumes
docker system df -v

# Curăță images nefolosite
docker image prune -a

# Curăță volumes nefolosite (ATENȚIE!)
docker volume prune

# Verifică mărime volume n8n
docker run --rm \
  --volumes-from n8n \
  alpine du -sh /home/node/.n8n
```

## Depanare

### Probleme Comune

#### 1. n8n nu pornește

**Simptom**: Container se oprește imediat după pornire.

```bash
# Verifică logs
docker-compose logs n8n

# Verifică dacă portul 5678 este disponibil
sudo netstat -tulpn | grep 5678
# sau
sudo lsof -i :5678

# Verifică resurse
docker stats

# Verifică permissions volume
ls -la | grep n8n_data
```

**Soluții**:

```bash
# Port ocupat - schimbă portul
# În docker-compose.yml:
ports:
  - "5679:5678"  # Folosește alt port

# Permissions - fix
docker-compose down
docker volume rm n8n_data
docker-compose up -d

# Dacă PostgreSQL nu e ready
# Adaugă în docker-compose.yml:
healthcheck pentru postgres
depends_on cu condition: service_healthy
```

#### 2. "Connection to database failed"

**Simptom**: n8n nu se poate conecta la PostgreSQL.

```bash
# Verifică dacă Postgres rulează
docker-compose ps postgres

# Verifică logs Postgres
docker-compose logs postgres

# Test conexiune manual
docker-compose exec postgres psql -U n8n -d n8n -c "SELECT 1"
```

**Soluții**:

```yaml
# Verifică credentials în docker-compose.yml
environment:
  - DB_POSTGRESDB_USER=n8n          # Trebuie să fie la fel
  - POSTGRES_USER=n8n               # în ambele servicii

  - DB_POSTGRESDB_PASSWORD=parola   # Trebuie să fie la fel
  - POSTGRES_PASSWORD=parola        # în ambele servicii

  - DB_POSTGRESDB_DATABASE=n8n      # Trebuie să fie la fel
  - POSTGRES_DB=n8n                 # în ambele servicii

# Adaugă health check pentru Postgres
healthcheck:
  test: ['CMD-SHELL', 'pg_isready -h localhost -U n8n -d n8n']
  interval: 5s
  timeout: 5s
  retries: 10

# Adaugă depends_on cu condition
depends_on:
  postgres:
    condition: service_healthy
```

#### 3. Webhooks nu funcționează

**Simptom**: Webhook URL nu primește requests.

```bash
# Test webhook cu curl
curl -X POST http://localhost:5678/webhook/your-path \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'

# Verifică că workflow este ACTIVE
# Verifică în Executions History dacă a primit request
```

**Soluții**:

```yaml
# Verifică WEBHOOK_URL în docker-compose.yml
environment:
  - WEBHOOK_URL=http://localhost:5678/

# Pentru producție cu domeniu:
  - WEBHOOK_URL=https://your-domain.com/

# Verifică firewall
sudo ufw status
sudo ufw allow 5678

# Pentru webhook extern (din internet):
# Folosește ngrok pentru testing:
ngrok http 5678
# Apoi folosește URL-ul ngrok în webhook calls
```

#### 4. Workflows nu se salvează

**Simptom**: Changes are lost după refresh sau restart.

```bash
# Verifică permissions volume
docker-compose down
sudo chown -R 1000:1000 ./workflows
docker-compose up -d

# Verifică dacă volume e mounted corect
docker inspect n8n | grep Mounts -A 20

# Verifică spațiu disk
df -h
```

#### 5. High CPU/Memory Usage

**Simptom**: n8n consumă multe resurse.

```bash
# Monitorizează resurse
docker stats n8n

# Verifică workflows active
# Probabil un loop infinit sau polling prea des

# Limitează resources în docker-compose.yml:
services:
  n8n:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          memory: 512M
```

**Debugging:**

```bash
# Verifică executions
# Dashboard → Executions
# Sortează by duration

# Oprește workflows suspicioase
# Verifică Schedule nodes cu interval prea mic
# Verifică loops fără exit condition
```

#### 6. Credentials nu funcționează

**Simptom**: "Invalid credentials" errors în noduri.

```bash
# Verifică N8N_ENCRYPTION_KEY
# Dacă schimbi encryption key, credentials existente nu mai funcționează

# Soluție:
# 1. Export credentials cu key-ul vechi
docker-compose exec n8n n8n export:credentials --all --output=/tmp/creds.json

# 2. Schimbă N8N_ENCRYPTION_KEY în docker-compose.yml

# 3. Restart
docker-compose restart n8n

# 4. Re-import credentials (vor fi re-encrypted cu new key)
docker-compose exec n8n n8n import:credentials --input=/tmp/creds.json

# SAU: Reconnect toate credentials manual în UI
```

#### 7. Node execution timeout

**Simptom**: "Execution timed out" errors.

```yaml
# Crește timeout la nivel de nod
Node Settings → Timeout: 60000 (60 secunde)

# Sau global în docker-compose.yml:
environment:
  - EXECUTIONS_TIMEOUT=300  # 5 minute
  - EXECUTIONS_TIMEOUT_MAX=600  # 10 minute max
```

### Debug Tools

#### Enable Debug Logging

```yaml
# În docker-compose.yml
environment:
  - N8N_LOG_LEVEL=debug
  - N8N_LOG_OUTPUT=console,file
  - N8N_LOG_FILE_LOCATION=/home/node/.n8n/logs/
```

```bash
# Vezi logs real-time
docker-compose logs -f n8n

# Logs din container
docker-compose exec n8n cat /home/node/.n8n/logs/n8n.log
```

#### Inspect Execution Data

```
Dashboard → Executions → Click pe execution

Vezi:
- Input/Output fiecărui nod
- Execution time per nod
- Errors și stack traces
- JSON data la fiecare pas
```

#### Test Individual Nodes

```
În Workflow Editor:
1. Selectează nodul
2. Click "Execute Node"
3. Vezi rezultatul imediat
4. Debug data flow
```

#### Browser DevTools

```
F12 → Console

Verifică:
- Network errors (failed API calls)
- JavaScript errors
- WebSocket connection status
```

### Helpful Commands

```bash
# Restart rapid
docker-compose restart n8n

# Rebuild complete (dacă ai schimbat docker-compose.yml)
docker-compose down
docker-compose up -d --build

# Logs ultimele 100 linii
docker-compose logs --tail=100 n8n

# Logs cu timestamps
docker-compose logs -t n8n

# Shell în container n8n
docker-compose exec n8n sh

# Shell în PostgreSQL
docker-compose exec postgres psql -U n8n -d n8n

# Verifică n8n version
docker-compose exec n8n n8n --version

# List workflows via CLI
docker-compose exec n8n n8n list:workflow

# Export specific workflow
docker-compose exec n8n n8n export:workflow --id=1 --output=/tmp/workflow.json

# Health check
curl http://localhost:5678/healthz
```

## Resurse și Suport

### Documentație Oficială

- 📚 [n8n Documentation](https://docs.n8n.io) - Documentație completă
- 🎓 [n8n Academy](https://docs.n8n.io/courses/) - Cursuri video
- 📘 [API Reference](https://docs.n8n.io/api/) - API documentation
- 🔧 [Node Reference](https://docs.n8n.io/integrations/) - Toate nodurile

### Comunitate

- 💬 [n8n Community Forum](https://community.n8n.io) - Forum oficial
- 💬 [Discord Server](https://discord.gg/n8n) - Chat real-time
- 🐙 [GitHub](https://github.com/n8n-io/n8n) - Issues, PRs, cod sursă
- 🐦 [Twitter @n8n_io](https://twitter.com/n8n_io) - Updates

### Video Tutorials

#### Canale YouTube Recomandate:

1. **[n8n Official](https://www.youtube.com/@n8n-io)**
   - Tutorial-uri oficiale
   - Feature announcements
   - Use cases

2. **[Nate Herk - AI Automation](https://www.youtube.com/@nateherk)**
   - AI workflows
   - Complex automations
   - Best practices

3. **[Workflow Automation](https://www.youtube.com/@workflowautomation)**
   - Tutorial-uri detaliate
   - Integration guides

### Template Libraries

- 🌐 [n8n Workflow Templates](https://n8n.io/workflows) - Official templates
- 🌐 [Community Workflows](https://community.n8n.io/c/workflows/10) - User shared

### Tools Utile

- 🔧 [Webhook.site](https://webhook.site) - Test webhooks
- 🔧 [JSONLint](https://jsonlint.com) - Validate JSON
- 🔧 [Crontab Guru](https://crontab.guru) - Cron expression tester
- 🔧 [Regex101](https://regex101.com) - Test regex
- 🔧 [JWT.io](https://jwt.io) - Decode JWT tokens
- 🔧 [Base64 Encode/Decode](https://www.base64encode.org)

### Suport Comercial

- 💼 [n8n Enterprise](https://n8n.io/enterprise) - Enterprise support
- 💼 [n8n Partners](https://n8n.io/partners) - Certified consultants

---

**Automatizare cu Succes! 🚀**

*Self-hosted. Open-source. Under your control.*

Pentru întrebări: [community.n8n.io](https://community.n8n.io)


