# Notifications

Two transports, one set of preferences. A browser gets Web Push (VAPID); the Android and iOS apps
get Firebase Cloud Messaging, because an Android web view has no `PushManager`. The user sees one
switch (Settings → Reminders), and the server decides *when* in exactly the same way for both
(`lib/reminders.ts`, `lib/adhan-push.ts`).

## What gets sent

| | When | Priority |
| --- | --- | --- |
| Daily reminder | Once a day, at the chosen slot in the user's own time zone, rotating between five texts | normal |
| Adhan nudge | At each chosen prayer's time, one line on why to pray | high |
| Fasting reminder | The evening before a voluntary fast the user asked about | normal |

No counts, no streak warnings, no "you're falling behind". Quiet hours are respected, and turning
the switch off deletes the row — nothing is kept to send later.

## Web Push (already live)

`NEXT_PUBLIC_VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT`. Nothing to do here.

## FCM, for the apps

The native half stays inactive until `FCM_SERVICE_ACCOUNT` is set, so the site works exactly as
before while you set this up.

1. **Create a Firebase project** at <https://console.firebase.google.com> — one project, any name.
   Google Analytics is not needed.
2. **Add the Android app.** Package name `com.ulamat.app` (it must match `appId` in
   `capacitor.config.ts`). Download `google-services.json` and put it at
   `android/app/google-services.json`. It is gitignored: it belongs to your Firebase project, not
   in the repository. The Gradle plugin is applied only when that file exists, so an Android build
   without it still succeeds (push simply does nothing).
   - For signed builds, add the app's SHA-1 and SHA-256 under Project settings → Your apps. The
     same fingerprints go in `ANDROID_CERT_FINGERPRINTS` for deep links (`docs/NATIVE_PHASE.md`).
3. **Add the iOS app** if you are shipping to iOS: bundle id `com.ulamat.app`, download
   `GoogleService-Info.plist` into `ios/App/App/`, and upload an APNs key (Apple Developer →
   Keys → Apple Push Notifications service) under Project settings → Cloud Messaging. iOS push
   needs a paid Apple Developer account; Android does not.
4. **Create a service account** — Project settings → Service accounts → Generate new private key.
   A JSON file downloads.
5. **Put it in Vercel** as `FCM_SERVICE_ACCOUNT`, for Production only. Paste the JSON as-is, or
   base64-encode it first if the newlines in `private_key` give you trouble:
   `base64 -w0 service-account.json`. Both are accepted.
6. **Redeploy.** The next cron run picks up `device_tokens`.

The server never stores this key: it signs a short-lived JWT with it, exchanges that for an access
token good for an hour, and caches that in memory (`lib/fcm.ts`).

## Checking it works

- On the phone, Settings → Reminders. The OS asks for permission, and a row appears in
  `device_tokens` with your `user_id`.
- Force a send: `curl -H "Authorization: Bearer $CRON_SECRET" https://<site>/api/cron/reminders`.
  The response is `{"sent":n,"removed":m}`. `sent: 0` with a row present means it is not due yet —
  a reminder goes out once per local day, at or after the slot hour.
- Nothing arrives, `sent` counted it: check the Firebase console → Cloud Messaging for delivery
  reports, and that the app was built *after* `google-services.json` was added.
- `removed` climbing: tokens are being rejected. Usually the `google-services.json` in the build
  belongs to a different Firebase project than `FCM_SERVICE_ACCOUNT`.

## Notes

- Messages carry both a `notification` block (so the OS shows them while the app is closed) and the
  same text in `data` (so the app can show it in-app and know which page a tap should open). The
  text is written on the server in the user's language — the app does not translate anything.
- The status-bar icon is `android/app/src/main/res/drawable/ic_stat_notify.xml`, tinted with
  `notify_accent`. Android needs a flat white silhouette; a coloured icon comes out as a grey box.
- No badge counts. An unread number is a score, and there is nothing here to chase.
