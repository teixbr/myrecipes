# MyRecipes — instructions for Claude

A personal iPhone recipe app, built by Bryan as a Christmas 2026 gift for his girlfriend.
Read `docs/HANDOFF.md` before doing anything: it holds every product and design decision so far.

## Hard rules
- **No running costs.** No servers, no paid APIs, no subscriptions. Everything runs on the phone.
  Gemini was ruled out: its free tier can't be used by apps serving UK users.
- **Offline first.** Only importing from a website or a social post needs the internet.
- **Design direction D** (sage, linen, oat, soft serif). Never hard-code colours; use the theme tokens,
  because the user can switch themes in Settings.
- Bryan's Mac is **Intel**, so the newest Xcode he can use is **Xcode 26.x** (iOS 26 SDK).
  Do not use iOS 27-only APIs. App Store Connect accepts Xcode 26 builds until April 2027.
- Target device: **iPhone 15** (no Apple Intelligence, so no Foundation Models framework).
  Test device: Bryan's iPad, so keep the app universal.

## Stack
SwiftUI · SwiftData · Vision (OCR) · Speech (`SpeechAnalyzer`) · AlarmKit (timers) · ZIPFoundation (export).
Optional later: a small on-device model via MLX Swift for parsing.
