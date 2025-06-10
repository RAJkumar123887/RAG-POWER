# RAG-POWER
Project: RAG-Powered FAQ Chatbot using n8n

📁 File Structure:
rag-chatbot-project/
│
├── workflows/
│   └── rag-chatbot-workflow.json  # n8n workflow file
│
├── scripts/
│   ├── setup.py                   # Dependency setup
│   └── utils.js                   # Vector DB, embedding, cleaning
│
├── docs/
│   ├── README.md                  # Setup, usage
│   └── technical-approach.md     # Design, decisions, challenges
│
└── data/
    └── sample-documents/         # Input documents

---

README.md (docs/README.md)
# RAG-Powered FAQ Chatbot

## 🎯 Objective
Build a Retrieval-Augmented Generation chatbot using `n8n` for orchestrating:
- Document ingestion and chunking
- Embedding with OpenAI/HuggingFace
- Vector storage in Pinecone/Chroma
- Query handling and response generation using an LLM

## 📥 Input
- User queries via Webhook or HTTP API

## ⚙️ Processing Flow
1. Receive query (n8n webhook trigger)
2. Clean/validate input
3. Perform vector search for top document chunks
4. Generate response via LLM (OpenAI, Mistral, etc.)
5. Return response to user

## 🧠 Tech Stack
- **n8n**: Workflow automation
- **Python/JS**: Chunking, embeddings, vector search
- **ChromaDB/Pinecone**: Vector store
- **LLMs**: OpenAI GPT-4, Mistral

## 🧪 Testing
Sample input documents provided in `data/sample-documents/`. Use `test_query.py` to simulate API.

## 🚀 Run Instructions
```
n8n start  # Start workflow UI
python scripts/setup.py  # Install required dependencies
```

---

technical-approach.md (docs/technical-approach.md)
# Technical Design & Approach

## 📐 Architecture
```
User Query → n8n → Clean Text → Retrieve Vectors → LLM → Output
```

## 🔍 RAG Strategy
- Chunk documents (500–1000 tokens)
- Store embeddings in vector DB
- Use cosine similarity for top-k retrieval (k=3 to 5)
- Combine context + user query
- Generate LLM output

## 💡 Choices Made
- Open-source: Used ChromaDB for easy local testing
- LLM: OpenAI GPT-4 via API
- Chunking: Overlapping chunk strategy to retain context

## ⚠️ Challenges
- Token limits during context aggregation
- Choosing optimal chunk size for semantic integrity
- Handling API rate limits & latency

---

utils.js (scripts/utils.js)
```js
const fs = require('fs');
const { encode } = require('gpt-3-encoder');
const { embedText } = require('./embedding');
const { insertToVectorDB, queryVectorDB } = require('./vector_db');

function chunkText(text, chunkSize = 500, overlap = 100) {
  const tokens = encode(text);
  const chunks = [];
  for (let i = 0; i < tokens.length; i += chunkSize - overlap) {
    const chunk = tokens.slice(i, i + chunkSize);
    chunks.push(chunk);
  }
  return chunks.map(chunk => chunk.join(' '));
}

module.exports = { chunkText };
```

setup.py (scripts/setup.py)
```python
import os

# Install dependencies
os.system("pip install openai chromadb")
os.system("npm install gpt-3-encoder")
```

---

rag-chatbot-workflow.json (workflows/rag-chatbot-workflow.json)
// Simplified JSON layout
```json
{
  "nodes": [
    { "name": "Webhook", "type": "webhook" },
    { "name": "Clean Input", "type": "code", "language": "JavaScript" },
    { "name": "Query Vector DB", "type": "httpRequest" },
    { "name": "Call LLM", "type": "httpRequest" },
    { "name": "Respond to User", "type": "response" }
  ]
}
