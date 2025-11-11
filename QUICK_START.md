# 🚀 Quick Start - n8n în 10 Minute

Ghid super rapid pentru a instala și rula n8n self-hosted cu Docker pe calculatorul tău.

## ⏱️ Timp necesar: 10-15 minute

## Cerințe

- ✅ Docker și Docker Compose instalat
- ✅ 2GB RAM disponibil
- ✅ 5GB spațiu disk
- ✅ Terminal/Command Prompt

## Pas 1: Verifică Docker (1 minut)

```bash
# Verifică că Docker este instalat
docker --version
docker-compose --version

# Dacă nu ai Docker, instalează:
# Windows/Mac: https://www.docker.com/products/docker-desktop/
# Linux: sudo apt install docker.io docker-compose
```

## Pas 2: Creează Structura (1 minut)

```bash
# Creează director pentru n8n
mkdir ~/n8n-docker
cd ~/n8n-docker

# Creează subdirectoare
mkdir workflows backups
```

## Pas 3: Creează docker-compose.yml (2 minute)

```bash
# Creează fișierul
nano docker-compose.yml
# sau folosește orice editor: code, vim, notepad++
```

**Copiază și lipește:**

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: n8n_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=n8n_password_123
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
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=n8n_password_123
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=admin123
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://localhost:5678/
      - GENERIC_TIMEZONE=Europe/Bucharest
      - NODE_ENV=production
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/workflows
      - ./backups:/home/node/backups
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  n8n_data:
  postgres_data:
```

**Salvează fișierul** (Ctrl+O, Enter, Ctrl+X în nano)

⚠️ **IMPORTANT**: Schimbă parolele înainte de producție!

## Pas 4: Pornește n8n (3 minute)

```bash
# Pornește containers
docker-compose up -d

# Verifică că rulează
docker-compose ps

# Vezi logs (opțional)
docker-compose logs -f n8n
# Apasă Ctrl+C pentru a ieși din logs
```

Ar trebui să vezi:

```
NAME            STATUS          PORTS
n8n             Up 30 seconds   0.0.0.0:5678->5678/tcp
n8n_postgres    Up 30 seconds   5432/tcp
```

## Pas 5: Deschide n8n (1 minut)

1. **Deschide browser**
2. **Navighează la**: `http://localhost:5678`
3. **Login cu Basic Auth**:
   - Username: `admin`
   - Password: `admin123`

4. **Creează Owner Account** (prima dată):
   - Email: adresa ta
   - First Name: prenumele tău
   - Last Name: numele tău
   - Password: parolă nouă pentru n8n (diferită de Basic Auth)

🎉 **Success!** n8n rulează acum pe calculatorul tău!

## Pas 6: Primul Workflow (3 minute)

### Creează Webhook → Email Workflow

**1. Creează Workflow Nou**
- Click `+ New Workflow`
- Denumește: "Test Webhook"

**2. Adaugă Webhook Node**
- Click "Add first step"
- Caută "Webhook"
- Selectează "Webhook"
- Configurare:
  - HTTP Method: `POST`
  - Path: (lasă gol)
  - Response: "Respond Immediately"
- Click "Listen for Test Event"

**3. Testează Webhook-ul**

Copiază URL-ul generat (ceva de genul `http://localhost:5678/webhook-test/abc123`)

În terminal nou:

```bash
curl -X POST http://localhost:5678/webhook-test/ABC123 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com",
    "message": "Hello from n8n!"
  }'
```

Ar trebui să vezi datele în n8n!

**4. Adaugă Set Node (Procesare)**
- Conectează de la Webhook
- Caută "Edit Fields" sau "Set"
- Add Field → String
  - Name: `greeting`
  - Value: `Hello {{ $json.name }}!`
- Add Field → String
  - Name: `timestamp`
  - Value: `{{ $now.toISO() }}`

**5. Test Workflow**
- Click "Test Workflow" în footer
- Trimite din nou curl request
- Verifică că toate nodurile sunt verzi ✓

**6. Salvează și Activează**
- Click "Save" (Ctrl/Cmd + S)
- Activează toggle-ul "Active"

✅ **Primul workflow complet!**

## Comenzi Utile

```bash
# Pornește n8n
docker-compose up -d

# Oprește n8n
docker-compose down

# Restart
docker-compose restart n8n

# Vezi logs real-time
docker-compose logs -f n8n

# Vezi logs Postgres
docker-compose logs -f postgres

# Stop și șterge tot (ATENȚIE!)
docker-compose down -v

# Update n8n la ultima versiune
docker-compose pull
docker-compose up -d

# Backup workflows
docker-compose exec n8n n8n export:workflow --all --output=/home/node/backups/backup.json

# Acces shell în container
docker-compose exec n8n sh
```

## Următorii Pași

### Opțional: Dezactivează Basic Auth

După ce ai cont n8n creat, poți dezactiva Basic Auth:

```yaml
# În docker-compose.yml
environment:
  - N8N_BASIC_AUTH_ACTIVE=false
```

```bash
docker-compose restart n8n
```

### Explorează Template-uri

Verifică folderul `Workflows/` pentru template-uri gata făcute:

1. **Webhook to Email** - Notificări simple
2. **Google Sheets Sync** - Sincronizare date
3. **Slack Bot** - Bot automatizat
4. **AI Content Assistant** - Generare conținut cu AI
5. **Database Backup** - Backup automat
6. **API Integration** - Template generic API

### Învață Mai Mult

📖 Citește documentația completă:
- **INSTRUCTIONS_RO.md** - Ghid complet în română
- **INSTRUCTIONS_EN.md** - Complete guide in English
- **README.md** - Project overview

## Troubleshooting Rapid

### ❌ Port 5678 deja folosit

```bash
# Schimbă portul în docker-compose.yml
ports:
  - "5679:5678"  # Folosește 5679 în loc de 5678

# Apoi accesează: http://localhost:5679
```

### ❌ Postgres nu pornește

```bash
# Șterge volume și reîncearcă
docker-compose down -v
docker-compose up -d
```

### ❌ n8n nu se conectează la Postgres

```bash
# Verifică logs
docker-compose logs postgres
docker-compose logs n8n

# Verifică că parolele match în docker-compose.yml
# POSTGRES_PASSWORD = DB_POSTGRESDB_PASSWORD
```

### ❌ Webhook nu primește requests

```bash
# Verifică că workflow este ACTIVE
# Verifică că URL-ul este corect
# Pentru producție, verifică firewall:
sudo ufw allow 5678
```

### ❌ Permission denied pe volumes

```bash
# Linux/Mac
sudo chown -R 1000:1000 ./workflows ./backups

# Restart
docker-compose restart n8n
```

## Resurse

### Documentație
- 📚 [n8n Docs](https://docs.n8n.io)
- 🎓 [n8n Academy](https://docs.n8n.io/courses/)
- 💬 [Community Forum](https://community.n8n.io)

### Video Tutorials
- 📺 [n8n YouTube](https://www.youtube.com/@n8n-io)
- 📺 [Nate Herk - AI Automation](https://www.youtube.com/@nateherk)

### Tools
- 🔧 [Webhook.site](https://webhook.site) - Test webhooks
- 🔧 [Crontab Guru](https://crontab.guru) - Cron expressions
- 🔧 [JSONLint](https://jsonlint.com) - Validate JSON

## Backup Important! ⚠️

```bash
# Creează backup manual
docker-compose exec n8n n8n export:workflow --all --output=/home/node/backups/workflows.json
docker-compose exec n8n n8n export:credentials --all --output=/home/node/backups/credentials.json

# Copiază din container
docker cp n8n:/home/node/backups/workflows.json ./backups/
docker cp n8n:/home/node/backups/credentials.json ./backups/

# Backup PostgreSQL
docker-compose exec postgres pg_dump -U n8n n8n > ./backups/database.sql
```

## 🎯 Checklist Finalizare

- [ ] Docker installed și rulează
- [ ] n8n containers pornite (postgres + n8n)
- [ ] Acces la http://localhost:5678
- [ ] Owner account creat
- [ ] Primul workflow testat
- [ ] Workflow salvat și activat
- [ ] Backup făcut (workflows + credentials)

---

**Gata!** Acum ai n8n self-hosted funcțional pe calculatorul tău! 🚀

**Pasul următor**: Citește **INSTRUCTIONS_RO.md** pentru ghidul complet.

*Self-hosted. Open-source. Under your control.*

