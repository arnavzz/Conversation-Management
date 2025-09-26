# Conversation Management & JSON Extraction with Groq (OpenAI-Compatible)

A production-minded, Colab-ready project that demonstrates conversation memory, summarization strategies, and structured information extraction using the Groq API via the OpenAI-compatible SDK.

## Highlights

- **Conversation management**: Maintain multi-turn context and keep it relevant
- **Summarization**: Periodically compress history to preserve key facts and reduce tokens
- **JSON schema classification & extraction**: Return structured outputs using OpenAI-style tool/function calling
- **Groq API integration**: Uses OpenAI-compatible endpoints for fast inference
- **Colab + GitHub workflow**: Easy to run, review, and version

---

## Tech Stack

- **Python**: Standard library only (no external frameworks)
- **Groq API**: OpenAI-compatible chat completions + function calling
- **OpenAI SDK**: Python client pointed at Groq (`https://api.groq.com/openai/v1`)
- **Google Colab**: Turnkey, reproducible runs
- **GitHub**: Version control and review

Industry-relevant skills covered: prompt engineering, conversation state management, model integration, function/tool calling, schema validation, reproducible workflows, and documentation.

---

## Setup & Installation

### Option A — Run in Google Colab

1. Open the notebook `Conversation_Management_and_Extraction.ipynb` in Colab.
2. Provide your Groq API key when prompted, or create a `.env` in Colab:
   ```python
   with open(".env", "w", encoding="utf-8") as f:
       f.write("GROQ_API_KEY=YOUR_KEY\nGROQ_MODEL=llama-3.1-70b-versatile\n")
   ```
3. Runtime → Run all. The notebook will auto-install the OpenAI client and point it to Groq.

### Option B — Run Locally

- Requirements: Python 3.10+
- Install:
  ```bash
  pip install -r requirements.txt
  ```
- Create `.env` in the project root:
  ```env
  GROQ_API_KEY=your_key_here
  GROQ_MODEL=llama-3.1-70b-versatile
  ```
- `.env` is already ignored by Git in `.gitignore`.

---

## Usage Examples

Below are minimal, self-contained examples mirroring the notebook.

### 1) Conversation Management + Summarization

```python
from dataclasses import dataclass, field
from typing import List, Dict
from openai import OpenAI
import os

client = OpenAI(base_url="https://api.groq.com/openai/v1", api_key=os.environ["GROQ_API_KEY"])
MODEL = os.getenv("GROQ_MODEL", "llama-3.1-70b-versatile")

@dataclass
class Message:
    role: str
    content: str

@dataclass
class ConversationManager:
    model: str
    summarize_every_k: int = 3
    system_prompt: str = "You are a concise helpful assistant."
    history: List[Message] = field(default_factory=list)
    turn_counter: int = 0

    def _messages(self) -> List[Dict[str, str]]:
        msgs = [{"role": "system", "content": self.system_prompt}]
        msgs += [{"role": m.role, "content": m.content} for m in self.history]
        return msgs

    def chat(self, user_text: str) -> str:
        self.history.append(Message("user", user_text))
        resp = client.chat.completions.create(model=self.model, messages=self._messages(), temperature=0.2)
        out = resp.choices[0].message.content
        self.history.append(Message("assistant", out))
        self.turn_counter += 1
        if self.summarize_every_k and self.turn_counter % self.summarize_every_k == 0:
            self._summarize_and_replace()
        return out

    def _summarize_and_replace(self) -> None:
        msgs = self._messages() + [{"role": "user", "content": "Summarize the conversation succinctly with key facts and constraints."}]
        resp = client.chat.completions.create(model=self.model, messages=msgs, temperature=0.0)
        summary = resp.choices[0].message.content
        self.history = [Message("assistant", f"[Summary]\n{summary}")]

    def truncate_last_n_turns(self, n: int) -> None:
        keep = []
        turns = 0
        for m in reversed(self.history):
            keep.append(m)
            if m.role == "assistant":
                turns += 1
                if turns >= n:
                    break
        self.history = list(reversed(keep))

cm = ConversationManager(model=MODEL, summarize_every_k=3)
cm.history.append(Message("assistant", "Hi! I can help with your questions."))
print(cm.chat("What's the weather in Paris?"))
print(cm.chat("Recommend two museums to visit."))
print(cm.chat("Book a table at 7pm near the Louvre."))  # triggers summarization
```

Expected behavior:
- After the 3rd user turn, history is replaced with a concise summary.
- You can also call `truncate_last_n_turns(n)`, or implement char/word-based truncation.

### 2) JSON Schema Classification & Extraction (Function Calling)

```python
schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "email": {"type": "string"},
        "phone": {"type": "string"},
        "location": {"type": "string"},
        "age": {"type": "integer"},
    },
    "required": ["name", "email", "phone", "location", "age"],
    "additionalProperties": False,
}

tools = [{
    "type": "function",
    "function": {
        "name": "extract_contact_info",
        "description": "Extract structured contact details from a chat.",
        "parameters": schema,
    },
}]

chat = """
Hi I'm Alice Johnson. You can reach me at alice@example.com or +1-415-555-1212.
I'm currently in San Francisco, and I'm 29 years old.
""".strip()

messages = [
    {"role": "system", "content": "You extract structured data. Infer conservatively or leave blank."},
    {"role": "user", "content": f"From this chat, extract fields using the tool:\n\n{chat}"},
]

resp = client.chat.completions.create(
    model=MODEL,
    messages=messages,
    tools=tools,
    tool_choice="auto",
    temperature=0.0,
)
call = resp.choices[0].message.tool_calls[0]
args = call.function.arguments  # JSON string
print(args)
```

Example output (truncated):
```json
{
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "phone": "+1-415-555-1212",
  "location": "San Francisco",
  "age": 29
}
```

---

## Demo

- Notebook: `Conversation_Management_and_Extraction.ipynb` (run end-to-end for full outputs)
- Colab: Open the notebook in Google Colab and Run all
- Screenshots: (optional) Add to `docs/screenshots/` and reference here

---

## Evaluation & Impact

This project showcases:
- **ML/AI Engineering**: Conversation state design, summarization strategies, token/latency control
- **API Integration**: Groq API via OpenAI-compatible SDK, function/tool calling
- **Data Handling**: Schema definition, structured extraction, minimal validation
- **Software Craft**: Clean, readable code; reproducible notebook; version-controlled workflow
- **Product Mindset**: Demonstrates how to build durable-assistant foundations that scale

---

## Future Improvements

- Advanced memory: vector-based retrieval (RAG) + hybrid summaries
- Robust schema validation (Pydantic / JSON Schema validators)
- Observability: structured logging, tracing, token/cost dashboards
- Eval harness: golden tests for summarization & extraction accuracy
- Streaming responses; retries/backoff; rate-limit handling
- Packaging: CLI wrapper or lightweight API service

---



