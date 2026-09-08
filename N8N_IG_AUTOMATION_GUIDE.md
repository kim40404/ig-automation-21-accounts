# 🤖 IG AUTOMATION DENGAN N8N - PANDUAN LENGKAP

> **Platform:** N8N (Self-hosted atau Cloud)  
> **Fitur:** Auto Follow, Auto Like, Auto Comment, Auto DM  
> **Setup:** 2 Akun Instagram (1 Utama + 1 Bot)

---

## 📋 DAFTAR ISI

1. [Prerequisites](#prerequisites)
2. [Architecture](#architecture)
3. [Setup N8N](#setup-n8n)
4. [Setup Instagram](#setup-instagram)
5. [Workflow 1: Auto Follow](#workflow-1-auto-follow)
6. [Workflow 2: Auto Like](#workflow-2-auto-like)
7. [Workflow 3: Auto Comment](#workflow-3-auto-comment)
8. [Workflow 4: Auto DM](#workflow-4-auto-dm)
9. [Troubleshooting](#troubleshooting)

---

## 📋 Prerequisites

### Yang Anda perlukan:

```
✅ 2 Akun Instagram (already have)
   ├─ Akun 1: Konten Utama (target yang mau di-promote)
   └─ Akun 2: Bot Account (yang akan auto-follow, like, etc)

✅ N8N (choose one):
   ├─ Option A: N8N Cloud (free tier)
   ├─ Option B: N8N Self-hosted (via Docker)
   └─ Option C: N8N Self-hosted (via VPS/Server)

✅ Instagram Graph API Access (for N8N nodes)
   ├─ Facebook Business Account
   ├─ Instagram Business Account
   └─ Access Token

✅ Browser dengan Selenium/Playwright (untuk N8N)
   ├─ Untuk automation yang lebih advanced
   └─ Optional tapi recommended

✅ Alamat Email & Nomor HP (untuk verifikasi Instagram)
```

---

## 🏗️ Architecture

```
YOUR IG AUTOMATION SETUP
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

BOT ACCOUNT (Akun 2)
├─ Auto Follow akun target
├─ Auto Like postingan
├─ Auto Comment
└─ Auto DM ke akun utama
       ↓
    N8N WORKFLOWS
├─ Workflow 1: Trigger → Find target users
├─ Workflow 2: Trigger → Follow target user
├─ Workflow 3: Trigger → Like post
├─ Workflow 4: Trigger → Generate & Post comment
└─ Workflow 5: Trigger → Send DM
       ↓
MAIN ACCOUNT (Akun 1)
├─ Receive follows dari bot
├─ Receive likes dari bot
├─ Receive comments dari bot
└─ Receive DM dari bot
       ↓
MONITORING & LOGGING
├─ Google Sheets (optional - untuk tracking)
├─ Database (optional - untuk analytics)
└─ N8N Logs (untuk debugging)
```

---

## 🚀 SETUP N8N

### OPTION A: N8N CLOUD (RECOMMENDED - Most Easy)

#### Step 1: Daftar N8N Cloud

```
1. Buka: https://n8n.cloud/
2. Click "Sign up"
3. Isi email & password
4. Verify email
5. Dashboard N8N sudah siap
6. Klik "Create workflow"
```

**Biaya N8N Cloud:**
```
├─ Free tier              : Rp 0/bulan
│  └─ 5 active workflows, limited executions
├─ Starter              : $20/bulan = Rp 320K
│  └─ Unlimited workflows, 50K executions/bulan
└─ Pro                  : $50/bulan = Rp 800K
   └─ Team collaboration, priority support
```

**Kelebihan Cloud:**
- ✅ Instant setup (tidak perlu install)
- ✅ Auto-backup
- ✅ Managed hosting

---

### OPTION B: N8N SELF-HOSTED VIA DOCKER (RECOMMENDED - More Control)

#### Step 1: Install Docker

**Windows/Mac:**
```bash
# Download Docker Desktop dari https://www.docker.com/products/docker-desktop
# Install & restart
docker --version  # Verify installation
```

**Linux (Ubuntu):**
```bash
sudo apt-get update
sudo apt-get install docker.io docker-compose
docker --version
```

#### Step 2: Run N8N via Docker

```bash
# Create folder untuk N8N
mkdir n8n-ig-automation
cd n8n-ig-automation

# Download docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=your-secure-password
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://localhost:5678/
    volumes:
      - n8n_data:/home/node/.n8n
    restart: unless-stopped

  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: n8n_db_password
      POSTGRES_DB: n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  n8n_data:
  postgres_data:
EOF

# Start N8N
docker-compose up -d

# Check if running
docker-compose logs n8n
```

**Access N8N:**
```
URL: http://localhost:5678
Username: admin
Password: your-secure-password
```

**Biaya Self-hosted:**
```
├─ VPS Server          : Rp 50K-150K/bulan (DigitalOcean, Linode)
├─ N8N License         : Rp 0 (open-source)
└─ Total              : Rp 50K-150K/bulan
```

---

## 📱 SETUP INSTAGRAM

### Step 1: Prepare Your 2 Accounts

**Akun 1 (Main Account):**
```
✅ Username: [your-main-account]
✅ Email: your-email@gmail.com
✅ Password: [your-password]
✅ 2FA: Disable temporarily (untuk testing)
✅ Account type: Business Account (recommended)
```

**Akun 2 (Bot Account):**
```
✅ Username: [your-bot-account]
✅ Email: bot-email@gmail.com
✅ Password: [bot-password]
✅ 2FA: Disable temporarily
✅ Account type: Business Account
✅ Bio: [something generic or empty]
```

### Step 2: Get Instagram Business Account Setup

⚠️ **IMPORTANT:** Instagram official API memerlukan Business Account & App Review.
Karena ini complicated, kita akan gunakan **Instagram Private API via N8N HTTP nodes + Selenium**.

---

## 🔧 WORKFLOW 1: AUTO FOLLOW

### Step 1: Create Workflow di N8N

```
1. Buka N8N Dashboard: http://localhost:5678
2. Click "+ Create workflow"
3. Rename: "IG Bot - Auto Follow"
4. Start creating nodes
```

### Step 2: Build the Workflow

```
WORKFLOW STRUCTURE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Trigger: Schedule]
    ↓
[Node 1: Get Instagram Username]
    ↓
[Node 2: Check if Already Following]
    ↓
[Decision: If not following]
    ↓
[Node 3: Execute Follow via Selenium]
    ↓
[Node 4: Log Result to Google Sheets]
```

### Step 3: Node Details

#### **Node 1: Trigger (Schedule)**

```
Type: Trigger → Cron Job
Schedule: Every 1 hour
Timezone: Asia/Jakarta

Settings:
├─ Minute: 0
├─ Hour: * (every hour)
└─ Save
```

**Screenshot location:** Click "Trigger" pada top-left

---

#### **Node 2: Get Target Username**

```
Type: Node → Function / HTTP Request
Method: GET
URL: https://api.instagram.com/v1/user/search?q={target_username}&access_token={access_token}

Response akan contain:
{
  "users": [
    {
      "id": "123456",
      "username": "target_username",
      "full_name": "Target Name"
    }
  ]
}
```

**TAPI CARA MUDAH (tanpa official API):**

Kita gunakan **Selenium node** untuk browser automation:

```
Type: Node → Execute Command / Webhook
Atau gunakan N8N Community Node: "Instagram"

Atau manual approach:
├─ Create HTTP node to get username
├─ Parse response
└─ Extract user ID
```

---

### SIMPLE SOLUTION: Gunakan N8N Community Nodes

N8N punya community nodes untuk Instagram yang lebih mudah.

**Cara install:**

```
1. Di N8N Dashboard → Settings → Community Nodes
2. Search: "instagram"
3. Install: "n8n-nodes-base-instagram"
4. Restart N8N
5. Sekarang ada "Instagram" node siap pakai
```

---

## 🎯 FULL WORKFLOW: AUTO FOLLOW (DETAILED)

### Workflow JSON (Copy-Paste ke N8N):

```json
{
  "nodes": [
    {
      "parameters": {
        "interval": [
          {
            "recurrence": "everyHour"
          }
        ]
      },
      "id": "1",
      "name": "Cron Trigger",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "url": "=https://www.instagram.com/api/v1/users/search/?ig_sig_key_version=4&signed_body={{$env.IG_SIG}}{\"user_input\":\"{{$node[\"Trigger\"].json.body.target_username}}\"}",
        "method": "GET",
        "authentication": "genericCredentialType",
        "genericCredentials": "instagram_credentials"
      },
      "id": "2",
      "name": "Search Target User",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.1,
      "position": [450, 300]
    },
    {
      "parameters": {
        "resource": "follow",
        "userId": "={{$node[\"Search Target User\"].json.body.users[0].pk}}",
        "operation": "follow"
      },
      "id": "3",
      "name": "Follow User",
      "type": "n8n-nodes-base-instagram.instagram",
      "typeVersion": 1,
      "position": [650, 300]
    },
    {
      "parameters": {
        "spreadsheetId": "your-google-sheet-id",
        "range": "A1",
        "values": "=[[\"{{new Date().toLocaleString()}}\", \"Followed\", \"{{$node[\"Search Target User\"].json.body.users[0].username}}\"]]",
        "options": {}
      },
      "id": "4",
      "name": "Log to Google Sheets",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 4.1,
      "position": [850, 300]
    }
  ],
  "connections": {
    "Cron Trigger": {
      "main": [
        [
          {
            "node": "Search Target User",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Search Target User": {
      "main": [
        [
          {
            "node": "Follow User",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Follow User": {
      "main": [
        [
          {
            "node": "Log to Google Sheets",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

---

## 💬 WORKFLOW 2: AUTO LIKE

```
WORKFLOW STRUCTURE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Trigger: New Post from Main Account]
    ↓
[Node 1: Get Latest Posts]
    ↓
[Node 2: Loop Through Posts]
    ↓
[Node 3: Like Each Post (via Bot Account)]
    ↓
[Node 4: Wait Random Delay (2-5 min)]
    ↓
[Node 5: Log to Google Sheets]
```

### Setup Steps:

```
1. Di N8N, buat workflow baru: "IG Bot - Auto Like"

2. Add Node: Cron Trigger
   Schedule: Every 30 minutes

3. Add Node: Instagram - Get Posts
   Parameter:
   ├─ Username: your-main-account
   ├─ Limit: 5 (get last 5 posts)

4. Add Node: Split In Batches
   Batch size: 1 (process 1 post at a time)

5. Add Node: Instagram - Like Post
   Parameter:
   ├─ Post ID: {{$node["Get Posts"].json.id}}
   ├─ Account: {{$env.IG_BOT_USERNAME}}

6. Add Node: Wait
   Duration: Random between 120-300 seconds (2-5 min)
   Reason: Simulate human behavior

7. Add Node: Google Sheets - Append
   Log: timestamp, action=like, post_id
```

---

## 💬 WORKFLOW 3: AUTO COMMENT

```
WORKFLOW STRUCTURE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Trigger: New Post from Main Account]
    ↓
[Node 1: Get Latest Posts]
    ↓
[Node 2: Generate Comment (via AI or predefined)]
    ↓
[Node 3: Post Comment to Instagram]
    ↓
[Node 4: Wait Random Delay (3-10 min)]
    ↓
[Node 5: Log Result]
```

### Setup Steps:

```
1. Create workflow: "IG Bot - Auto Comment"

2. Add Node: Cron Trigger
   Schedule: Every 1 hour

3. Add Node: Get Latest Posts
   Get posts from: your-main-account

4. Add Node: Function / Code
   Purpose: Generate comment
   
   Code:
   ├─ Option A: Predefined comments (random from array)
   ├─ Option B: API call to generate via OpenAI
   └─ Option C: Template-based

   SIMPLE CODE:
   const comments = [
     "Mantap banget! 🔥",
     "Suka kontennya! 👍",
     "Amazing! ✨",
     "Keren sekali! 💯",
     "Follow back? 😊"
   ];
   return {
     comment: comments[Math.floor(Math.random() * comments.length)]
   };

5. Add Node: Instagram - Post Comment
   Parameter:
   ├─ Post ID: {{$node["Get Posts"].json.id}}
   ├─ Comment: {{$node["Function"].json.comment}}
   ├─ Account: {{$env.IG_BOT_USERNAME}}

6. Add Node: Wait
   Random delay: 180-600 seconds (3-10 min)

7. Add Node: Log to Google Sheets
```

---

## 📨 WORKFLOW 4: AUTO DM

```
WORKFLOW STRUCTURE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Trigger: When received Follow / Comment]
    ↓
[Node 1: Check if New Follower]
    ↓
[Node 2: Generate Welcome Message]
    ↓
[Node 3: Send DM to New Follower]
    ↓
[Node 4: Log Result]
```

### Setup Steps:

```
1. Create workflow: "IG Bot - Auto DM"

2. Add Node: Webhook Trigger
   OR Cron Trigger (check every 15 min)

3. Add Node: Get New Followers
   Check: followers yang belum menerima DM
   (dari database/Google Sheets)

4. Add Node: Function / Code
   Generate DM message:
   
   const messages = [
     "Halo! Terima kasih sudah follow 👋",
     "Senang berkenalan dengan mu! 😊",
     "Jangan lupa lihat postingan terbaru kami 🔥"
   ];
   
   return {
     message: messages[Math.floor(Math.random() * messages.length)]
   };

5. Add Node: Instagram - Send Message
   Parameter:
   ├─ Recipient: {{$node["New Followers"].json.username}}
   ├─ Message: {{$node["Generate Message"].json.message}}
   ├─ Account: {{$env.IG_BOT_USERNAME}}

6. Add Node: Log to Google Sheets + Mark as sent
```

---

## 🔑 SETUP CREDENTIALS DI N8N

### Step 1: Add Instagram Credentials

```
1. Di N8N Dashboard → Credentials (icon kunci)
2. Click "+ Create new"
3. Choose: Instagram
4. Fill credentials:
   ├─ Username: your-bot-account-username
   ├─ Password: your-bot-account-password
   ├─ Account ID: [auto-fill saat login]
   └─ Save
```

### Step 2: Add Google Sheets (untuk logging)

```
1. Click "+ Create new"
2. Choose: Google Sheets
3. Authenticate dengan Google Account
4. Save
```

### Step 3: Environment Variables

```
Buka file .env (jika self-hosted):

N8N_IG_BOT_USERNAME=your-bot-account
N8N_IG_BOT_PASSWORD=your-bot-password
N8N_MAIN_ACCOUNT_USERNAME=your-main-account
N8N_GOOGLE_SHEET_ID=your-google-sheet-id
IG_SIG=your-instagram-signature-key
```

---

## 🧪 TESTING & DEBUGGING

### Test Workflow 1 (Auto Follow):

```
1. Buka workflow "IG Bot - Auto Follow"
2. Click "Test workflow" atau "Execute workflow"
3. Check logs:
   ├─ ✅ SUCCESS: Akan muncul "Followed user successfully"
   ├─ ❌ ERROR: Check error message
   └─ Lihat di Google Sheets jika record berhasil

4. Debugging:
   ├─ Node Error? Click node → lihat error message
   ├─ Credential Error? Check credentials di N8N
   ├─ API Error? Check Instagram rate limits
   └─ Log Error? Check Google Sheets permission
```

### Test Workflow 2 (Auto Like):

```
1. Buka workflow "IG Bot - Auto Like"
2. Execute
3. Cek:
   ├─ Check Instagram (main account) → lihat likes dari bot
   ├─ Check Google Sheets → lihat log
   └─ Check N8N logs → lihat execution details
```

### Test Workflow 3 (Auto Comment):

```
1. Execute
2. Cek di Instagram post:
   ├─ Main account post → lihat comments dari bot
   ├─ Lihat comment text (generated atau predefined)
   └─ Check timestamp cocok dengan workflow execution
```

### Test Workflow 4 (Auto DM):

```
1. Execute
2. Cek Instagram messages:
   ├─ Main account → Direct Messages
   ├─ Lihat DM dari bot
   └─ Check message content
```

---

## 📊 SETUP GOOGLE SHEETS (untuk tracking)

### Step 1: Create Google Sheet

```
1. Buka https://sheets.google.com
2. Create new spreadsheet: "IG Bot Automation Logs"
3. Create sheets (tabs):
   ├─ Sheet 1: "Follows"
   │  └─ Columns: Timestamp | Action | Username | Status
   ├─ Sheet 2: "Likes"
   │  └─ Columns: Timestamp | Action | Post ID | Status
   ├─ Sheet 3: "Comments"
   │  └─ Columns: Timestamp | Action | Post ID | Comment | Status
   └─ Sheet 4: "DMs"
      └─ Columns: Timestamp | Action | Recipient | Message | Status

4. Share dengan N8N:
   ├─ Click "Share"
   ├─ Add email: [N8N service account email]
   └─ Give Editor access
```

### Step 2: Get Google Sheet ID

```
URL: https://docs.google.com/spreadsheets/d/{SHEET_ID}/edit

Copy SHEET_ID dan pakai di N8N nodes
```

---

## ⏰ SCHEDULING

### Setup Auto-run Workflows:

```
WORKFLOW 1: Auto Follow
├─ Trigger: Every 2 hours
├─ Time: 08:00, 10:00, 12:00, 14:00, 16:00, 18:00, 20:00
└─ Randomize ±15 minutes

WORKFLOW 2: Auto Like
├─ Trigger: Every 1 hour
├─ Add random delay: 120-300 seconds
└─ All day, spread throughout

WORKFLOW 3: Auto Comment
├─ Trigger: Every 2 hours
├─ Add random delay: 180-600 seconds
└─ Peak hours: 09:00-22:00

WORKFLOW 4: Auto DM
├─ Trigger: Every 6 hours
├─ Or: Webhook (realtime)
└─ Time: 09:00, 13:00, 17:00, 21:00
```

---

## ⚠️ ANTI-BAN TIPS

### 1. Action Timing
```
❌ JANGAN: Follow 20 orang dalam 1 menit
✅ GUNAKAN: Follow 1 orang setiap 2 jam
           Spacing: Random 120-300 seconds
```

### 2. Rate Limits
```
Instagram limits per akun:
├─ Follow: ~100-400 per hari
├─ Like: ~200-500 per hari
├─ Comment: ~100-200 per hari
└─ DM: ~50-100 per hari

Gunakan HANYA 50% dari limit untuk safety:
├─ Follow: 50-200 per hari (max 5 per jam)
├─ Like: 100-250 per hari (max 10 per jam)
├─ Comment: 50-100 per hari (max 5 per jam)
└─ DM: 25-50 per hari (max 2 per jam)
```

### 3. Human-Like Behavior
```
✅ Add random delays antara actions
✅ Random action order (sometimes follow before like, etc)
✅ Don't interact with same account too often
✅ Vary comment text (jangan template sama)
✅ Mix actions (follow + like + comment, bukan hanya follow)
✅ Offline time (jangan 24/7, off dari 23:00-08:00)
```

### 4. Account Warmup
```
MINGGU 1:
├─ Follow: 50 orang
├─ Like: 100 posts
├─ Comment: 30 posts
└─ DM: 10 orang

MINGGU 2:
├─ Follow: 100 orang
├─ Like: 150 posts
├─ Comment: 50 posts
└─ DM: 20 orang

MINGGU 3+:
├─ Follow: 200 orang
├─ Like: 250 posts
├─ Comment: 100 posts
└─ DM: 50 orang
```

---

## 🔍 MONITORING & ALERTS

### Setup Alert jika Error:

```
Di N8N, setelah workflow selesai, add node:

[Decision Node]
├─ If Status = ERROR
│  └─ Send Alert Email
└─ Else: Continue

Alert settings:
├─ Email to: your-email@gmail.com
├─ Subject: IG Bot Error - {{$node["Node Name"].error}}
└─ Body: Error details...
```

---

## 📈 METRICS TO TRACK

```
Setiap hari catat:
├─ Total Follow: X orang
├─ Total Like: X posts
├─ Total Comment: X posts
├─ Total DM: X orang
├─ Bot Account Followers: X
├─ Bot Account Status: Active/Suspended
└─ Main Account Growth: +X followers

Bulanan:
├─ Success Rate: XX%
├─ Error Rate: XX%
├─ Ban Risk: Low/Medium/High
└─ Next Actions: Increase/Maintain/Decrease
```

---

## 🆘 TROUBLESHOOTING

### Problem 1: "Invalid Credentials"

```
Solution:
1. Check username/password di N8N credentials
2. Login manually di Instagram (check 2FA)
3. Check if account is suspended/locked
4. Regenerate credentials di N8N
5. Test again
```

### Problem 2: "Rate Limited by Instagram"

```
Solution:
1. Reduce action frequency
2. Add longer delays (300-600 seconds)
3. Reduce daily limits
4. Wait 24 hours before resuming
5. Switch to less aggressive schedule
```

### Problem 3: "Bot Account Suspended"

```
❌ RIP. Create new bot account & start warmup again
Prevention:
✅ Follow anti-ban tips strictly
✅ Don't exceed rate limits
✅ Human-like behavior always
✅ Monitor logs daily
```

### Problem 4: "Comments/DM tidak terkirim"

```
Solution:
1. Check if recipient account is private
2. Check if message contains banned keywords
3. Check rate limits (DM limit might be reached)
4. Try with different message
5. Check N8N logs for error details
```

---

## ✅ CHECKLIST IMPLEMENTASI

```
PHASE 1: SETUP (Day 1-2)
□ Install N8N (Cloud atau Self-hosted)
□ Setup 2 Instagram accounts
□ Add Instagram credentials ke N8N
□ Add Google Sheets credentials
□ Create Google Sheet untuk logging

PHASE 2: BUILD WORKFLOWS (Day 3-5)
□ Build Workflow 1: Auto Follow
□ Build Workflow 2: Auto Like
□ Build Workflow 3: Auto Comment
□ Build Workflow 4: Auto DM
□ Add logging untuk setiap workflow

PHASE 3: TESTING (Day 6-8)
□ Test auto follow (5 orang)
□ Test auto like (10 posts)
□ Test auto comment (5 posts)
□ Test auto DM (5 orang)
□ Check logs di Google Sheets
□ Debug errors

PHASE 4: DEPLOYMENT (Day 9-10)
□ Set proper scheduling
□ Enable all workflows
□ Add alert notifications
□ Monitor for 24 hours
□ Check ban risk
□ Adjust if needed

PHASE 5: OPTIMIZATION (Day 11+)
□ Analyze metrics
□ Improve comment quality
□ Adjust timing/frequency
□ Monitor bot account health
□ Scale up gradually
```

---

## 📞 RESOURCES

| Resource | Link |
|----------|------|
| **N8N Docs** | https://docs.n8n.io/ |
| **N8N Community** | https://community.n8n.io/ |
| **Instagram API Docs** | https://developers.instagram.com/ |
| **N8N Community Nodes** | https://www.npmjs.com/search?q=n8n-nodes |
| **Google Sheets API** | https://developers.google.com/sheets/api |

---

## 💰 COST SUMMARY

```
N8N Cloud Free tier:        Rp 0/bulan
OR
N8N Self-hosted (VPS):      Rp 50K-150K/bulan

Google Sheets:              Rp 0 (free)
Google Drive:               Rp 0 (15GB free)

TOTAL:                      Rp 0 - 150K/bulan
```

---

**Ready to implement? Start with Phase 1! 🚀**

---

*Last updated: September 5, 2026*
