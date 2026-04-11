# 🤖 ML Job Intelligence Pipeline
> A production-grade, AI-powered job hunting automation system — built with n8n, Groq (Llama 3.1), and a self-calibrating feedback loop.

![Pipeline Status](https://img.shields.io/badge/status-active-brightgreen)
![n8n](https://img.shields.io/badge/n8n-automation-orange)
![Groq](https://img.shields.io/badge/LLM-Groq%20Llama%203.1-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## 🧠 What Is This?

Most job hunters manually scroll job boards, copy-paste descriptions, and write generic cover letters. This pipeline eliminates all of that.

Every 2 hours, this system automatically:
1. Scrapes 60+ ML/AI jobs from 3 sources
2. Scores each job against my actual resume using an LLM (0–100)
3. Generates a tailored cover letter for strong matches
4. Sends real-time Telegram alerts with scores and cover letters
5. Logs everything to a Google Sheets CRM
6. Learns from my interview outcomes to improve future scoring




This is not a tutorial project. It is a fully operational, self-improving job intelligence system running in production.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TRIGGER LAYER                                │
│         Schedule (every 2hrs) + Manual Trigger                       │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────────┐
│                      BOOTSTRAP LAYER                                 │
│   Read Existing Job IDs → Read Feedback Examples                    │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         ▼                 ▼                 ▼
   Fetch Remotive    Fetch Arbeitnow    Fetch Adzuna
         │                 │                 │
   Split+Normalize   Split+Normalize   Split+Normalize
         └─────────────────┼─────────────────┘
                           │
                    Dedup Check
                           │
                    AI Scoring (Groq)
                           │
              ┌────────────┴────────────┐
           score >= 70             score < 70
              │                        │
   ┌──────────┴──────────┐        Write to Sheet
score >= 75         score 70-74
   │                    │
Generate CL        Write to Sheet
   │                    │
Write to Sheet     Telegram Alert
   │
Telegram Alert + CL
```


---

## ✨ Features

| Feature | Description |
|---|---|
| 🔍 Multi-source ingestion | Fetches from Remotive, Arbeitnow, Adzuna simultaneously |
| 🧠 LLM scoring | Groq Llama 3.1 scores each job 0–100 against your resume |
| 📝 Cover letter generation | Auto-generates tailored cover letters for jobs ≥75 |
| 🔁 Deduplication | Never processes the same job twice across runs |
| 📊 Google Sheets CRM | Full job tracking with scores, recommendations, status |
| 📱 Telegram alerts | Real-time notifications with job details and cover letters |
| 💬 Status updates | Reply applied/interview/rejected to update sheet instantly |
| 🔄 Feedback loop | Interview/rejection outcomes improve future scoring |
| 📈 Weekly reports | Monday morning pipeline analytics sent to Telegram |
| 🚨 Error handling | Global error alerts + retry logic on all HTTP nodes |
| ❤️ Health checks | Daily Telegram ping confirming pipeline is alive |


---

## 📸 Screenshots

<table>
  <tr>
    <td><b>ML Job Pipeline Canvas</b></td>
    <td><b>Job Status Updater</b></td>
  </tr>
  <tr>
    <td><img src="docs/screenshots/Canv-%20ML%20Job%20Pipeline.png" width="400"/></td>
    <td><img src="docs/screenshots/Canv-%20Job%20Status%20Updater.png" width="400"/></td>
  </tr>
  <tr>
    <td><b>Telegram Job Alert</b></td>
    <td><b>Telegram Error Alert</b></td>
  </tr>
  <tr>
    <td><img src="docs/screenshots/Tel%20Job%20Match%20.png" width="400"/></td>
    <td><img src="docs/screenshots/Tel%20Error%20Alert.png" width="400"/></td>
  </tr>
  <tr>
    <td><b>Google Sheets CRM</b></td>
    <td><b>Weekly Report Workflow</b></td>
  </tr>
  <tr>
    <td><img src="docs/screenshots/Google%20Sheet.png" width="400"/></td>
    <td><img src="docs/screenshots/Canv-%20Weekly%20Job%20Report.png" width="400"/></td>
  </tr>
</table>


---

## 📁 Repository Structure

```
ML-Job-Intelligence-Pipeline/
|
|-- workflows/
|   |-- ml_job_pipeline.json
|   |-- job_status_updater.json
|   |-- weekly_job_report.json
|   |-- pipeline_error_handler.json
|
|-- docs/
|   |-- architecture.md
|   |-- setup.md
|   |-- screenshots/
|       |-- telegram_alert.png
|       |-- google_sheet.png
|       |-- n8n_canvas.png
|
|-- .env.example
|-- LICENSE
|-- README.md
```


## ⚙️ Tech Stack

| Layer | Technology |
|---|---|
| Automation engine | n8n (self-hosted) |
| LLM inference | Groq API (Llama 3.1 8B Instant) |
| Job sources | Remotive API, Arbeitnow API, Adzuna API |
| Storage / CRM | Google Sheets API |
| Notifications | Telegram Bot API |
| Deployment | Railway (Docker) |
| Language | JavaScript (n8n Code nodes) |

---

## 🚀 Setup Guide

### Prerequisites
- n8n installed locally (`npm install -g n8n`) or deployed on Railway
- Groq API key — [Get free key](https://console.groq.com/keys)
- Telegram bot token — Create via [@BotFather](https://t.me/botfather)
- Google account for Sheets OAuth
- Adzuna API key — [Free signup](https://developer.adzuna.com/signup)

### Step 1 — Clone The Repo

```bash
git clone https://github.com/SIYAM1809/ML-Job-Intelligence-Pipeline.git
cd ML-Job-Intelligence-Pipeline
```

### Step 2 — Start n8n

```bash
n8n start
```
Open `http://localhost:5678`

### Step 3 — Set Up Credentials

In n8n → Settings → Credentials, add:

| Credential | Type | Notes |
|---|---|---|
| Google Sheets | OAuth2 | Sign in with Google |
| Telegram | API Token | Paste BotFather token |
| Groq | API Key | Paste from console.groq.com |

### Step 4 — Create Google Sheet

Create a spreadsheet named `Job Pipeline` with two sheets:

**Sheet 1: Job Tracker**

job_id | title | company | url | source | date_posted | date_found |
score | recommendation | match_reasons | missing_skills | cover_letter | status | notes

**Sheet 2: Feedback**

job_id | title | company | description | tags | outcome | date_added

### Step 5 — Import Workflows

For each JSON in `/workflows`:
1. n8n → New Workflow → three dots → Import from file
2. Reconnect credentials when prompted

### Step 6 — Update Resume

Open **ML Job Pipeline** → **Build Scoring Prompt** node → replace `MY_RESUME` with your resume text.

### Step 7 — Activate All Workflows

Toggle **Active** ON for all 4 workflows.

---

## 📱 How To Use

### Receiving Job Alerts

🎯 New ML Job Match!
Score: 87/100
Role: ML Engineer
Company: Stripe
Fit: APPLY
📝 Strong alignment with RAG and MLOps requirements
✅ Why it fits:

LlamaIndex + Qdrant experience matches stack
Published research demonstrates ML depth
Docker + FastAPI covers deployment requirements

❌ Missing: Scala, Spark
📄 Cover Letter:
[auto-generated tailored cover letter]
🔗 https://remotive.com/...


### Updating Job Status

Reply directly to any job alert:

| Command | Action |
|---|---|
| `applied` | Marks job as applied in sheet |
| `interview` | Marks interview + saves as positive feedback |
| `rejected` | Marks rejected + saves as negative feedback |
| `offer` | Marks offer + saves as positive feedback |
| `skip` | Marks as skipped |

### Weekly Report (Every Monday 9am)

📊 Weekly Job Pipeline Report
Total Jobs Tracked: 127
High Matches (≥75): 23
Avg Match Score: 74.3/100
Status Breakdown:

Applied: 8
Interview: 3
Offer: 1
Rejected: 2

Interview Conversion Rate: 37.5%

---

## 🧬 The Feedback Loop

When you mark a job as `interview` or `offer`, it's saved as a positive example. When you mark `rejected`, it's saved as negative. On the next run, these get injected into the LLM prompt:


LEARNING FROM PAST OUTCOMES:
Jobs that led to interviews/offers (score HIGHER):
Title: ML Engineer | Company: Stripe | Tags: python, pytorch, mlops
Jobs that led to rejections (score LOWER):
Title: Data Analyst | Company: TELUS | Tags: excel, tableau

The system calibrates scoring toward roles that actually convert for your specific profile. It gets smarter every week.

---

## 🚢 Deployment (Railway)

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template/n8n)

Set these environment variables on Railway:

```env
N8N_EDITOR_BASE_URL=https://your-app.up.railway.app
WEBHOOK_URL=https://your-app.up.railway.app
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your_secure_password
```

---

## 🛡️ Error Handling & Resilience

- **Retry logic** — All HTTP nodes retry 3 times with 5s delay
- **Continue on fail** — LLM nodes fail gracefully
- **Global error handler** — Any failure triggers instant Telegram alert
- **Data guards** — Every code node validates input before processing
- **Daily health check** — 8am Telegram ping confirms pipeline is alive

---

## 🗺️ Roadmap

- [ ] LinkedIn Jobs scraping
- [ ] HuggingFace Jobs API
- [ ] Gmail integration — auto-send applications for score ≥ 90
- [ ] Salary range extraction and filtering
- [ ] Dashboard UI for pipeline analytics
- [ ] Multi-resume support

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 👤 Author

**Md. Aman Uddin Siyam**

[![GitHub](https://img.shields.io/badge/GitHub-SIYAM1809-black?logo=github)](https://github.com/SIYAM1809)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-amansiyam18-blue?logo=linkedin)](https://linkedin.com/in/amansiyam18)
[![Kaggle](https://img.shields.io/badge/Kaggle-amansiyam-blue?logo=kaggle)](https://kaggle.com/amansiyam)

---

*If this project helped you, consider giving it a ⭐*

