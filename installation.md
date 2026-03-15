# Installation & Setup Guide
**Credit Risk Pipeline — MacBook 2017 compatible**

---

## Tools you need

### Core tools

| Tool | What it's for | Free? |
|------|--------------|-------|
| VS Code | Code editor | Yes |
| Python 3.10+ | All pipeline code | Yes |
| Jupyter Notebook | Your existing ML work | Yes |
| Git | Version control | Yes |
| Homebrew | Mac package manager | Yes |

### Python libraries

| Library | What it's for |
|---------|--------------|
| `openai` | LLM calls (email composer + loan categorizer) |
| `langchain` | RAG orchestration |
| `chromadb` | Local vector store (lightweight, runs on your Mac) |
| `pandas` | Data handling |
| `scikit-learn` | Your existing ML model |
| `google-auth` + `google-api-python-client` | Gmail API |
| `python-dotenv` | Manage API keys safely |
| `jupyter` | Notebook support |
| `tiktoken` | Token counting for chunking |

### API keys you need

| Service | Cost | What for |
|---------|------|---------|
| OpenAI | ~$0.01/email | LLM for email + categorizer |
| Google Cloud | Free tier | Gmail sending |

---

## Step-by-step installation

### Step 1 — Install Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Step 2 — Install Python 3.10+

```bash
brew install python@3.10
python3 --version   # should print 3.10.x or higher
```

### Step 3 — Install VS Code

Download from: https://code.visualstudio.com

Install these extensions:
- Python (Microsoft)
- Jupyter (Microsoft)

### Step 4 — Install Git

```bash
brew install git
git --version
```

### Step 5 — Create project and virtual environment

```bash
mkdir credit-risk-pipeline
cd credit-risk-pipeline
python3 -m venv venv
source venv/bin/activate
```

> Run `source venv/bin/activate` every time you open a new terminal session.

### Step 6 — Install all Python libraries

```bash
pip install openai langchain langchain-openai langchain-community \
  chromadb pandas scikit-learn google-auth google-auth-oauthlib \
  google-api-python-client python-dotenv jupyter tiktoken
```

> Takes 3–5 minutes. No GPU needed — all heavy lifting is done via APIs.

### Step 7 — Set up API keys

Create a `.env` file in your project root:

```
OPENAI_API_KEY=sk-your-key-here
```

Add to `.gitignore` so you never accidentally commit it:

```bash
echo ".env" >> .gitignore
```

---

## Gmail API setup

This is the trickiest part — follow these steps exactly:

1. Go to https://console.cloud.google.com
2. Create a new project (e.g. "CreditRiskPipeline")
3. Go to **APIs & Services → Enable APIs**
4. Search for and enable **Gmail API**
5. Go to **APIs & Services → Credentials**
6. Click **Create Credentials → OAuth client ID**
7. Application type: **Desktop app**
8. Download the `credentials.json` file
9. Place it in your project root
10. Add `credentials.json` and `token.json` to `.gitignore`

On first run, a browser window opens asking you to log in. After that, it runs fully automatically.

---

## Project folder structure

```
credit-risk-pipeline/
│
├── venv/                        # virtual environment (don't touch)
├── .env                         # API keys (never commit)
├── .gitignore
├── credentials.json             # Gmail OAuth (never commit)
│
├── data/
│   └── loan_applications.csv
│
├── policy_docs/                 # RAG source documents
│   ├── interest_rates.txt
│   ├── rejection_offers.txt
│   └── legal_disclaimers.txt
│
├── notebooks/
│   └── CreditRisk.ipynb
│
├── pipeline/
│   ├── model.py
│   ├── fraud_check.py
│   ├── rag.py
│   ├── email_composer.py
│   └── email_sender.py
│
└── main.py
```

---

## Useful commands

```bash
# Activate environment (every session)
source venv/bin/activate

# Launch Jupyter
jupyter notebook

# Run the full pipeline
python main.py

# Deactivate environment
deactivate
```

---

## Performance notes for 2017 MacBook

- ChromaDB runs entirely locally — very lightweight, no issues
- OpenAI API does the heavy lifting remotely — no GPU needed
- Keep your CSV small during testing (5–10 rows first)
- ChromaDB vector store only needs to be built once — loads from disk after that
- Close heavy browser tabs when running the pipeline for the first time

---

## Rough build timeline

| Week | What to build |
|------|--------------|
| 1 | Set up environment, migrate notebook to `pipeline/model.py`, write `fraud_check.py` |
| 2 | Build RAG with ChromaDB, write email composer |
| 3 | Set up Gmail API, wire `main.py`, test end-to-end |
