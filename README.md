# Feely

A minimalist, **local-first** feeling tracker — a native **iOS** app built with
React and wrapped in Capacitor.

Log how you feel on a 0–10 scale, tag it, add a note, and watch the trend across
days, weeks, and months. Your entries stay on your device — no accounts, no
servers, no tracking. The only thing the app sends is feedback you write
yourself (see [Data & privacy](#data--privacy)).

> This is the **iOS branch**: a pure native build with the web/PWA paths removed.
> The `main` branch keeps the dual PWA + iOS codebase.

## Features

- **Record** — 0–10 feeling score; custom, colour-coded tags (a "New tag" sheet
  with a five-colour picker; optionally grouped into **Mood**, **Activity**, and
  **Body & Health**); and a free-text note. One tap to check in, from either the
  header pill or the full-width button at the foot of the form, then a
  confirmation card with your 7-day average and 30-day check-in rate.
- **Chart** — daily / weekly / monthly trend line with an average score, a
  delta pill versus the previous period, a spread band, point sizes that reflect
  data reliability, a drag-to-scrub inspector, and a tag-distribution breakdown
  split by group. Plus a **check-in rate** card — the share of the last 30 days
  you logged, with the all-time rate under it — opening a sheet with the streaks,
  both rates, and a four-week heat-map.
- **Data** — full history, newest first. Tap a record to edit, swipe left to
  delete. Backdate entries to any date/time.
- **Settings** — light / auto / dark theme, **language** (English / Čeština /
  Suomi), configurable start-of-day boundary, a **daily reminder** that nudges
  you at a chosen time on any day you haven't logged, CSV export (date-ranged),
  and JSON backup & restore.
- **Feedback** — a button in the Settings header opens a bottom sheet to send
  ideas, bug reports, or a note; it emails the message via
  [Web3Forms](https://web3forms.com) (see [Configuration](#configuration)).
- **Localized** — full UI in English, Czech, and Finnish, with locale-aware date
  formatting; the choice is persisted.
- **Native feel** — real haptics through the iOS feedback generators (see
  [`FEATURES.md` §10](./FEATURES.md)), a native launch screen, a native action
  sheet for the per-entry delete, bottom sheets (streak, edit, feedback, new tag,
  and the reset & restore confirmations) with drag-to-dismiss, and CSV/JSON export
  through the system share sheet (Files, Mail, AirDrop).

## Design

A green-header, two-accent look (green + a coral/orange secondary) with a warm
canvas, self-hosted **Inter** for body copy and **Fredoka** for the wordmark,
score numbers, and section headings, plus soft layered shadows — all
theme-aware across light and dark. Colours live as CSS variables in
`src/index.css`.

## Tech stack

- [React 19](https://react.dev/) + TypeScript, built with [Vite 8](https://vite.dev/)
- [Capacitor 8](https://capacitorjs.com/) for the native iOS shell, with
  [`@capacitor/haptics`](https://capacitorjs.com/docs/apis/haptics),
  [`@capacitor/splash-screen`](https://capacitorjs.com/docs/apis/splash-screen),
  [`@capacitor/filesystem`](https://capacitorjs.com/docs/apis/filesystem) +
  [`@capacitor/share`](https://capacitorjs.com/docs/apis/share) for file export,
  [`@capacitor/local-notifications`](https://capacitorjs.com/docs/apis/local-notifications)
  for the daily reminder,
  [`@capacitor/action-sheet`](https://capacitorjs.com/docs/apis/action-sheet)
  for destructive confirmations,
  [`@capacitor/keyboard`](https://capacitorjs.com/docs/apis/keyboard),
  [`@capacitor/app`](https://capacitorjs.com/docs/apis/app) for foreground
  re-sync, and
  [`@capacitor/device`](https://capacitorjs.com/docs/apis/device) for the context
  attached to feedback
- Apple's [Foundation Models](https://developer.apple.com/documentation/foundationmodels)
  (iOS 26+, on-device only, weak-linked) for the written week / month summary
- [Tailwind CSS v4](https://tailwindcss.com/), with self-hosted
  [Fredoka](https://fonts.google.com/specimen/Fredoka) (`@fontsource`) for the
  wordmark — bundled rather than CDN-loaded so the app renders identically
  offline and makes no third-party requests
- [Dexie](https://dexie.org/) over IndexedDB for local storage, with
  [dexie-react-hooks](https://dexie.org/docs/dexie-react-hooks/useLiveQuery()) for
  reactive (live) queries
- [@tanstack/react-virtual](https://tanstack.com/virtual) for list virtualization
- [Vitest](https://vitest.dev/) + [Testing Library](https://testing-library.com/)
  (jsdom, fake-indexeddb) for unit and component tests

## Getting started

```bash
npm install
npm run build:ios    # build the web assets and sync them into the iOS project
npx cap open ios     # open in Xcode, then Run (⌘R)
```

## Configuration

The **Feedback** form posts to [Web3Forms](https://web3forms.com). Copy
`.env.example` to `.env` and set your access key:

```bash
cp .env.example .env
# then edit .env:
VITE_WEB3FORMS_ACCESS_KEY=your-web3forms-access-key
```

`.env` is git-ignored. The key is a client-side Web3Forms key (public by design)
and gets inlined into the build; while it's unset, the Feedback button still
opens but sending fails with a clear message.

## Scripts

| Command           | Description                                  |
| ----------------- | -------------------------------------------- |
| `npm run build`   | Type-check (`tsc -b`) and build to `dist/`   |
| `npm run build:ios` | Build, then `cap sync` the assets into the iOS project |
| `npm run lint`    | Run ESLint                                   |
| `npm run test`    | Run the Vitest suite                         |
| `node scripts/generate-icons.mjs` | Regenerate the iOS app icon and launch-screen art from `public/favicon.svg` |

`tsc -b` type-checks the tests too, via the `tsconfig.test.json` project
reference; `tsconfig.app.json` on its own excludes them. Both run under
`strict`, without which the codebase's `!` assertions go unchecked.

Bump the version in **both** `package.json` and Xcode's `MARKETING_VERSION` —
Settings reads the first, feedback reports read the second, and
`src/tests/version.test.ts` fails if they drift.

## iOS app

A [Capacitor](https://capacitorjs.com/) shell around a React web view. It targets
**iPhone only, portrait**, iOS 15+.

Requires Xcode and, to run on a device or submit, an Apple Developer account.
Dependencies resolve through Swift Package Manager; there is no CocoaPods step.

`cap sync` copies `dist/` into the iOS project, so re-run `build:ios` after any
change to the React code — Xcode alone won't pick it up.

Haptics only work on a **real device**; the Simulator has no Taptic Engine.

### App icon

The 1024px icon is rendered from `public/favicon.svg` by `node
scripts/generate-icons.mjs`, which also emits a **dark** variant (iOS 18+) — the
mark on the app's dark canvas with a light F — registered under a `dark`
luminosity appearance in the `AppIcon` asset catalog, so the home-screen icon
isn't a bright white square in dark mode.

### Launch screen

A static asset-catalog image with light and dark variants, generated from the
same source as the icon by `node scripts/generate-icons.mjs`. It resolves against
the **system** appearance, not the app's own theme setting — nothing of ours is
running yet when iOS draws it.

The splash itself is held until the app paints, via `src/lib/splash.ts`. Setting
`launchAutoHide: true` in `capacitor.config.ts` makes that module dead code — the
plugin's own timer dismisses the splash first.

### File export

Anything that produces a file must go through `src/lib/saveFile.ts`, which
downloads on the web and shares the file on native. A plain `<a download>` looks
like it works everywhere, but WKWebView ignores it silently — no file, no error.

### Daily reminder

Native only, off by default, 9 PM. Enabling it is what triggers the iOS
permission prompt — iOS asks once per install, so don't move that request to
startup. Notifications only fire on a **real device or a booted Simulator**, and
the queue is a rolling window of dates rebuilt on foreground; see
[`FEATURES.md` §6.5](./FEATURES.md).

### Destructive confirmations

Irreversible actions confirm before running. **Deleting an entry from the edit
sheet** uses a native iOS **action sheet** (`src/lib/confirm.ts`) — the
platform-standard red, irreversible choice. **Restore from backup** and **Reset
App** (Settings) use a shared **confirm bottom sheet** (`ConfirmSheet.tsx`,
matching the new-tag/feedback/streak sheets) — coral tone for the irreversible
reset, green for the wanted restore — since each needs a fuller warning. Per-row
delete in the Data list stays a direct swipe, Mail-style.

### Keyboard

`@capacitor/keyboard` runs with `resize: none` (`capacitor.config.ts`), so the
web view stays full-height behind the keyboard rather than shrinking with a hard
seam. `src/lib/keyboard.ts` then handles `keyboardWillShow`: it drops the
floating nav pill, pads the page, and smoothly scrolls the focused field above
the keyboard. Bottom sheets (edit, feedback) stay anchored to the bottom — their
surface continues behind the keyboard — and grow a keyboard-tall band of bottom
padding that pushes their content up so the focused field clears the keyboard.
The padding is animated, so the sheet glides up on show and back down on hide
instead of snapping.

While any modal/sheet is open the page is pinned with `position: fixed`
(`Modal.tsx`), not just `overflow: hidden` — on iOS the latter doesn't stop
WKWebView from scrolling the tab underneath to reveal a focused input, which
drags the fixed overlay (a flash at the top) and leaves the page scrolled. The
scroll offset is captured on open and restored on close.

### Dynamic Type

WKWebView ignores the iOS text-size setting, so `src/lib/dynamicType.ts`
measures `-apple-system-body` and publishes the ratio as `--dt-scale`; the root
font size multiplies by it and every type size is in `rem`. **Keep it that way
— a `text-[Npx]` anywhere opts that element out of scaling.** Capped at 1.5×;
accessibility sizes go to 3.1×, which this layout can't absorb. See
[`FEATURES.md` §10](./FEATURES.md).

### Reduced Motion

`prefers-reduced-motion` collapses every transition and animation to ~0ms from
one block in `src/index.css`. It has to be `!important` (several transitions are
set inline), and CSS can't reach scripted smooth scrolls, so those ask
`src/lib/motion.ts`. See [`FEATURES.md` §10](./FEATURES.md).

### Bottom navigation

The bottom nav is a custom CSS "liquid glass" pill (`BottomNav.tsx`), not a
native tab bar. A native iOS Liquid Glass tab bar was prototyped and reverted:
the available plugins render the full-width system bar and can't reproduce the
pill's scroll-collapse or narrow-screen menu, and none implement the iOS 26
`UITabBarController.tabBarMinimizeBehavior` needed to collapse the bar on scroll.
Revisit if a plugin adds that. See [`FEATURES.md` §10](./FEATURES.md).

## Data & privacy

Entries live in IndexedDB and settings in `localStorage`, inside the app's own
container; on launch the app asks WebKit to keep that storage persistent, so it
isn't the first thing evicted under disk pressure. No entry, tag, setting or
streak is ever sent anywhere, and there is no analytics or telemetry. The container lives under `Library/`, which iOS
includes in **iCloud Backup** by default — restoring a device or moving to a new
phone brings entries back. This is backup, not sync: it doesn't keep two devices
in step and isn't visible in iCloud Drive. Deleting the app deletes the data.

**There are two exceptions, both off unless you choose them.**

**Apple Health sync** (Settings → Apple Health, off by default) writes each
check-in into Apple Health as a State of Mind sample — the score only, never
tags or notes. It is one-way; Feely never reads from Health. The data stays on
the device, but Health itself syncs to iCloud when the user has Health enabled
there, so this is a real widening of where entries live and the onboarding copy
says so.

**The week / month summary** on the Chart tab reads entries *and notes* — the
only feature that does — but with Apple's on-device model (Foundation Models,
`SystemLanguageModel`), never the Private Cloud Compute one, so nothing leaves
the phone. Where the model isn't available (no Apple Intelligence, or a
language it doesn't write, such as Czech today) the card doesn't appear.

**The Feedback sheet.** Nothing is sent until you write a
message and tap Send; that message plus app version/build, commit hash,
platform, device model, OS version and language then go to
[Web3Forms](https://web3forms.com), which emails them to the developer. It is
declared in `ios/App/App/PrivacyInfo.xcprivacy` — **keep the App Store Connect
privacy answers in step with that file**, and don't reintroduce an unqualified
"nothing ever leaves your device" claim in the copy.

**Settings → JSON Backup** is the portable, user-controlled copy — round-tripping
entries, custom tags and their colours, streak days, and every setting including
the chosen language and reminder — for moving data between devices out of band.

## Documentation

See [`FEATURES.md`](./FEATURES.md) for a complete functional specification of the
app's behavior and data model.

[`PRIVACY.md`](./PRIVACY.md) is the user-facing privacy policy, for the App Store
listing. Keep it in step with this section and `PrivacyInfo.xcprivacy`.

## License

[PolyForm Noncommercial License 1.0.0](./LICENSE) — free for noncommercial use.
