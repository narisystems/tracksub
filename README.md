# TrackSub — Bill & Subscription Tracker

![Build Status](https://img.shields.io/badge/build-passing-brightgreen) ![License](https://img.shields.io/badge/license-MIT-blue) ![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?logo=flutter) ![Platform](https://img.shields.io/badge/platform-Android-3DDC84?logo=android) ![Version](https://img.shields.io/badge/version-1.0.0-orange)

> **Stop guessing what you're paying for.** TrackSub is a private, local-first subscription and bill tracker — no bank account required, no data harvesting, no surprises.

Built for people who want to know exactly where their money goes without handing over their banking credentials to do it. A drop-in replacement for Mint's subscription tracking (with CSV import), and a privacy-respecting alternative to Rocket Money.

---

## 📱 Screenshots

| Dashboard | Subscriptions | Add / Edit | Price History |
|-----------|--------------|------------|---------------|
| ![Dashboard](docs/screenshots/dashboard.png) | ![Subscriptions](docs/screenshots/subscriptions.png) | ![Add](docs/screenshots/add_edit.png) | ![History](docs/screenshots/price_history.png) |

> _Screenshots coming in v1.0.1. See the [Play Store listing](https://play.google.com/store/apps/details?id=com.tracksub.app) for current screenshots._

---

## ✨ Features

### Free vs. Premium

| Feature | Free | Premium ($3.99/mo) |
|---|:---:|:---:|
| Unlimited subscription tracking | ✅ | ✅ |
| Smart reminders before due dates | ✅ | ✅ |
| Category organization | ✅ | ✅ |
| Search and filter | ✅ | ✅ |
| JSON / CSV export | ✅ | ✅ |
| Mint CSV import | ✅ | ✅ |
| Dashboard with spending overview | ✅ | ✅ |
| **Paused subscription state** | ✅ | ✅ |
| Local-first — all data stays on device | ✅ | ✅ |
| Banner ads | Shown | **Removed** |
| Ad-free experience | ❌ | ✅ |
| Google Drive automatic backup | ❌ | ✅ |
| Price history tracking | ❌ | ✅ |

### 🔒 Privacy by Design

TrackSub never asks for your bank login, never connects to financial institutions, and never transmits your subscription data to our servers. Everything lives on your device. The optional Google Drive backup is encrypted and stored in **your** Drive account — NaRiSystems has no access to it.

### ⏸️ Paused Subscriptions

Paused a streaming service for the summer? Mark it as **Paused** — TrackSub keeps the record, silences the reminders, and excludes it from your active spending total until you're ready to resume. No other free tracker does this.

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **UI / Framework** | [Flutter 3.x](https://flutter.dev) | Cross-platform UI, targeting Android |
| **State Management** | [Riverpod 2.x](https://riverpod.dev) | Reactive, compile-safe state management |
| **Local Database** | [Drift](https://drift.simonbinder.eu) | Type-safe SQLite ORM, all data stored on-device |
| **Background Tasks** | [WorkManager](https://pub.dev/packages/workmanager) | Scheduling due-date reminders reliably |
| **Subscriptions** | [RevenueCat](https://www.revenuecat.com) | Premium billing via Google Play |
| **Ads** | [Google AdMob](https://admob.google.com) | Free tier monetization |
| **Backup** | Google Drive REST API | Optional cloud backup (Premium only) |

---

## 🚀 Getting Started

### Prerequisites

- [Flutter SDK](https://docs.flutter.dev/get-started/install) **3.19.0 or later**
- Android SDK (API 21+)
- A Google AdMob account (for ads) — [get your App ID](https://admob.google.com)
- A RevenueCat account (for billing) — [get your API key](https://app.revenuecat.com)
- Java 17 (required by Gradle)

### 1. Clone the repository

```bash
git clone https://github.com/narisystems/tracksub.git
cd tracksub
```

### 2. Install dependencies

```bash
flutter pub get
```

### 3. Configure environment variables

TrackSub uses `--dart-define` flags to inject secrets at build time so no API keys are committed to source control.

Create a local file called `.env.sh` (already in `.gitignore`) with your keys:

```bash
# .env.sh — never commit this file
export ADMOB_APP_ID="ca-app-pub-XXXXXXXXXXXXXXXX~XXXXXXXXXX"
export ADMOB_BANNER_ID="ca-app-pub-XXXXXXXXXXXXXXXX/XXXXXXXXXX"
export REVENUECAT_API_KEY="appl_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
export GOOGLE_DRIVE_CLIENT_ID="XXXXXXXX-XXXXXXXX.apps.googleusercontent.com"
```

Source the file before building:

```bash
source .env.sh
```

### 4. Run in debug mode

```bash
flutter run \
  --dart-define=ADMOB_APP_ID=$ADMOB_APP_ID \
  --dart-define=ADMOB_BANNER_ID=$ADMOB_BANNER_ID \
  --dart-define=REVENUECAT_API_KEY=$REVENUECAT_API_KEY \
  --dart-define=GOOGLE_DRIVE_CLIENT_ID=$GOOGLE_DRIVE_CLIENT_ID
```

### 5. Build a release APK / App Bundle

**APK (for sideloading / testing):**

```bash
flutter build apk --release \
  --dart-define=ADMOB_APP_ID=$ADMOB_APP_ID \
  --dart-define=ADMOB_BANNER_ID=$ADMOB_BANNER_ID \
  --dart-define=REVENUECAT_API_KEY=$REVENUECAT_API_KEY \
  --dart-define=GOOGLE_DRIVE_CLIENT_ID=$GOOGLE_DRIVE_CLIENT_ID
```

**App Bundle (for Play Store submission):**

```bash
flutter build appbundle --release \
  --dart-define=ADMOB_APP_ID=$ADMOB_APP_ID \
  --dart-define=ADMOB_BANNER_ID=$ADMOB_BANNER_ID \
  --dart-define=REVENUECAT_API_KEY=$REVENUECAT_API_KEY \
  --dart-define=GOOGLE_DRIVE_CLIENT_ID=$GOOGLE_DRIVE_CLIENT_ID
```

The signed `.aab` will be at `build/app/outputs/bundle/release/app-release.aab`.

> **Signing:** Configure your keystore in `android/key.properties`. See the [Flutter deployment guide](https://docs.flutter.dev/deployment/android) for full signing setup instructions.

### 6. Generate Drift database code

If you modify any Drift table definitions, regenerate the type-safe code:

```bash
dart run build_runner build --delete-conflicting-outputs
```

---

## 📁 Project Structure

```
tracksub/
├── android/                    # Android host project & Gradle config
├── lib/
│   ├── main.dart               # App entry point, env injection
│   ├── app/
│   │   ├── app.dart            # MaterialApp, theme, routing
│   │   └── constants.dart      # dart-define accessors, app-wide constants
│   ├── core/
│   │   ├── database/           # Drift tables, DAOs, and database class
│   │   ├── models/             # Domain models (Subscription, PriceEntry, etc.)
│   │   └── utils/              # Date helpers, CSV parser, Mint importer
│   ├── features/
│   │   ├── dashboard/          # Spending overview screen + providers
│   │   ├── subscriptions/      # List, detail, add/edit screens + providers
│   │   ├── reminders/          # WorkManager task scheduling
│   │   ├── backup/             # Google Drive backup/restore logic
│   │   ├── billing/            # RevenueCat integration, premium gate widget
│   │   └── settings/           # Settings screen, export/import, theme toggle
│   └── shared/
│       ├── widgets/            # Reusable UI components
│       └── theme/              # Color scheme, text styles, app theme
├── test/
│   ├── unit/                   # Model and utility tests
│   ├── widget/                 # Widget tests
│   └── integration/            # Full flow integration tests
├── docs/
│   └── screenshots/            # Play Store and README screenshots
├── pubspec.yaml
└── README.md
```

---

## 🤝 Contributing

Contributions are welcome! TrackSub is a small indie project, so even small improvements — bug fixes, typo corrections, translations — make a real difference.

### How to contribute

1. **Fork** the repository and create a feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes.** Please follow the existing code style and keep PRs focused on a single concern.

3. **Write or update tests** where applicable. Run the test suite before opening a PR:
   ```bash
   flutter test
   ```

4. **Open a Pull Request** against the `main` branch. Fill in the PR template and describe what your change does and why.

### Reporting bugs

Please use [GitHub Issues](https://github.com/narisystems/tracksub/issues) and include:
- Device model and Android version
- TrackSub version (visible in Settings → About)
- Steps to reproduce
- What you expected vs. what happened

### Feature requests

Open a [GitHub Discussion](https://github.com/narisystems/tracksub/discussions) before opening an issue for feature requests. This lets the community weigh in before implementation work begins.

### Code of Conduct

Be respectful. This project follows the [Contributor Covenant](https://www.contributor-covenant.org/version/2/1/code_of_conduct/) Code of Conduct.

---

## 📄 License

```
MIT License

Copyright (c) 2026 NaRiSystems

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

<div align="center">

Built with ❤️ by [NaRiSystems](https://narisystems.github.io/tracksub) · [Play Store](https://play.google.com/store/apps/details?id=com.tracksub.app) · [Privacy Policy](https://narisystems.github.io/tracksub/privacy)

</div>