# The recitation checker, on your own machine

The pronunciation check needs a speech model. Hugging Face Inference Endpoints bill by the hour
whether anyone recites or not, so this is the same model behind a small service you run yourself.

`speech-server/` is the whole thing: FastAPI, `faster-whisper`, and the Quran fine-tune of Whisper
(`tarteel-ai/whisper-base-ar-quran`) — the one that hears recitation properly, rather than generic
Whisper guessing at classical Arabic.

It speaks exactly what the site already sends, so nothing in the app changes: POST a 16 kHz mono
WAV, get `{"text": "..."}` back. `HF_ENDPOINT_URL` points at it instead.

## What it costs

| | |
| --- | --- |
| Hetzner CX22 (2 vCPU, 4 GB) | about €4 a month |
| Any $5–6 VPS | the same |
| Your own PC | nothing, while it is switched on |

Fixed, whether ten people recite or ten thousand. The base model is ~150 MB and needs about 1 GB
of memory; a recitation of ten seconds takes roughly two to four seconds on two CPU cores. No GPU.

## Deploy

On a fresh Ubuntu box with Docker installed:

```bash
git clone <your repo> ulamat && cd ulamat/speech-server
export SPEECH_SECRET="$(openssl rand -hex 32)"
docker compose up -d --build
curl localhost:8000/health          # {"ok":true,"model":"tarteel-ai/whisper-base-ar-quran"}
```

The first build downloads the model and bakes it into the image, so restarts are instant.

### TLS in front

The service listens on localhost only. Vercel must reach it over https, so put Caddy in front —
two lines, and it gets a certificate by itself:

```
speech.ulamat.com {
    reverse_proxy localhost:8000
}
```

Point an `A` record at the server's IP first.

### Then, in Vercel

```
HF_ENDPOINT_URL=https://speech.ulamat.com
HF_TOKEN=<the same SPEECH_SECRET>
```

Redeploy. The names are the old ones on purpose: `lib/speech.ts` did not need changing, and one
day you might go back to a managed endpoint.

## Check it end to end

Record a few seconds of Arabic as a 16 kHz mono WAV and send it the way the site does:

```bash
curl -X POST https://speech.ulamat.com \
  -H "Authorization: Bearer $SPEECH_SECRET" \
  -H "Content-Type: audio/wav" \
  --data-binary @recitation.wav
```

`{"text":"بسم الله الرحمن الرحيم"}` means it works. Then recite in the app: a score appears
instead of "Checking is not set up yet".

## If it is slow, or wrong

- **Slow.** `SPEECH_COMPUTE=int8` is already the fast setting. Two cores is the floor; four is
  comfortable. `small` instead of `base` is more accurate and about three times slower.
- **Hears nothing.** The site sends 16 kHz mono; if you are testing by hand, check your file is
  too. `ffmpeg -i in.m4a -ar 16000 -ac 1 out.wav`.
- **Memory.** `mem_limit` in `compose.yml` is 1.5 GB. A larger model needs more.
- **Cold start.** The model loads at boot, not on the first request, and the site retries a 503
  once — so a restart costs nobody their recitation.

## Why not somewhere else

- **Vercel.** No GPU, a 250 MB bundle limit and a function timeout. Whisper does not fit.
- **In the browser.** Whisper compiles to WASM and would cost nothing to run, but it is a 40–75 MB
  download before the first check and 15–30 seconds per recitation on a cheap Android phone. Most
  of the people this app is for have exactly that phone.
- **A managed API.** Works, and every recitation leaves for someone else's server. The privacy
  policy says recordings are never stored; keeping the model on your own box is how that stays
  true by construction rather than by promise.
