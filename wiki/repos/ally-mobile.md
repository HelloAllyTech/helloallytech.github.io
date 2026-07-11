---
title: ally-mobile — Mobile App
tags: [repo, mobile, react-native, ios, android]
summary: React Native (0.79.1) iOS/Android app for mental-health counselors, providing live counseling sessions with audio streaming and transcription, peer reviews, and AI voice simulations against the Ally backend.
---

# ally-mobile — Mobile App

## Purpose

`ally-mobile` is the React Native application for counselors on the Ally platform. It supports secure, privacy-first counselor–client sessions with live audio streaming and transcription, detailed call summaries, peer review workflows, AI-driven voice simulations for training, and analytics/gamification (achievements, leaderboards, badges). The repository is open source under the MIT License. The app package version is `1.5.0`; iOS/Android application IDs are `com.helloally.app` (prod) and `com.helloally.app.dev` (dev).

## Tech Stack

Sourced from `package.json`:

- **Framework:** React Native `0.79.1`, React `19.0.0`
- **Language:** TypeScript `^5.0.4`
- **State Management:** Redux Toolkit `@reduxjs/toolkit ^2.7.0` with `react-redux ^9.2.0`
- **Navigation:** React Navigation v7 — `@react-navigation/native`, `native-stack`, `stack`, `bottom-tabs`, `drawer`
- **Real-time audio (simulations):** `@livekit/react-native ^2.9.1`, `@livekit/react-native-webrtc ^137.0.1`, `livekit-client ^2.15.2`
- **WebSocket signaling/transcription (live sessions):** `socket.io-client ^4.8.1`
- **Audio capture/playback:** `react-native-live-audio-stream`, `react-native-audio-api`, `react-native-sound-player`, `react-native-video`
- **Firebase:** `@react-native-firebase/app ^22.4.0`, `@react-native-firebase/crashlytics` (crash reporting)
- **Auth SDKs:** `@react-native-google-signin/google-signin`, `@invertase/react-native-apple-authentication`
- **Notifications:** `@notifee/react-native`
- **Config:** `react-native-config` (env vars), consumed via `src/config/env.ts`
- **UI/charts/animation:** `@shopify/react-native-skia`, `react-native-reanimated`, `@gorhom/bottom-sheet`, `echarts` / `@wuba/react-native-echarts`, `react-native-chart-kit`, `lottie-react-native`, `react-native-svg`
- **i18n:** `i18next` / `react-i18next`
- **Storage:** `@react-native-async-storage/async-storage`
- **Node engine:** `>=18`

## Architecture & Feature Areas

The app follows a component-based architecture with Redux Toolkit for state, stack/tab/drawer navigation, a global theme provider (`src/theme/`), and custom hooks. Source is organized under `src/`: `assets`, `components`, `config`, `constants`, `contexts`, `hooks`, `localization`, `navigation`, `screens`, `services`, `store`, `theme`, `types`, `utils`.

Navigation is composed of `RootNavigator`, `TabNavigator`, and `DrawerNavigator` (`src/navigation/`). The Redux store (`src/store/`) currently holds `chatSlice`, `creditsSlice`, `summarySlice`, and `userSlice`.

Feature areas (from `src/screens/` and `src/services/`):

- **Live sessions & audio streaming / transcription** — `ListeningScreen` splits into `MicrophoneListeningScreen` (in-app microphone capture) and `ConferenceListeningScreen` (cloud-telephony/conference audio). These use Socket.IO for live audio streaming and transcription via hooks `useLiveAudioStream`, `useSocket`, and `useSocketConnection`. Related services: `callsApi`, `callTranscriptApi`, `callSummaryApi`, `chatApi`. Post-session review is handled by `CallLog`, `CallSummaryScreen`, and `CaseDetailsScreen`.
- **Reviews (peer review)** — `ReviewScreen`, `ReviewDetailsScreen`, and `ShareForReviewScreen`, with comment/thread/reaction flows. Services include `reviewsApi`, `reviewThreadsApi`, `reactionsApi`, `generalCommentsApi`, and a parallel "scribe" set (`scribeReviewsApi`, `scribeReviewThreadsApi`, `scribeReactionsApi`, `scribeGeneralCommentsApi`). A `ReviewApiContext` (`src/contexts/`) selects the active review API.
- **Simulations (AI voice training)** — `SimulationsScreen`, `SimulationDetailsScreen`, `SimulationPathDetailsScreen`, `SimulationAssistantScreen` (the live LiveKit voice session with participant tiles and audio-wave visualization), and `SimulationFeedbackScreen`. Services: `simulationsApi`, `simulationSummaryApi`, `simulationTranscriptApi`. This is the LiveKit-based integration (the training agent lives in sibling repo `ally-ai-learn`).
- **Auth** — Google Sign-In and Apple authentication SDKs; `authApi` and `userApi` services; a `SplashScreen` and `SuspendedAccountScreen`/`LogoutScreen` for account states.
- **Analytics & gamification** — `UserAnalyticsScreen`, `AchievementsScreen`, `LeaderboardScreen`, `ReactionsScreen`, `CommunityScreen`; services `analyticsApi`, `badgesApi`, `leaderboardApi`, `reflectionsApi`.
- **Wellbeing / misc** — `BoxBreathingScreen`, `ProfileSettingsScreen`, `SearchScreen` (`searchApi`, `placesApi`), plus `settingsApi`, `appConfigApi`, `customFieldsApi`, and `notificationService`.

## Integration Points

- **ally-be (backend REST):** All HTTP requests use `API_BASE_URL` + `API_VERSION` (e.g. `/v1/...`) as configured in `.env` and validated in `src/config/env.ts`. The `src/services/*Api.ts` modules wrap these endpoints atop a shared `api.ts` client.
- **Socket.IO namespaces (live sessions, via ally-be):** `MICROPHONE_SOCKET_PATH=/microphone-chat` for in-app microphone audio streaming, and `CLOUD_TELEPHONY_PATH=/cloud-telephony-chat` for conference/cloud-telephony audio. These paths are appended to `API_BASE_URL` to establish WebSocket connections (see `src/hooks/useSocket.ts`).
- **LiveKit (simulations):** `SimulationAssistantScreen` connects to LiveKit rooms via `@livekit/react-native` for real-time voice simulations (the agent is served by `ally-ai-learn`).
- **Deep linking to ally-web:** `ALLY_WEB_URL` is used for deep linking and sharing links to the Ally web app.
- **Firebase:** Crashlytics reporting and platform config; requires `GoogleService-Info.plist` (iOS) and `google-services.json` (Android).

Required environment variables (from `env/.env.example`; the app throws at startup if any required one is missing, per `src/config/env.ts`):

| Variable | Purpose |
|----------|---------|
| `ENVIRONMENT` | `development` \| `production` |
| `API_BASE_URL` | Base URL for ally-be REST API and WebSockets |
| `API_VERSION` | API version prefix (e.g. `v1`) |
| `ALLY_WEB_URL` | ally-web base URL for deep links/sharing |
| `MICROPHONE_SOCKET_PATH` | Socket.IO path `/microphone-chat` |
| `CLOUD_TELEPHONY_PATH` | Socket.IO path `/cloud-telephony-chat` |
| `GOOGLE_WEB_CLIENT_ID` | Google OAuth web/Android client ID |
| `GOOGLE_IOS_CLIENT_ID` | Google OAuth iOS client ID |
| `NOTIFICATION_SCHEDULE_DAY` / `_HOUR` / `_MINUTE` | Local notification schedule (defaults Mon 11:00) |

## Local Setup

Prerequisites: Node.js >= 18, Ruby (iOS), Xcode, Android Studio, CocoaPods.

```bash
# 1. Install JS dependencies (runs patch-package + husky install via lifecycle scripts)
npm install

# 2. iOS native dependencies
bundle install
bundle exec pod install

# 3. Environment config — create .env in repo root from the documented sample
cp env/.env.example .env   # then fill in values

# 4. Platform secrets (both git-excluded)
#   iOS:     ios/GoogleService-Info.plist
#   Android: android/app/google-services.json
#   Android release signing: provide your own keystore locally (never commit)

# 5. Start Metro bundler
npm start                  # npx react-native start --reset-cache

# 6. Run the app
npm run ios                # iOS simulator/device
npm run android:dev        # Android dev variant (appId com.helloally.app.dev, installDevDebug)
npm run android:prod       # Android prod variant (appId com.helloally.app, installProdDebug)
npm run android            # plain react-native run-android
```

Release/build helpers: `npm run build:dev` / `build:prod` (Gradle assemble Dev/Prod Release), `npm run android:clean`, and `npm run bundle:android` / `bundle:ios` to produce JS bundles.

## Testing & Code Quality

- **Tests:** Jest (`jest.config.js`, `jest.setup.ts`, `__mocks__/`). Run with `npm test` (`jest --passWithNoTests --forceExit`) or `npm run test:coverage`. Uses `@testing-library/react-native` and `@testing-library/jest-native`. Tests are colocated in `__tests__/` directories across `src/`.
- **Linting/formatting:** ESLint (`.eslintrc.js`, `npm run lint`), Prettier (`.prettierrc.js`, `npm run format`). Strict import grouping/ordering enforced via `@ianvs/prettier-plugin-sort-imports` and `eslint-plugin-unused-imports`.
- **Git hooks:** Husky (`.husky/`) + lint-staged (ESLint `--fix` and Prettier on staged `src/**/*.{js,jsx,ts,tsx}`); Commitlint (`commitlint.config.js`) enforces Conventional Commits.
- **Type checking:** TypeScript (`tsconfig.json`, `tsconfig.tests.json`).
- **Patches:** `patch-package` runs on `postinstall`; native/library patches live in `patches/`.

## Key Documentation

- `README.md` — Overview, features, tech stack, project structure, design system, full getting-started and scripts reference.
- `CONTRIBUTING.md` — Git branch-naming and Conventional Commit conventions, PR guidelines, code review process.
- `SECURITY.md` — Vulnerability reporting (admin@helloally.ai) and the list of secret files that must never be committed.
- `CODE_OF_CONDUCT.md` — Community code of conduct.
- `env/.env.example` — Fully commented reference for every required environment variable.
- `.github/ISSUE_TEMPLATE/` — `BUG_REPORT.md` and `FEATURE_REQUEST.md` issue templates.

> **Repo status note:** at migration time this checkout was on `master`, 26 commits behind `origin/master` (fast-forwardable), with uncommitted changes limited to `Gemfile.lock` and `package-lock.json` (no source files modified). Docs reflect the current working tree.

---

*Part of the [Ally Platform](../platform/overview.md). See also: [Architecture](../platform/architecture.md), [ally-be](ally-be.md), [ally-web](ally-web.md).*
