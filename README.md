# Novel Quest

Novel Quest is a React + Vite reading workspace for long-form manuscripts. It combines PDF ingestion, image OCR, AI chat, translation, reading insights, and Firebase-backed persistence so readers can upload material, ask questions about it, and organize their notes in one place.

## What It Does

- Uploads PDFs and converts them to readable manuscript text.
- Extracts text from images through the `/api/ocr` endpoint.
- Lets readers chat with an AI assistant about the loaded manuscript.
- Generates quick "magic" insights from the current page or selected context.
- Supports translation from selected text.
- Saves books, reading progress, bookmarks, highlights, and collection metadata.
- Uses Firebase Authentication for anonymous, email/password, and Google sign-in.
- Syncs library data and reader annotations with Firestore.

## Tech Stack

- Frontend: React 19, Vite, Tailwind CSS, Lucide React
- Backend API: Flask, Flask-CORS
- AI provider: Groq
- Database/Auth: Firebase Authentication + Firestore
- Deployment: Vercel

## Project Structure

```text
.
+-- api/
|   +-- index.py          # Flask API for chat, OCR, and PDF parsing
+-- src/
|   +-- App.jsx           # Main app UI and reader logic
|   +-- lib/firebase.js   # Firebase setup
|   +-- components/       # UI components
+-- .env                  # Local environment variables
+-- package.json          # Frontend scripts and dependencies
+-- requirements.txt      # Python dependencies for the API
+-- vercel.json           # Vercel rewrites and headers
+-- vite.config.js        # Local dev server + /api proxy
```

## Environment Variables

Create a `.env` file with:

```env
GROQ_API_KEY=your_groq_api_key

VITE_FIREBASE_API_KEY=your_firebase_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_FIREBASE_MEASUREMENT_ID=your_measurement_id

VITE_APP_ID=novel-quest-v1
```

Notes:

- `GROQ_API_KEY` is required for `/api/chat` and `/api/ocr`.
- All `VITE_FIREBASE_*` variables are required for auth and Firestore sync.
- `VITE_APP_ID` is used to namespace app data.
- Vite embeds `VITE_*` variables at build time. If you change Firebase keys, restart `npm run dev` locally and redeploy production so old bundles do not keep using stale values.

## Local Development

### 1. Install frontend dependencies

```bash
npm install
```

### 2. Create and activate a Python environment

```bash
python -m venv venv
```

Windows PowerShell:

```powershell
.\venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source venv/bin/activate
```

### 3. Install backend dependencies

```bash
pip install -r requirements.txt
pip install pymupdf4llm
```

`pymupdf4llm` is used by `api/index.py` for PDF-to-Markdown parsing, but it is not currently listed in `requirements.txt`.

### 4. Run the Flask API

```bash
python api/index.py
```

The API runs on `http://127.0.0.1:5000`.

### 5. Run the frontend

```bash
npm run dev
```

The Vite dev server runs on `http://localhost:5173` and proxies `/api` requests to the Flask server.

## API Endpoints

### `POST /api/chat`

Chats against the loaded manuscript context using Groq.

Request body:

```json
{
  "systemPrompt": "You are a helpful assistant.",
  "prompt": "What happens in chapter one?",
  "context": "manuscript text",
  "mode": "strict"
}
```

### `POST /api/ocr`

Extracts readable text from one or more base64-encoded images.

Request body:

```json
{
  "images": [
    {
      "base64": "....",
      "mimeType": "image/jpeg"
    }
  ]
}
```

### `POST /api/parse_pdf`

Accepts a multipart upload with a `file` field and converts the PDF into Markdown/text.

## Deployment

The repo is already set up for Vercel:

- `vercel.json` rewrites `/api/chat`, `/api/ocr`, and `/api/parse_pdf` to `api/index`.
- All other routes rewrite to `index.html` for the SPA.

Before deploying, make sure these environment variables are configured in Vercel:

- `GROQ_API_KEY`
- `VITE_FIREBASE_API_KEY`
- `VITE_FIREBASE_AUTH_DOMAIN`
- `VITE_FIREBASE_PROJECT_ID`
- `VITE_FIREBASE_STORAGE_BUCKET`
- `VITE_FIREBASE_MESSAGING_SENDER_ID`
- `VITE_FIREBASE_APP_ID`
- `VITE_FIREBASE_MEASUREMENT_ID`
- `VITE_APP_ID`

## Current Caveats

- `requirements.txt` does not yet include `pymupdf4llm`, even though the API imports it at runtime.
- The backend currently uses Groq model names directly inside `api/index.py`, so model swaps require a code change.
- Firebase auth and Firestore rules must be configured correctly in your Firebase project or sign-in/data sync will fail.

## Scripts

```bash
npm run dev
npm run build
npm run preview
npm run lint
```
