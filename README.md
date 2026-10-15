# libugry (library + bug haha get it get it </3)

TIRED of endless bugs from stupid library conflicts?? NO MORE >:D this evening i built an app that queries a Context Graph (built w @neo4j) that maps out OS-specific crashes + CVEs + hidden constraints. it outputs full reasoning for the versions it picks to keep your env safe ^_~

<img width="3674" height="2200" alt="HEsw41TWYAAbSgK" src="https://github.com/user-attachments/assets/97d14717-431b-492d-b44b-f03334310953" />

<img width="2998" height="1694" alt="HEsw412bIAAB4UV" src="https://github.com/user-attachments/assets/5351d047-56b2-4098-ab97-3133fe5eb2ed" />

<img width="3670" height="2198" alt="HEsw416a8AA63pi" src="https://github.com/user-attachments/assets/6a9c2a80-7039-40aa-a459-2132f5d15fd2" />

<img width="3674" height="2200" alt="HEsw41TWYAAbSgK" src="https://github.com/user-attachments/assets/07a9f92e-1740-40ea-811d-adb591a32cab" />





Context-aware Python library dependency advisor. Where pip asks "does this version exist?", libugry asks "which version should I use on **this exact machine**, and **why**?"

Standard package managers operate on a **State Clock** — static PyPI metadata. libugry operates on an **Event Clock** — a Neo4j Context Graph that stores observed reality: crash history per OS/arch/Python, verified working combinations, CVE vulnerabilities, license types, and the full reasoning trace of every past decision. The graph gets smarter with every install.

**Stack:** Python · [Claude API](https://console.anthropic.com) (claude-sonnet-4-6 tool-use) · Neo4j 5 · Streamlit · [OSV.dev](https://osv.dev) · PyPI JSON API

---

## Quickstart

**Prerequisites:** Docker + Docker Compose, Python 3.9+, Anthropic API key

### 1. Configure credentials

Copy `.env` and fill in:
```
ANTHROPIC_API_KEY=sk-ant-...
NEO4J_PASSWORD=changeme
```

### 2. Start Neo4j

```bash
docker compose up -d
# wait ~15s — Neo4j browser at http://localhost:7474
```

### 3. Install dependencies

```bash
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 4. Initialize and seed

```bash
python main.py init   # creates Neo4j constraints + indexes
python main.py seed   # loads 13 libraries, crash events, bundles
```

### 5. Run

**CLI (natural language REPL):**
```bash
python main.py
> I need pandas and scikit-learn for a data pipeline
> why does scipy crash on my machine?
> assess numpy + torch compatibility
> install requests
```

**Web UI:**
```bash
streamlit run ui.py
```

---

## Commands

| Command | What it does |
|---|---|
| `python main.py` | Interactive REPL — natural language queries |
| `python main.py query <lib> [<lib2> ...]` | Recommend versions, no install |
| `python main.py install <library>` | Navigate → confirm → sandbox test → write outcome |
| `python main.py history <library>` | Table of past decisions + outcomes |
| `python main.py graph <library>` | Dependency tree |
| `python main.py init` | Apply Neo4j schema (safe to re-run) |
| `python main.py seed` | Load foundational data (safe to re-run) |

See [QUICKSTART.txt](QUICKSTART.txt) for full setup details, troubleshooting, and how the system works.

---

## How it works

1. **env_detector.py** fingerprints your machine: `{os, arch, python}`
2. **agents/navigator.py** runs a Claude tool-use loop — querying the graph for CVEs, crash records, compatibility, licenses, dep conflicts, and verified bundles — then returns a structured recommendation with confidence level
3. Every recommendation writes a **Decision node** to Neo4j with the full tool call trace (provenance: which queries Claude ran and in what order)
4. **sandbox.py** (install path) creates a temp venv, runs `pip install`, captures the result
5. **agents/feedback.py** classifies crash causes (regex → Claude haiku fallback) and writes `CrashCause` + `Outcome` nodes back to the graph

The Context Graph stores six layers: Security (CVE), Environment (CRASHES_ON / COMPATIBLE_WITH), Causation (CrashCause WHY nodes), Legal (License), Structural (DEPENDS_ON constraints), and Decision + Provenance.
