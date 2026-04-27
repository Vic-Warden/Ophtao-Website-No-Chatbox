# Ophtao — Cabinet d'Ophtalmologie Dr Mégret & Associés

Static website for an ophthalmology clinic located at 60 rue Hoche, 78800 Houilles.

**Note:** This is the initial deployment. The AI chatbot widget (local RAG pipeline using DrBERT and AnythingLLM) is not included and will be integrated in a future release.

---

## Stack

Pure static site — HTML, CSS, and a single JavaScript file for UI interactions (hamburger menu, dropdown navigation). No build step, no dependencies.

---

## Project Structure

```
├── index.html
├── medical_team.html
├── consultation.html
├── pathologies.html
├── corrections.html
├── technical-equipment.html
└── assets/
    ├── css/          # base, layout, components, editorial, home, team
    ├── js/main.js
    └── images/
```

---

## Local Development

Do not open HTML files directly via `file://` — use a local server to avoid browser security restrictions.

With the VS Code **Live Server** extension:
1. Right-click `index.html` → **Open with Live Server**.
2. Site available at `http://127.0.0.1:5500`.

---

## Roadmap — Chatbot Integration

The next version will include a local AI assistant with the following components:

| Component | Role |
|-----------|------|
| AnythingLLM | RAG engine + vector store |
| Ollama (Gemma 3 12B) | Local LLM, CPU-only |
| DrBERT FastAPI | French medical text embeddings |
| LanceDB | Local vector database |

All inference runs on-premises. No patient data leaves the clinic's server.

### Deployment Steps (pending clinic server setup)

1. **Start DrBERT API (Python)**
    ```bash
    cd drbert_api
    python3.11 -m venv venv
    source venv/bin/activate  # Windows: venv\Scripts\activate
    pip install -r requirements.txt
    python main.py
    ```

2. **Start Ollama**
    ```bash
    docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
    docker exec -it ollama ollama pull gemma3:12b
    ```

3. **Start AnythingLLM**
    ```bash
    docker run -d \
      -p 3001:3001 \
      -v ${PWD}/anythingllm_storage:/app/server/storage \
      --name anythingllm \
      mintplexlabs/anythingllm
    ```

4. **Configure RAG workspace**
    - Create a workspace named **Ophtao** (slug: `ophtao`).
    - Chat mode → **Query**.
    - Inject knowledge base (`Knowledge_Base/`).

5. **Configure frontend chatbot**
    - Duplicate `chatbox/js/config.example.js` → `chatbox/js/config.js`.
    - Fill in the values:
    ```javascript
    const CONFIG = {
      ANYTHINGLLM_BASE : 'http://localhost:3001',
      WORKSPACE_SLUG   : 'ophtao',
      API_KEY          : 'YOUR_API_KEY_HERE'
    };
    ```
    - Reintegrate links to `chatbox/css/chatbox.css` and `chatbox/js/script.js` in each HTML page.

---

## Troubleshooting (future version with chatbot)

| Symptom | Cause | Solution |
|----------|-------|----------|
| Embeddings fail in AnythingLLM | DrBERT API not running | Ensure `python main.py` is active in `drbert_api/` |
| `ModuleNotFoundError: No module named 'fastapi'` | Wrong Python environment | Activate venv: `source venv/bin/activate` |
| Rust/Cargo errors during `pip install` | Python 3.14+ not supported | Downgrade to Python 3.11 or 3.12 |
| Red status dot in chat | AnythingLLM unreachable | Check Docker container: `docker ps` |