# Frontend (Next.js)

This package contains the CortexFlow interaction layer and visualization workspace.

Joint collaboration repository by the MnM group.

Private codebase. Licensing context is defined in [../LICENSE](../LICENSE).

## Stack

- Next.js App Router
- React + TypeScript
- Tailwind CSS v4
- Recharts + Three.js
- Gemini API (Transcription and Analysis)

## Environment Variables

`.env.local` fields:

```env
GEMINI_API_KEY=
GEMINI_API_BASE=https://generativelanguage.googleapis.com/v1beta
GEMINI_TRANSCRIBE_MODEL=gemini-2.0-flash
BACKEND_URL=http://localhost:8000
NEXT_PUBLIC_BACKEND_WAKE_ENABLED=true

# Firebase client
NEXT_PUBLIC_FIREBASE_API_KEY=
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
NEXT_PUBLIC_FIREBASE_PROJECT_ID=
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
NEXT_PUBLIC_FIREBASE_APP_ID=

# Firebase admin
FIREBASE_PROJECT_ID=
FIREBASE_CLIENT_EMAIL=
FIREBASE_PRIVATE_KEY=

# Supabase
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
```

## API Surface

- `POST /api/transcribe`: audio upload and Gemini API transcription
- `POST /api/analyze`: proxy to backend analysis endpoint at `BACKEND_URL`
- `POST /api/wake-backend`: best-effort health ping to reduce cold-start delay
- `GET /api/wake-backend`: backend readiness status for launch-screen gating
- `POST /api/account/bootstrap`: Firebase-authenticated account upsert into Supabase
- `GET /api/reports`: fetch signed-in user reports from Supabase
- `POST /api/reports`: persist a report for the signed-in user
- `DELETE /api/reports/[id]`: delete one report for signed-in user
- `DELETE /api/reports/clear`: clear signed-in user report history

## Supabase SQL Setup

Run the SQL in `supabase/schema.sql` inside your Supabase SQL editor before starting the app.

## Hosting Profile

- Frontend platform: Vercel
- Vercel root directory: `frontend`
- Backend target: Hugging Face Spaces (Docker)

Typical Vercel variables:

- `GEMINI_API_KEY`
- `GEMINI_TRANSCRIBE_MODEL`
- `BACKEND_URL`
- `NEXT_PUBLIC_BACKEND_WAKE_ENABLED`

## Local Development Reference

```bash
npm ci
npm run dev
```

Create `frontend/.env.local` manually with the variables listed above before running the dev server.

## Architectural Constraints (Gemini API)

While this codebase is optimized specifically for the Gemini API, there is a known limitation regarding data accuracy for word-level timestamps:

> **Hallucinated Word Timestamps (Data Accuracy Issue)**
> You are asking the Gemini model to return strict `{word, start, end}` timestamps. While Gemini is fantastic at understanding audio content and transcribing it, it is a multimodal LLM and not a dedicated Automatic Speech Recognition (ASR) engine. LLMs notoriously hallucinate exact timestamp alignments.
>
> **The Impact:** Gemini will either invent inaccurate millisecond timestamps or simply return an empty array (as instructed if unsure). This means the `extractPauseMap` function will either process garbage data or nothing at all.
>
> **The Fix for SOTA:** If precise word-level timestamps and pause mapping are crucial for your application, you should route this specific task through a dedicated ASR engine like Deepgram or OpenAI's Whisper, which are heavily optimized for exact millisecond-level word alignments. However, due to hackathon constraints, this project explicitly relies on Gemini.
