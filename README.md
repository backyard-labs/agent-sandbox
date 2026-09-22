# Agent Sandbox

A zero-backend, client-side reasoning agent testbed and tool-use laboratory running entirely in the browser.

Designed for testing and evaluating OpenAI-compatible models (such as DeepSeek, OpenRouter, Groq, and local endpoints) with real-world tool execution, multi-step agentic chaining, persistent sessions, and granular conversation introspection.

---

## Key Features

* **100% Client-Side Architecture**: Zero backend servers, Python daemons, or proxies required. Runs directly from any static web server or browser origin.
* **Client-Side Python (WASM)**: In-browser code evaluation using [Pyodide](https://pyodide.org/) (WebAssembly). Executes Python calculations deterministically with sandboxed in-memory `stdout`/`stderr` capture and zero network exposure.
* **Real Tool Chaining**: Live search via the Wikipedia REST API (with optional Tavily fallback), Pyodide code interpretation, and mock database queries.
* **Recursive Multi-Step Loop**: Configurable iteration ceiling (1–12 steps) enabling the agent to autonomously reason, search, execute Python code, inspect outputs, and synthesize final answers.
* **State & Session Persistence**: Multi-session management saved directly to browser `localStorage` with session switching, renaming, and turn isolation.
* **Deep Observability**:
  * Complete **Conversation History Visualizer** drawer.
  * Per-turn character counts and token estimation.
  * Individual turn deletion and pruning.
  * Prompt cache hit/miss telemetry, end-to-end latency, and token generation speed metrics.
  * Instant export to formatted JSON and Markdown.

---

## Quickstart & Local Deployment

Because the entire application is contained in a single static file (`index.html`), setup takes seconds.

### Prerequisites

* Any modern web browser with WebAssembly support (Chrome, Edge, Firefox, Brave, Safari).
* An OpenAI-compatible API key (e.g., DeepSeek, OpenAI, Groq, or OpenRouter).

### Running Locally

1. Clone the repository:
   ```powershell
   git clone https://github.com/backyard-labs/agent-sandbox.git
   cd agent-sandbox
   ```

2. Start a local static HTTP server:
   ```powershell
   # Using Python 3
   python -m http.server 8000

   # Or using Node (npx)
   npx serve .
   ```

3. Open your browser and navigate to:
   ```text
   http://localhost:8000
   ```

---

## Deployment to GitHub Pages

To host the sandbox online with zero infrastructure:

1. Push the repository to GitHub.
2. Go to **Settings** > **Pages** in your repository.
3. Under **Build and deployment** > **Source**, select **Deploy from a branch**.
4. Set the branch to `master` (or `main`) and the folder to `/ (root)`.
5. Click **Save**. The sandbox will be live at `https://<your-username>.github.io/agent-sandbox/`.

---

## Configuration & Usage

### 1. Configure the Runtime
* Click the model/settings pill in the header (e.g., `deepseek-chat ⚙`).
* Enter your **API Base URL** (defaults to `https://api.deepseek.com`).
* Enter your **API Key** (stored strictly in browser memory; never written to `localStorage` or disk).
* Set **Max Agent Steps** (default: `5`, range: `1–12`).
* Enable **Stream responses (SSE)** for real-time token streaming.
* Click **Apply**.

### 2. Available Tools
The model can autonomously invoke any of the following tools:

| Tool | Engine | Description |
| :--- | :--- | :--- |
| `web_search` | MediaWiki REST API | Live Wikipedia search returning snippets and article URLs. |
| `code_interpreter` | Pyodide (WASM) | Evaluates Python expressions and scripts inside an isolated browser WebAssembly sandbox. |
| `sql_query` | In-Memory Mock | Emulates SQL query execution over sample tabular databases. |

> **Important Note on Tool Dispatch (Live Mode)**: For the model to receive tool definitions and execute function calls, ensure the **Mock Tool** selector in the UI is set to any specific tool (e.g., Code Interpreter, Web Search, or SQLite) or multi-tool profile rather than `None (no tool)`. When set to `None`, no tool schema array is appended to the API payload, and the model will operate purely as a direct completion LLM.

### 3. Inspect and Debug
* Click the **History: N messages** button to inspect the full conversation payload.
* Review exact tool parameters, raw standard output captures, and intermediate model thoughts.
* Delete or prune specific messages to test agent recovery or simulate edge cases.
* Use **Copy Markdown** or **Download JSON** to export traces for evaluation reports.

---

## Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                       Browser Tab                           │
│                                                             │
│   ┌──────────────┐      SSE Stream      ┌───────────────┐   │
│   │   UI View    │ ◄──────────────────► │  DeepSeek /   │   │
│   │ (Tailwind)   │                      │  OpenAI API   │   │
│   └──────┬───────┘                      └───────┬───────┘   │
│          │                                      │           │
│          ▼ Multi-Step Engine                    ▼           │
│   ┌─────────────────────────────────────────────────────┐   │
│   │             Local Tool Dispatch Engine              │   │
│   │                                                     │   │
│   │  • Pyodide WASM (stdout/err capture, isolated)     │   │
│   │  • Wikipedia REST API (CORS fetch)                  │   │
│   │  • Session Store (localStorage state sync)          │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## License

MIT License. Free for open research, testing, and extension.