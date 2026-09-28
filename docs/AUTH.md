# Signing in

Onboarding first, then **five days of learning with no account at all**, then a name, an email and
a password, once. After that the password is all it takes — nothing to wait for in an inbox every
time.

## The five free days

Asking for an account before anyone has seen a lesson is asking a stranger to commit. Asking on the
sixth day is asking someone who has already learnt something, and who now has progress that only an
account can carry to another phone.

`lib/auth-gate.ts` is the whole rule, and it is pure, so it is tested rather than guessed at. The
proxy stamps `ilm-since` on the first request from a device and reads it afterwards. A clock set
forward or a hand-edited cookie buys nothing: a first visit in the future counts as today.

Onboarding, sign-in, invite links and the notice pages are open at every point. So is everything
when Supabase is not configured — there is nothing to sign in to.

Inside the app the website is not served at all, so the first launch lands on onboarding
(`lib/native-request.ts`).

| | When |
| --- | --- |
| **Password** | Every sign-in |
| **Email code** | Confirming the address at sign-up, and resetting a forgotten password |
| **Google** | One tap, for whoever prefers it |
| **PIN** | Opens the app on a device that has already signed in |

## Why a code and not a link

A link has to be opened in the same browser that asked for it. On a phone that is rarely true: the
mail app opens its own web view, the link lands in a session that has no idea what was happening,
and inside our own app it leaves the app entirely. A code is six digits typed where the question
was asked, and behaves the same in a browser and in the app.

- Sign-up: `signUp` with the name in `options.data`, then `verifyOtp` with `type: "signup"`.
- Forgotten password: `resetPasswordForEmail`, `verifyOtp` with `type: "recovery"`, then
  `updateUser({ password })`.

### Supabase needs these changes

**Authentication → Email Templates**, for both **Confirm signup** and **Reset password**. The
default body has `{{ .ConfirmationURL }}`; it has to offer the code instead:

```html
<h2>Your code</h2>
<p>Enter these six digits in the app:</p>
<p style="font-size:28px;letter-spacing:6px;font-weight:700">{{ .Token }}</p>
<p>The code works for one hour. If you didn't ask for it, ignore this email.</p>
```

Without that edit people get a link they cannot type into the code screen.

**Authentication → Providers → Email**: confirm email on, OTP expiry an hour or less, minimum
password length 8, leaked-password protection on.

## Bots

**Authentication → Settings → Bot and Abuse Protection**: Turnstile, with the secret from
Cloudflare. The site key goes in `NEXT_PUBLIC_TURNSTILE_SITE_KEY`, and `components/auth/Turnstile.tsx`
renders the widget on the three screens that send email or guess passwords — sign-up, sign-in and
the reset request. Supabase verifies the token; we only pass it.

The lessons are free and public, so this is not about hiding them. It is that a sign-up form
anyone can post to becomes a way to send email to strangers from our address, and a password form
anyone can post to becomes a way to guess passwords all day. With no site key set, the widget
renders nothing and everything still works.

## The PIN is a lock, not a password

Six digits is a million guesses. A server can be asked a million questions; a phone in your pocket
cannot. So the PIN never reaches Supabase:

- First sign-in on a device is the password (or Google). The session is stored by Supabase's
  client, as always.
- The PIN is then offered once (`PinOffer`), and set in Settings any time after (`PinRow`).
- `PinLock` wraps the app. It asks once per open — `sessionStorage`, so moving between pages and
  refreshing do not ask again, and closing the app does.
- Ten wrong tries signs the session out and sends you back to the password screen. That is the case that
  matters: the phone is already unlocked and in someone else's hands.
- A new phone has no session, so it starts again with the password. There is nothing to "recover".

What is stored on the device is PBKDF2-SHA-256 over the digits, 210,000 iterations, with a random
16-byte salt — never the digits. The comparison is constant-time. Obvious PINs (all one digit, a
run, the handful everyone picks) are refused when setting one: a lock that opens on `123456` is
decoration. `lib/pin.ts`, tested in `tests/pin.test.ts`.

## Google

`signInWithOAuth` with `redirectTo` at `/auth/callback`, which exchanges the code for a session and
records the Terms acceptance. In Supabase: Authentication → Providers → Google, with the client id
and secret from the Google Cloud console, and `https://<project>.supabase.co/auth/v1/callback` as
the authorised redirect URI.

## What to check after changing the domain

Authentication → URL Configuration: Site URL and the redirect allow-list. Codes do not care, but
Google and every link in an email do.
