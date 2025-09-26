# Conversation-Management

## Colab Notebook

- Open `Conversation_Management_and_Extraction.ipynb` in Google Colab.
- Provide your `GROQ_API_KEY` when prompted (or set it in the Colab environment).
- Optionally set `GROQ_MODEL` (default: `llama3-70b-8192`).

## Tasks Implemented

- Conversation management with:
  - Truncation by last N turns
  - Truncation by character limit
  - Truncation by word limit
  - Periodic summarization every k-th conversation cycle (history replacement)
- JSON schema style extraction using OpenAI-style function calling on Groq API
  - Extracts: name, email, phone, location, age
  - Manual schema validation without external libraries

## How to Run

1. Open the notebook in Colab and run all cells top-to-bottom.
2. Enter your Groq API key when prompted.
3. Observe outputs for:
   - Multiple conversation samples and different truncation settings
   - Periodic summarization after every 3rd turn
   - Parsing 3 chats and schema validation

## Environment (.env)

- Create a `.env` file in the project root with:
  ```env
  GROQ_API_KEY=your_key_here
  GROQ_MODEL=llama3-70b-8192
  ```
- The notebook automatically loads `.env` (no external libraries) and falls back to a prompt if the key is missing.
- `.env` is already listed in `.gitignore` so it won’t be committed.

Quick commands to create files:
- PowerShell:
  ```powershell
  "GROQ_API_KEY=your_key_here`nGROQ_MODEL=llama3-70b-8192" | Out-File -Encoding utf8 .env
  "GROQ_API_KEY=your_key_here`nGROQ_MODEL=llama3-70b-8192" | Out-File -Encoding utf8 .env.example
  ```
- Bash:
  ```bash
  printf "GROQ_API_KEY=your_key_here\nGROQ_MODEL=llama3-70b-8192\n" > .env
  printf "GROQ_API_KEY=your_key_here\nGROQ_MODEL=llama3-70b-8192\n" > .env.example
  ```

## Local Usage

- Requires Python 3.10+
- Install dependencies:
  ```bash
  pip install -r requirements.txt
  ```
- Set environment variables (alternative to .env):
  ```powershell
  $env:GROQ_API_KEY="YOUR_KEY"
  $env:GROQ_MODEL="llama3-70b-8192"  # optional
  ```

## Git & GitHub

Initialize repo and push:
```bash
git init
git add Conversation_Management_and_Extraction.ipynb README.md requirements.txt
git commit -m "Initial commit: Colab notebook for conversation management and JSON extraction with Groq API"
 git branch -M main
 git remote add origin https://github.com/<your-username>/<your-repo>.git
 git push -u origin main
```

