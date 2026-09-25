# StudyMate AI — AI Learning Assistant

An AI-powered document learning platform that lets you upload a PDF and turn it into an interactive study companion — chat with it, summarise it, get concepts explained, generate quizzes with scoring, and build flashcards for revision.

**Live Demo:** https://ai-assistant-learning-beryl.vercel.app/

**GitHub Repository:** https://github.com/shikharr05/AI-ASSISTANT-LEARNING

---

## Overview

StudyMate AI was built to solve a simple problem: students and self-learners spend a disproportionate amount of time manually searching through PDFs — textbook chapters, lecture notes, research papers, or scanned handwritten notes — to find answers, summarise content, or prepare for tests. StudyMate AI turns any uploaded PDF into an interactive, AI-powered study companion that answers questions grounded strictly in that document, summarises it, explains difficult concepts, generates quizzes, and generates flashcards — all powered by Retrieval-Augmented Generation (RAG) so responses stay accurate to the source material instead of relying on the model's general knowledge.

This was built as a solo project, taken from an initial working prototype through several real engineering challenges to a fully secured, production-hardened application assessed against the **OWASP Top 10 for LLM Applications** framework.

---

## Features

- 🔐 **Authentication** — user sign-up and sign-in
- 📄 **PDF Upload & Viewer** — upload any PDF, including scanned/image-based documents, and view it in-app
- 💬 **AI Chat** — ask free-form questions about the document, grounded in its actual content via RAG
- 📝 **Summarise** — generate a concise summary of the entire document
- 💡 **Explain a Concept** — type any topic from the document and get a detailed, document-grounded explanation
- ❓ **AI-Generated Quizzes** — auto-generate quizzes from the document, attempt them in-app, and get a detailed scorecard
- 🗂️ **Flashcards** — generate flashcards from the document and star important ones for later revision

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React.js |
| Backend | Node.js, Express.js |
| Database | MongoDB Atlas |
| Vector Search | MongoDB Atlas Vector Search |
| AI / LLM | Google Gemini API (text generation, embeddings, PDF parsing) |
| File Storage | AWS S3 (via `multer-s3`, `getObjectCommand`) |
| API Testing | Postman |

---

## Architecture

1. **Upload** — User uploads a PDF. The file is stored persistently in **AWS S3**, and simultaneously sent to the **Gemini API** for text extraction (including OCR-capable parsing of scanned/image-based PDFs).
2. **Chunking & Embedding** — Extracted text is split into overlapping chunks (to preserve context across boundaries) and each chunk is converted into a vector embedding using **Gemini's embeddings model**.
3. **Storage** — Chunk embeddings are stored in **MongoDB Atlas**, indexed for vector search, scoped to the specific document and uploading user.
4. **Retrieval (RAG)** — When a user asks a question or triggers any AI feature, their query is embedded the same way, and **MongoDB Atlas Vector Search** retrieves the most semantically relevant chunks — matching by meaning, not just literal keywords.
5. **Generation** — Retrieved chunks are passed to **Gemini**, wrapped in explicit delimiter tags instructing the model to treat them strictly as reference data, and the model generates a grounded response (chat answer, summary, explanation, quiz, or flashcards).

This RAG pipeline powers every AI feature in the app, ensuring responses stay grounded in the actual uploaded document rather than the model's general training knowledge.

---

## Engineering Challenges & Solutions

**1. OCR-less PDF parsing failing on scanned/image PDFs**
Local Node.js-based PDF parsing failed on hand-clicked image PDFs and PPT-to-PDF conversions with no selectable text layer. Solved by offloading text extraction to the Gemini API, which reads such documents natively.

**2. Naive keyword-based RAG failing on semantic mismatches**
The first retrieval implementation scored chunks by literal keyword overlap, which failed whenever a user phrased a question using a synonym or different wording than the source document. Solved by switching to vector-based RAG — embedding chunks with Gemini and retrieving via MongoDB Atlas Vector Search based on semantic similarity.

**3. Uploaded documents lost on deployment**
Render's ephemeral disk was cleared every 15 minutes during inactivity, wiping uploaded PDFs and causing "failed to fetch PDF" errors. Solved by integrating AWS S3 for persistent cloud storage.

---

## Security — OWASP Top 10 for LLM Applications

After reaching a stable feature-complete state, the application was assessed end-to-end against the **OWASP Top 10 for LLM Applications** framework. Every fix was independently tested using Postman and live attack simulations rather than assumed correct from code review alone.

| Risk | Status |
|---|---|
| LLM01 – Prompt Injection | Secured & Tested (7/7 attack variants blocked) |
| LLM02 – Sensitive Information Disclosure | Secured & Tested |
| LLM03 – Supply Chain | Tested — Clean |
| LLM04 – Data and Model Poisoning | Not Applicable |
| LLM05 – Improper Output Handling | Secured & Tested |
| LLM06 – Excessive Agency | Not Applicable |
| LLM07 – System Prompt Leakage | Partially Covered (via LLM01 fix) |
| LLM08 – Vector and Embedding Weaknesses | Secured & Tested |
| LLM09 – Misinformation | Secured & Tested |
| LLM10 – Unbounded Consumption | Secured & Tested |

Highlights from the assessment:
- **Prompt Injection:** Untrusted document content is wrapped in explicit delimiter tags with instructions to treat it strictly as data, never as commands. Tested against 7 real attack variants (direct extraction, persona hijacking, guardrail bypass, delimiter-name leakage) — all blocked.
- **Sensitive Information Disclosure:** Fixed a production bug where malformed requests leaked raw parser error output; error handling now only shows safe, reviewed messages in production.
- **Vector and Embedding Weaknesses:** Added `userId` directly into the MongoDB Atlas vector search filter, so cross-user document access is blocked at the database layer — not just by route-level checks.
- **Unbounded Consumption:** Implemented tiered rate limiting on upload and AI-generation endpoints to prevent abuse of costly Gemini/AWS-backed routes.

---

## Getting Started

```bash
# Clone the repository
git clone <repo-url>
cd ai-learning-assistant

# Install dependencies
cd backend && npm install
cd ../frontend && npm install

# Set up environment variables (see .env.example)
# GEMINI_API_KEY, MONGODB_URI, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_BUCKET_NAME, JWT_SECRET

# Run the backend
cd backend && npm run dev

# Run the frontend
cd frontend && npm run dev
```

---

## Author

**Naman Kumar Gupta**
B.Tech, Mathematics & Computing, National Institute of Technology, Kurukshetra
