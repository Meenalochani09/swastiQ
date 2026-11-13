# SwastiQ — Vercel deployment (no GitHub required)

SwastiQ is a static browser app with server-side API routes for Gnani Prisma speech-to-text and Gnani Timbre text-to-speech.

## Project structure

- `public/index.html` — SwastiQ web app
- `api/stt.js` — microphone WAV audio → Gnani Prisma → transcript
- `api/tts.js` — text → Gnani Timbre → playable audio
- `api/status.js` — reports whether voice services are configured (never returns the API key)
- `api/state.js` — optional cloud backup when Upstash Redis environment variables are configured
- `local-server.js` — local development server for `http://localhost:8000`

## Deploy directly from your computer without GitHub

This project includes Vercel serverless API routes. Use the Vercel CLI deployment (not a static-only drag-and-drop host), so `/api/stt` and `/api/tts` are deployed too.

1. Install Node.js 20 or newer.
2. Extract this ZIP into a folder.
3. Open a terminal in that folder.
4. Run `npx vercel login` and complete the browser sign-in.
5. Run `npx vercel` and answer the setup questions:
   - Set up and deploy? **Y**
   - Which scope? Choose your Vercel account.
   - Link to existing project? **N** for a new project, or **Y** if you already created one.
   - Project name: `swastiq` (or your preferred name).
   - Directory: `./`
   - Framework preset: **Other**
   - Build command: leave blank / press Enter.
   - Output directory: leave blank / press Enter.
6. Before production deployment, add your Gnani key to the Vercel project:
   - Open Vercel Dashboard → your project → **Settings → Environment Variables**.
   - Name: `GNANI_API_KEY`
   - Value: paste your actual Gnani API key.
   - Select **Production** (also select Preview and Development if you will use those environments).
   - Save.
7. In the same project folder, run `npx vercel --prod`.
8. Open the public `https://your-project.vercel.app` URL. Allow microphone access when prompted. Microphone access requires HTTPS (Vercel provides HTTPS).

If you add or change the key after deployment, redeploy with `npx vercel --prod`.

## Local testing

1. Copy `.env.example` to `.env.local`.
2. Open `.env.local` and replace the placeholder with your real key:
   `GNANI_API_KEY=your_actual_key`
3. Run:
   `npm start`
4. Open `http://localhost:8000`.

Never share `.env.local`, commit it, or include a real API key in a ZIP uploaded publicly. `.gitignore` excludes local environment files.

## Vercel environment variables

Required:
- `GNANI_API_KEY` — the API key created in your Gnani dashboard.

Optional:
- `GNANI_STT_MODEL` — defaults to `prisma-v3.0`.
- `GNANI_TTS_MODEL` — defaults to `timbre-v2.5`.
- `GNANI_TTS_VOICE` — defaults to `Nalini`.
- `GNANI_TTS_LANG` — defaults to `en-IN`.

The same `GNANI_API_KEY` is used by the backend STT and TTS routes. It is never exposed to the browser. The browser calls only the same-origin `/api/stt` and `/api/tts` endpoints.

## Notes

- The microphone records short WAV clips and sends them to `/api/stt`.
- The app stops a recording at approximately 28 seconds; the backend also enforces a request size limit.
- The browser microphone works on localhost and on HTTPS deployment URLs, not on an ordinary insecure HTTP public URL.
- `api/state.js` is optional. Without Upstash Redis environment variables, app data remains in that browser's local storage.
- Vercel serverless functions do not provide a persistent WebSocket server. This project uses REST for voice transcription and synthesis.
