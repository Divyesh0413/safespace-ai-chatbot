# SafeSpace AI 🧠💬

> **AI-Powered Mental Health Companion — Bridging Mental Health Care with AI**

[![Backend](https://img.shields.io/badge/Backend-FastAPI-009688?logo=fastapi)](https://safespace-ai-backend.onrender.com/)
[![Dashboard](https://img.shields.io/badge/Dashboard-Streamlit-FF4B4B?logo=streamlit)](https://safespace-dashboard.streamlit.app/)
[![AI Engine](https://img.shields.io/badge/AI-Groq%20Llama%203.3%2070B-F55036)](https://groq.com/)
[![Database](https://img.shields.io/badge/Database-Supabase-3ECF8E?logo=supabase)](https://supabase.com/)

---

## 📌 Overview

Mental health is a critical aspect of overall well-being, yet millions of people do not receive timely support due to stigma, high costs, and limited access to professionals. SafeSpace AI is a **privacy-first, empathetic mental health chatbot** powered by conversational AI that provides early-stage emotional support and bridges this treatment gap.

**Team Members:**
- Sarthak Pandey (0501AD231058)
- Divyesh Tiwari (0501AD231024)
- Pratham Pandey (0501AD231044)
- Dipransh Singh Baghel (0501AD231023)

---

## 🌐 Live Deployments

| Service | URL |
|---------|-----|
| 🤖 Chatbot | [safespace-ai-backend.onrender.com](https://safespace-ai-backend.onrender.com/) |
| 📊 Research Dashboard | [safespace-dashboard.streamlit.app](https://safespace-dashboard.streamlit.app/) |

---

## 🎯 Problem Statement

- Mental health issues like anxiety and depression are increasing globally
- Many people lack access to proper care due to stigma, cost, and shortage of experts
- There is an urgent need for an accessible, non-judgmental first point of contact

**Proposed Solution:** An AI-based chatbot for initial emotional support that uses NLP & Machine Learning to understand emotions, provides empathetic responses and coping strategies, ensures privacy and 24/7 availability, and encourages users to seek professional help when needed.

---

## ✨ Key Features

### 🧠 Empathetic AI Responses
- **Validates** — acknowledges and mirrors the user's current emotion
- **Inquires** — asks open-ended questions to encourage deeper reflection
- **Anonymizes** — automatically removes personal details to protect privacy
- **Supports** — provides immediate crisis response for severe distress

### 🔐 Privacy & PII Anonymization
- Built with **Microsoft Presidio** + **spaCy** (`en_core_web_lg`)
- Custom deny-list of 60+ Indian first names
- Regex recognizers for name-introduction phrases
- All chat logs are stored fully anonymized — personal details are strictly hidden
- No account or email required to use the chatbot

### 🚨 Crisis Detection
- Automatically detects mentions of self-harm or severe distress
- Immediately responds with crisis helpline numbers and safety check-ins
- Crisis Timeline in the dashboard flags urgent risk events

### 🌐 Multilingual / Hinglish Support
- Users can write in Hindi using English letters (Hinglish) following natural conversational grammar
- The AI responds naturally to mixed-language input

### 📊 Research Analytics Dashboard
- **Emotion distribution** — pie charts showing mood variety across sessions
- **Daily trend tracking** — line graphs of emotional patterns over time
- **Crisis Timeline** — visual log of high-risk interactions
- **Session metrics** — unique sessions, total messages, average response time
- **Mood filters** — filter chat logs by specific emotions
- Real-time auto-refresh powered by Streamlit

---

## 🏗️ Architecture & Tech Stack

```
User (Browser / Mobile)
        │
        ▼
  FastAPI Backend  ──────────►  Groq (Llama 3.3 70B)
  (Render)                        AI Response Generation
        │
        ▼
  PII Anonymization Pipeline
  (Presidio + spaCy en_core_web_lg)
        │
        ▼
  Supabase / PostgreSQL  ◄──────  Streamlit Dashboard
  (Anonymized Storage)             (Streamlit Cloud)
```

| Layer | Technology |
|-------|-----------|
| Backend Framework | FastAPI |
| AI Engine | Groq — Llama 3.3 70B |
| PII Anonymization | Microsoft Presidio + spaCy `en_core_web_lg` |
| Database (Production) | Supabase / PostgreSQL |
| Database (Local Dev) | SQLite |
| Dashboard | Streamlit |
| Backend Hosting | Render |
| Dashboard Hosting | Streamlit Cloud |

---

## 🗄️ Database Setup (Supabase)

### ⚠️ IPv6 / Transaction Pooler Requirement

Both Render (free tier) and Streamlit Cloud only support **IPv6 outbound networking**. You **must** use Supabase's Transaction Pooler URL:

- **Port:** `6543`
- **Username format:** `postgres.PROJECT_REF`
- **Connection string format:**
  ```
  postgresql://postgres.PROJECT_REF:PASSWORD@aws-0-REGION.pooler.supabase.com:6543/postgres
  ```

Using the direct database URL (port 5432) will fail on Render and Streamlit Cloud free tiers.

### Required Tables

Run the following in the Supabase SQL Editor:

```sql
CREATE TABLE IF NOT EXISTS conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id TEXT NOT NULL,
    user_message TEXT,
    bot_response TEXT,
    emotion TEXT,
    is_crisis BOOLEAN DEFAULT FALSE,
    timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id TEXT UNIQUE NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    last_active TIMESTAMPTZ DEFAULT NOW()
);
```

### Environment Variables

```env
GROQ_API_KEY=your_groq_api_key
DATABASE_URL=postgresql://postgres.PROJECT_REF:PASSWORD@aws-0-REGION.pooler.supabase.com:6543/postgres
```

---

## 🚀 Local Development

### Prerequisites

- Python 3.10+
- Node.js (optional, for frontend tooling)

### Backend Setup

```bash
git clone https://github.com/SarthakPandey84/safespace-ai-chatbot.git
cd safespace-ai-chatbot

pip install -r requirements.txt

# Download the required spaCy model
python -m spacy download en_core_web_lg

# Run locally (uses SQLite automatically)
uvicorn main:app --reload
```

### Dashboard Setup

```bash
cd dashboard
pip install -r requirements.txt
streamlit run app.py
```

> **Note:** Local runs use SQLite by default. Set `DATABASE_URL` in your environment to switch to Supabase/PostgreSQL.

---

## 🔧 Deployment

### Backend (Render)

The `render.yaml` configures the build and start commands. Key requirement — the build command must download `en_core_web_lg` (not `en_core_web_sm`):

```yaml
buildCommand: pip install -r requirements.txt && python -m spacy download en_core_web_lg
startCommand: uvicorn main:app --host 0.0.0.0 --port $PORT
```

Set `DATABASE_URL` (Transaction Pooler URL) and `GROQ_API_KEY` in Render's environment variable settings.

### Dashboard (Streamlit Cloud)

Point Streamlit Cloud at the `/dashboard` directory. Set the same `DATABASE_URL` environment variable in the Streamlit Cloud secrets manager.

---

## 📦 Requirements

Key dependencies in `requirements.txt`:

```
fastapi
uvicorn
groq
presidio-analyzer
presidio-anonymizer
spacy
psycopg2-binary        # Required for PostgreSQL on Render — do not omit
supabase
streamlit
```

> `psycopg2-binary` must be explicitly listed — it is easy to accidentally omit and will cause PostgreSQL connection failures on Render.

---

## 🧪 PII Anonymization Pipeline

The anonymization system uses a layered approach:

1. **spaCy NER** (`en_core_web_lg`) — entity recognition; `en_core_web_lg` is required for acceptable accuracy on Indian names (`en_core_web_sm` performs poorly)
2. **Microsoft Presidio** — PII detection and redaction
3. **Custom deny-list** — 60+ common Indian first names not caught by the base model
4. **Regex recognizers** — catches name-introduction phrases (e.g., "my name is …", "I am …")

All messages pass through this pipeline before being stored in the database. The original user message is never persisted.

---

## 📈 Dashboard Metrics (Sample)

| Metric | Value |
|--------|-------|
| Total Messages | 16 |
| Unique Sessions | 11 |
| Average Response Time | ~729 ms |
| Leading Emotion | Sadness |

The dashboard updates automatically with a configurable TTL cache and periodic `st.rerun()` calls.

---

## 🔮 Future Work

- **End-to-end encryption** for sensitive user conversations
- **Professional dashboard** for psychologists — real-time mood tracking, crisis alerts, appointment management, and AI-generated treatment suggestions
- **Psychological assessment modules** — standardized screening for stress, anxiety, and depression (PHQ-9, GAD-7)
- **Voice-based emotional detection**
- **Multilingual support** (beyond Hinglish)
- **Direct connection** with certified psychologists for professional counseling
- **Personalized therapy recommendations** based on longitudinal session data

---

## 📚 References

1. D. B. Olawade et al., "Enhancing Mental Health with Artificial Intelligence: Current Trends and Future Prospects," *Journal of Medicine, Surgery and Public Health*, vol. 3, 2024.
2. M. F. Grub and T. K. Naab, "Perceived Credibility and Social Competence of Chatbots in Mental Health Apps and the Influence of Algorithm Literacy," *Digital Health*, vol. 12, 2026.
3. "Artificial Intelligence and the Dehumanization of Patient Care," 2024.
4. "Artificial Intelligence in Nursing: Current Trends, Possibilities and Pitfalls," 2024.

---

## 🎓 Academic Context

This project was developed as a **B.Tech Final Year Project** in Artificial Intelligence & Data Science. The two core academic contributions are:

1. **PII Anonymization Pipeline** — a multi-layer, Indian-name-aware anonymization system combining Presidio, spaCy large model, custom deny-lists, and regex recognizers
2. **Research-Grade Analytics Dashboard** — emotion distribution, crisis timelines, and session metrics suitable for mental health research

---

## ⚠️ Disclaimer

SafeSpace AI is an early-stage support tool and is **not a replacement for professional mental health care**. If you or someone you know is in crisis, please contact a licensed mental health professional or a local crisis helpline immediately.

---

*Built with ❤️ for a safer, more accessible mental health future.*