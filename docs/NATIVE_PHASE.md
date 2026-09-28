# Native app

The Android and iOS apps are in this repository: `android/` and `ios/`, built with **Capacitor 8**. The shell loads the deployed site (the app needs a server for the recitation check, sync and the admin panel) and adds what a browser tab cannot.

## What the shell adds

| Native | Where |
|---|---|
| No public website in the app: onboarding → account → lessons, and nothing to sell | `lib/native-request.ts`, `proxy.ts` |
| Splash screen, app icons (all densities, adaptive on Android) | `native/assets` → `npm run native:assets` |
| Offline screen in five languages, retries by itself when the device is back | `native/www/index.html` |
| Android back button: goes back in the app, leaves from the first screen | `components/NativeBridge.tsx` |
| Deep links: `https://<host>/…` and `ulamat://…` open the right page | `NativeBridge` + the manifest / Info.plist |
| System share sheet (ayah, invite link, progress card) | `lib/native.ts` → `share()` |
| Haptics through the OS engine instead of a raw vibration | `lib/haptics.ts` |
| Status bar follows the theme | `NativeBridge` |
| Links to other sites open in the system browser, with its address bar | `NativeBridge` |
| Reminders through FCM, since an Android web view has no Web Push | `lib/fcm.ts`, `lib/native-push.ts` → `docs/PUSH.md` |
| Home-screen widget: next prayer and today's goal | `android/…/widget`, `lib/widget.ts` |
| Microphone, location and notification permissions explained in the OS prompt | `AndroidManifest.xml`, `Info.plist` |
| No cloud backup or device transfer of on-device data (journal, check-ins) | `android/app/src/main/res/xml/data_extraction_rules.xml` |

`lib/native.ts` is a no-op in a browser, so one build runs on the web and in the app.

## Working on it

```bash
npm run native:sync        # after any dependency or config change
npm run native:assets      # regenerate icons and splash from native/assets/icon.png
npm run native:android     # open Android Studio
npm run native:ios         # open Xcode (macOS only)
```

Point the shell somewhere else while developing:

```bash
NATIVE_URL=http://192.168.1.20:3000 npm run native:sync
```

Building an APK/AAB needs Android Studio (or the command-line SDK) — `cd android && ./gradlew assembleRelease`. Building for iOS needs a Mac with Xcode.

## Before the stores

1. **Domain.** Change `app_host` in `android/app/src/main/res/values/strings.xml` and `NEXT_PUBLIC_SITE_URL`, then `npm run native:sync`.
2. **Android App Links.** Set `ANDROID_CERT_FINGERPRINTS` (the SHA-256 of your Play App Signing key, and your upload key while testing, comma-separated) and `ANDROID_PACKAGE`. The site serves `/.well-known/assetlinks.json` from them.
3. **iOS Universal Links.** Set `APPLE_APP_ID` to `<Team ID>.com.ulamat.app`; the site serves `/.well-known/apple-app-site-association`. Add the Associated Domains capability (`applinks:<host>`) in Xcode.
4. **Signing.** Android: an upload key, then Play App Signing. iOS: a distribution certificate and provisioning profile.
   Keep the keystore **outside this checkout** (for example `C:\Users\<you>\keystores\ulamat-upload.jks`, or `~/keystores/` on macOS and Linux). `.gitignore` also refuses `*.jks`, `*.keystore` and `keystore.properties`, but that is the second line of defence, not the plan: a key that reaches the repository lets anyone publish an update as you, and Play cannot rotate an upload key once the app is live. Back it up somewhere you will still have in five years, with its passwords.
5. **Store listing.** Screenshots come from `npm run gallery:shots`; the privacy answers come from the privacy policy and `docs/DATA_AND_API.md` (no data is sold, nothing is stored from the microphone).

Apple rejects apps that are only a web page in a box, so keep the native parts above (and add push, below) before submitting.

## The website is not in the app

Someone who installed the app has already been sold on it, so the landing page, the feature pages, the FAQ and the press page are not served to it — they would be dead ends behind a bottom navigation bar. The shell appends `UlamatApp` to its user agent (`appendUserAgent` in `capacitor.config.ts`), the proxy reads it before any page is built, and a site path in the app redirects to wherever that person belongs: `/welcome`, `/login`, or `/today`.

- `lib/native-request.ts` — `isNativeRequest(ua)` and the one exception, `allowedInApp`.
- **The legal pages stay reachable** (`/privacy`, `/terms`, `/delete-account`, …): Settings links to them, the consent text points at them, and both stores require a privacy policy that opens from inside the app.
- A browser is untouched: the same deployment serves the full website as before.
- Changing the marker means changing it in both places, and rebuilding the app — an old build keeps the old user agent.

## When the app sits on the logo

The splash hides itself after three seconds, so this should not happen any more — but if a launch
still ends on a blank or an offline screen, it is the remote page, not the shell:

1. **Is the site up?** `curl -I https://ulamat.com/` from your own machine. The app loads that URL
   and nothing else; if it 404s or the certificate is not ready yet, there is nothing to show.
2. **Does the build point where you think?** `android/app/src/main/assets/capacitor.config.json`
   holds the URL that was compiled in. An APK built before the domain was attached still asks for
   the old host.
3. **Watch it launch.** `NATIVE_DEBUG=1 npx cap sync android`, rebuild, then open `chrome://inspect`
   on a desktop with the phone plugged in: the web view's console says what failed.
4. **Ask the phone.** `adb logcat | grep -i capacitor` shows the load error directly.

A failed load now shows `native/www/index.html` — the offline screen, in five languages, which
retries by itself — rather than the web view's own error page.

## Notifications

Done — `docs/PUSH.md` has the Firebase steps. The switch is the same one the browser uses (Settings → Reminders); inside the app it registers an FCM token instead of a Web Push subscription, and the permission prompt appears when the switch is turned on, never on first launch. Inactive until `FCM_SERVICE_ACCOUNT` is set.

## Home-screen widget (Android)

A 3×2 widget: the next prayer with a countdown, and a bar for today's goal. Tap opens Today.

- `android/app/src/main/java/com/ulamat/app/widget/` — `UlamatWidgetProvider` (draws it), `WidgetState` (what to draw), `WidgetPlugin` (the app's only way in, registered in `MainActivity`).
- Plain `RemoteViews` and Java, not Jetpack Glance: Glance would pull Kotlin and Compose into a project that has neither, for a widget with two lines of text and a bar.
- **No network, ever.** It draws what the app last wrote. `components/WidgetSync.tsx` pushes new text whenever the next prayer or today's minutes change; Android also redraws it every 30 minutes (the shortest period it honours), and the countdown is recomputed from the stored timestamp at each redraw.
- **Every string arrives translated**, in the user's digits, with the direction set — the widget has no idea what language it is in. The two countdown templates keep `{h}` and `{m}` for it to fill (`lib/widget.ts`). Before the app has been opened once there is no language to use, so it shows "Open the app once" from `strings.xml`.
- A home screen is read by whoever picks up the phone, so nothing else goes on it: no streak, no name, no history.
- Light and dark from `values/colors.xml` and `values-night/colors.xml`, since a launcher cannot use the app's theme.

**Not on iOS.** A WidgetKit widget is a separate Swift target with an App Group, not something Capacitor can carry. Worth doing, but it is its own piece of work.

### Later, if they earn their keep

- **The medium widget:** today's five prayers as circles, tapping one opens the check-in.
- **Fajr alarm with a recited dua.** iOS `AlarmKit` or a local notification with a custom sound; Android `AlarmManager.setAlarmClock` with a full-screen intent.
- **Offline speech.** On-device recognition would let the recitation check work with no connection.

---

## Original notes (before the shell existed)

## 1. Home-screen widgets

- **iOS:** WidgetKit (SwiftUI) extension. **Android:** Jetpack Glance app widget.
- Small: next prayer + countdown. Medium: five prayer circles for today (read-only; tap opens the check-in).
- Data: the app writes `{ times, checkins, locale }` to a shared container (iOS App Group `UserDefaults`, Android `DataStore`) whenever the store changes. Widgets never call the network.
- Refresh: timeline entries at each prayer time (iOS), `WorkManager` at each prayer time (Android).
- Localized digits and RTL come from the saved locale. No faces, no music.

## 2. Fajr alarm with recited dua

- Real alarm, not a push: iOS `AlarmKit` (iOS 26+) or a critical-alert-free local notification with a custom sound; Android `AlarmManager.setAlarmClock` + full-screen intent.
- The sound is a **recited dua or the adhan by a reciter** (voice only, no instruments). Files bundled, not streamed.
- Dismiss screen shows the waking-up dua (Bukhari 6312) with meaning, and a large "I'm up" button (≥ 44 px). Snooze max 2 × 5 min, worded gently.
- Times come from the same `lib/prayer.ts` calculation and offsets, recalculated daily and after travel.

## 3. Masjid finder

- Background location is **not** needed; only when-in-use.
- Source options: OpenStreetMap Overpass (`amenity=place_of_worship` + `religion=muslim`) — free, no key; or Google Places (costs, needs key). Start with OSM, cache per area for a week.
- List sorted by distance, with a map pin sheet and "Open in Maps". Community-edited jama'ah times are out of scope for v1; show the masjid note that local times win.
- Privacy: coordinates stay on the device; the Overpass query sends only a rounded bounding box.

## 4. Halal scanner

- Barcode via the native camera plugin (ML Kit / VisionKit), then look up ingredients in Open Food Facts.
- Show **ingredients and flagged items only** (e.g. gelatin, carmine/E120, alcohol, lard), each with "why this may matter". The app never says "halal" or "haram" for a product — certification bodies do; show a certification logo only if the data has one.
- Flag list lives in content and goes through scholar review like other rulings (madhab differences on seafood, alcohol traces, etc.).

## Order

1. Capacitor shell + widgets (highest daily value). 2. Fajr alarm. 3. Masjid finder. 4. Halal scanner (needs the most review).
