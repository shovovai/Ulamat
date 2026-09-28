# Pronunciation engine

**One question:** can this user say this Arabic correctly? The check looks only at how each word
and letter sounds — the letter itself (makhraj), harakat, madd length, shaddah and sukun, and the
commonly confused pairs ح/ه, ع/ء, ق/ك, ص/س, ذ/ز, ث/س, ض/د, ط/ت, ظ/ز.

It never checks how many times something is said (ruku tasbih is checked once), dhikr counts,
salah positions or actions, rulings, or whether someone prayed.

## Shape (both engines)

`lib/pronunciation/types.ts` — `PronunciationEngine.check(audio, expectedText)` returns
`{ words: [{ text, status, letters: [{ char, status, expectedSound, heardSound?, issue? }] }], confidence }`
plus `score`, `serious`, `minor`. Letter status: `correct | wrong_sound | missing | extra | short_madd | long_madd | unclear`.
The recite screen only ever reads this shape, so switching engines never changes the UI.

**Severity:** serious = a letter became another letter, a letter/vowel was dropped or changed (meaning can
change) → "Fix this first". Minor = madd length (and later ghunnah / light tajweed) → "Polish later".

## Phase 1 — WordEngine (live)

`lib/pronunciation/word-engine.ts`. The existing Whisper Quran model (`tarteel-ai/whisper-base-ar-quran`,
`HF_ENDPOINT_URL`) or the browser recogniser gives a transcript; words are aligned (`lib/align.ts`).
Letters inherit the word result (all correct, or all "unclear"), with one addition: if the heard word
differs from the expected word **only** by confused pairs, that letter is marked `wrong_sound`. The old
fuzzy word match would have let such a word through; that was a false accept and is now caught.

Limits: a Quran-trained Whisper tends to write the correct Quran text even when a letter was said wrongly,
and it cannot hear madd length at all (see `docs/PRONUNCIATION_EVAL.md`).

## Phase 2 — LetterEngine (built, behind a flag)

`PRONUNCIATION_ENGINE=letter` + `PHONEME_ENDPOINT_URL` (optionally `PRONUNCIATION_LETTER_ITEMS=salah-fatiha,…`).

1. **G2P** (`g2p.ts`): fully voweled text → expected phones, each tied to its word and letter. Rules:
   harakat, shaddah (gemination), tanween, madd (natural 2, muttasil/munfasil 4–5, lazim 6, 'arid at a stop),
   sun/moon letters, hamzat al-wasl, idgham/iqlab of noon sakin and tanween across words, helper kasra when two
   sakins meet, ha' al-kinayah silah, the lengthened lam in "Allah", and waqf at the end of each ayah/utterance.
   Unit tests: Al-Fatiha (every ayah), Tashahhud, Durood Ibrahim (`tests/g2p.test.ts`).
2. **Phoneme recogniser**: an Arabic phoneme CTC model (wav2vec2-style) on a Hugging Face endpoint.
   Contract: `POST audio/wav → { labels, frame_ms, log_probs[T][labels] }`, labels mapped to our phone
   symbols (`<b>` = blank). A thin handler on the endpoint does the mapping.
3. **Forced alignment** (`letter-scoring.ts` `forceAlign`): Viterbi over the CTC topology → frames per phone.
4. **Scoring**: GOP per phone (mean log-prob gap to the best competitor); confusion pairs (partner beats the
   expected phone by a margin → `wrong_sound` with `heardSound`); missing (blank dominates); madd length from
   the aligned duration ÷ the reciter's own harakah (median short vowel), judged against 2 / 4–5 / 6 counts.

Thresholds live in `TUNE` and are tuned with the evaluation harness.

## Model and data choice

Hugging Face could not be queried directly from the build environment; candidates were found by web search
(September 2026). **None has a stated licence**, so none is adopted yet:

| Candidate | What | Licence | Notes |
|---|---|---|---|
| `FatimahEmadEldin/wav2vec2-xls-r-300m-iqraeval` | XLS-R 300M, CTC over 68 MSA phonemes (Halabi phonetizer) for Qur'anic mispronunciation (Iqra'Eval 2026) | not stated | Closest fit to our contract; F1 ≈ 0.20 on the shared task |
| Multi-level CTC model from *Automatic Pronunciation Error Detection and Correction of the Holy Quran's Learners* (arXiv 2509.00094) | Quran Phonetic Script (encodes tajweed), reports ~0.16% PER | not stated | Best reported accuracy; also has a segmentation dataset (`obadx/recitation-segmentation`) |
| `TBOGamer22/wav2vec2-quran-phonetics` | wav2vec2 CTC, Quran phonetics | not stated | Little documentation |
| `IbrahimSalah/Wav2vecLarge_quran_syllables_recognition` | syllables, not phonemes | not stated | Wrong unit for letter scoring |
| Datasets: `IqraEval/Iqra_train`, `IqraEval/Iqra_TTS`, `IqraEval/QuranMB.v2` | native + synthetic mispronunciations | not stated | For fine-tuning and evaluation |

**Decision:** keep Phase 2 off. Next steps: ask the authors of the first two for a licence (or fine-tune
`facebook/wav2vec2-xls-r-300m`, Apache-2.0, ourselves on permissively licensed data), deploy it behind the
contract above, record labelled clips, and switch on only when it beats Phase 1 in `docs/PRONUNCIATION_EVAL.md`.

## When the check says it cannot hear you

Most "I said it right and it did not take" comes from one of four things, in this order:

1. **The room.** A phone held at arm's length in a room with other people gives a recording where
   the recitation is a small part of a loud signal. Before anything is sent, `lib/wav.ts`
   (`prepareRecording`) removes the DC offset, cuts the silence at each end and lifts the speech to
   a full level — which is what makes a careful quiet reciter legible. It also measures the speech
   against the room (`snrDb`): under 10 dB the screen says the room is loud rather than blaming the
   reciter. The browser's own `noiseSuppression`, `echoCancellation` and `autoGainControl` are all
   asked for too.
2. **The endpoint was asleep.** `HF_ENDPOINT_URL` scales to zero, so the first recitation after a
   quiet spell meets a 503 while it wakes. `lib/speech.ts` retries once after 2.5 s; a second 503
   is reported as busy, and the practice still counts.
3. **Nothing is configured.** With no `HF_ENDPOINT_URL` the route answers `not_configured` and the
   screen says checking is not set up yet. Nothing about the recitation is wrong in that case, and
   no amount of retrying changes it — set the endpoint. `speech-server/` runs the same model on
   your own box for the price of the smallest VPS there is (`docs/SPEECH.md`).
4. **The recitation really did differ.** Then the word list shows which word, and the tips say what
   to do about it.

Inside the native app, only 2–4 apply: an Android web view has no `SpeechRecognition`, so there is
no instant in-browser check to fall back on, and every recitation goes to the server.

## Privacy

Audio is processed in memory and discarded after the request, for both engines. Only scores, word statuses
and letter-issue codes (e.g. `H>h`, `madd`) are saved. (The one exception is the Level 3 teacher review, which
the user consents to separately.)
