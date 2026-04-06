# 📝 Blog Writing Agent

An autonomous, multi-agent blog writing system powered by **LangGraph**, **GPT-4.1-mini**, and **Gemini** — with a **Streamlit** frontend. Give it a topic, and it researches the web, plans a structured outline, writes every section in parallel, and generates relevant images — all in one pipeline.

---

## 🧠 Architecture Overview

The system is built as a **LangGraph StateGraph** with the following node pipeline:

```
START
  └─► Router
        ├─► (if needs research) Research (Tavily)
        └─► Orchestrator (Plan)
              └─► Worker × N  [parallel fan-out]
                    └─► Reducer Subgraph
                          ├─► merge_content
                          ├─► decide_images
                          └─► generate_and_place_images
                                └─► END
```

### Node Descriptions

| Node | Description |
|---|---|
| **Router** | Decides research mode: `closed_book`, `hybrid`, or `open_book`. Generates Tavily search queries if needed. |
| **Research** | Runs Tavily web searches, synthesizes and deduplicates results into structured `EvidenceItem` objects. Filters by recency for `open_book` mode. |
| **Orchestrator** | Uses GPT-4.1-mini to produce a structured `Plan` (blog title, audience, tone, blog kind, and a list of `Task` objects). |
| **Worker** | One worker spawned per task via fan-out. Each writes a single Markdown section grounded by evidence. |
| **merge_content** | Sorts and merges all section outputs into a single Markdown document. |
| **decide_images** | Decides where images add value (max 3), inserts `[[IMAGE_N]]` placeholders, and produces image prompts. |
| **generate_and_place_images** | Calls **Gemini 2.5 Flash Image** to generate images and replaces placeholders with embedded Markdown image links. |

---

## 🗂️ Project Structure

```
.
├── bwa_backend.py       # LangGraph graph definition, nodes, schemas
├── bwa_frontend.py      # Streamlit UI
├── images/              # Auto-created; generated images saved here
├── *.md                 # Auto-saved blog output files
└── .env                 # API keys (not committed)
```

---

## ⚙️ Setup

### 1. Clone the repo

```bash
git clone <your-repo-url>
cd <repo-folder>
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

Required packages include:

- `langgraph`
- `langchain-openai`
- `langchain-community`
- `streamlit`
- `pandas`
- `pydantic`
- `python-dotenv`
- `google-genai`
- `tavily-python` *(optional, for research modes)*

### 3. Configure environment variables

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_key
GOOGLE_API_KEY=your_google_key          # For Gemini image generation
TAVILY_API_KEY=your_tavily_key          # Optional — enables web research
```

> If `TAVILY_API_KEY` is not set, the agent will skip research and run in `closed_book` mode.

---

## 🚀 Running the App

```bash
streamlit run bwa_frontend.py
```

Then open [http://localhost:8501](http://localhost:8501) in your browser.

---

## 🖥️ Frontend Tabs

| Tab | What it shows |
|---|---|
| **🧩 Plan** | Blog title, audience, tone, blog kind, and a table of all tasks |
| **🔎 Evidence** | Web sources gathered during research (URLs, dates, source names) |
| **📝 Markdown Preview** | Rendered blog with inline images + download buttons |
| **🖼️ Images** | Generated images with download-as-zip option |
| **🧾 Logs** | Full event log from the LangGraph stream |

### Sidebar Features
- **Generate New Blog** — enter a topic and an as-of date, then click Generate
- **Past Blogs** — browse and reload previously generated `.md` files from the working directory

---

## 📦 Output

After generation, the app produces:

- A **Markdown file** (`<blog_title>.md`) saved to the working directory
- An `images/` folder with any generated PNG images
- In-app downloads for: raw Markdown, full ZIP bundle (MD + images), and images-only ZIP

---

## 🔍 Research Modes

| Mode | When used | Recency window |
|---|---|---|
| `closed_book` | Evergreen/conceptual topics | No research |
| `hybrid` | Topics needing recent examples or tools | Last 45 days |
| `open_book` | News, latest releases, pricing, policy | Last 7 days |

---

## 🧱 Blog Kinds Supported

- `explainer`
- `tutorial`
- `news_roundup`
- `comparison`
- `system_design`

---

## 📌 Notes

- The LLM used for planning, writing, and routing is **GPT-4.1-mini** (`langchain-openai`).
- Image generation uses **Gemini 2.5 Flash Image** via `google-genai`.
- Image generation fails gracefully — if an image can't be generated, a fallback block with the prompt and error is embedded in the Markdown instead.
- All blog sections are written in parallel (fan-out) for speed.


