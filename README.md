# VitalySync

**A mobile wellness companion for recognizing burnout-related patterns and supporting small, sustainable changes.**

VitalySync combines a Flutter app with a Node.js/Express API and PostgreSQL. People can record daily wellbeing signals, complete periodic reflections, see trends and burnout-oriented scores, and receive practical suggestions tied to their recent data. The project is an active academic and portfolio application. It is a wellness aid, not a diagnostic or clinical tool.

## Case study

### The problem

Sleep, workload, mood, movement, food, and recovery affect one another, but they are usually tracked in separate places. A single difficult day also says little about a longer pattern. VitalySync was built to make repeated check-ins useful: collect a manageable set of signals, show how they change over time, and turn them into concrete next steps without presenting a score as a medical diagnosis.

### The approach

1. **Collect context.** Onboarding establishes a wellness profile and an initial burnout reflection. Daily logs cover sleep, mood, energy, hydration, workload, symptoms, activity, and recovery habits. Weekly check-ins add deeper reflection items.
2. **Make patterns visible.** The API stores timestamped logs and calculates scores and summaries. The app displays histories, trends, and dashboards so users can inspect the evidence behind a difficult week.
3. **Suggest a small action.** Adaptive nudges use recent context, preferences, and feedback. Users can mark an idea done, ask for another, or postpone it. Nutrition and exercise flows offer more focused guidance.
4. **Keep the experience usable.** Authored guidance remains available when AI wording cannot be generated. The mobile app caches selected content and coordinates reminder delivery with the API.

### Key product flows

| Flow | What it does |
| --- | --- |
| Onboarding and account | Captures a baseline and preferences; supports authentication and account management. |
| Daily and weekly check-ins | Records lifestyle signals and periodic reflection without treating missing entries as zeroes. |
| Dashboard and reports | Shows burnout-oriented history, wellness trends, and exportable personal reports. |
| Nutrition and activity | Records meals and movement, with image-assisted meal analysis and exercise goals where configured. |
| Smart nudges | Offers context-aware actions with completion, skip, snooze, and feedback controls. |
| Reminders | Uses device notifications and saved nudge content so an alert opens the same suggestion the user received. |

### Engineering decisions

- **Scores are computed in application code.** The backend scoring engine combines daily signals with reflection dimensions and keeps questionnaire/scoring versions explicit. AI is used for wording and analysis features, not as the source of the numerical burnout score.
- **Missing data stays missing.** The scoring and insights paths account for incomplete logs rather than assuming an unrecorded habit was poor or good.
- **Suggestions have a stable identity.** The backend persists selected nudge content and delivery state. The app and Android overlay use that saved content to avoid changing the text between card, notification, and history.
- **External services are optional at the core.** The backend can use OpenAI for generated wording and meal analysis, USDA FoodData Central for nutrition data, OpenWeather for environmental context, and Brevo/SMTP for account email. Credentials stay on the server. Authored copy covers the main suggestion flow when generation is unavailable.
- **Security is enforced at the API boundary.** Protected routes use authentication and account matching; middleware also applies CORS, rate limits, request size limits, and HTTP security headers. These controls still require production configuration and testing.

### Current scope and limitations

VitalySync is under active development. Burnout-oriented scores and reflection prompts are for self-awareness; they have not been presented here as a validated clinical assessment. Notification timing depends on device permissions and operating-system behavior. AI and external data features need their respective credentials and network access. This case study describes implemented flows and design choices, not measured user outcomes.

## Architecture

```text
Flutter app (mobile and supported Flutter targets)
  |  REST/JSON, account token, locale
  v
Express API
  |-- authentication, check-ins, scoring, reports, nudges
  |-- optional AI, nutrition, weather, and email integrations
  v
PostgreSQL (versioned SQL migrations)
```

The Flutter client keeps screens in `lib/features`, shared services and widgets in `lib/shared`, and app setup in `lib/app`. The API separates routes, controllers, services, middleware, and database migrations. The API base URL is set with Flutter's `API_BASE_URL` Dart define; API credentials are read from backend environment variables.

## Repository map

```text
VitalySync/
├── vitalysync-frontend/   Flutter client, assets, and widget tests
├── vitalysync-backend/    Express API, SQL migrations, and Node tests
├── .gitignore             Workspace, secret, dependency, and build exclusions
├── LICENSE                MIT license
└── README.md              Project overview and setup
```

See the [frontend README](vitalysync-frontend/README.md) for app-specific structure and UI conventions. Local research documents, work outputs, Android build packages, editor settings, and agent instructions are excluded from this repository.

## Run locally

### Prerequisites

- Flutter SDK compatible with the Dart constraint in `vitalysync-frontend/pubspec.yaml` (currently Dart `^3.9.2`)
- Node.js and npm compatible with the backend dependencies
- PostgreSQL and a local database
- An Android/iOS device or supported Flutter desktop/web target for the client

### 1. Configure and start the API

From `vitalysync-backend`, copy `.env.example` to `.env` and set a local PostgreSQL connection and a long, random `AUTH_TOKEN_SECRET`. Set optional service keys only for the integrations you intend to use. The example file contains placeholders; keep the real `.env` local.

```powershell
cd vitalysync-backend
Copy-Item .env.example .env
npm ci
npm run migrate
npm run dev
```

The server defaults to port `3000`. Check `http://localhost:3000/api/health` for a running API and `/api/ready` for database readiness. `npm start` runs migrations before starting the server; `npm run dev` does not, so run migrations first in development.

### 2. Run the Flutter app

In a second terminal:

```powershell
cd vitalysync-frontend
flutter pub get
flutter run --dart-define=API_BASE_URL=http://localhost:3000
```

Use a host address reachable from the device when `localhost` refers to the emulator or phone itself. For Flutter web, the backend's `CORS_ALLOWED_ORIGINS` must include the local web origin. If no Dart define is supplied, the app uses the deployed backend URL in `lib/shared/config/api_config.dart`.

### Configuration notes

- The backend accepts `DATABASE_URL` or the split `DB_*` settings shown in `.env.example`.
- `AUTH_TOKEN_SECRET` is required for production. Keep it out of client builds and source control.
- `OPENAI_API_KEY`, `USDA_API_KEY`, `OPENWEATHER_API_KEY`, and email provider settings enable their respective integrations. Do not put these values in Flutter code or `--dart-define` flags.
- Set production CORS origins, proxy hops, database TLS, and rate limits for the actual deployment environment.

## Checks

```powershell
cd vitalysync-backend
npm test
```

```powershell
cd vitalysync-frontend
flutter analyze
flutter test
```

Some integration paths also need a running database, configured providers, or a real device to verify end-to-end behavior. Review SQL migrations before applying them to a shared database.

## Privacy and repository hygiene

The root `.gitignore` excludes local `.env` files, credentials and signing keys, dependency folders, generated builds, reports, and the workspace-only folders named above. The backend `.env.example` documents variable names with non-secret placeholders. Before sharing a deployment, review its provider keys, database URL, and signing configuration in the hosting platform; only server-side secrets belong there.

## License

The project source is released under the [MIT License](LICENSE). Third-party packages and assets retain their respective licenses.
