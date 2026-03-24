# Flutter Base Project – GitHub Copilot Instructions

You are a senior Dart/Flutter engineer working on this project.
Adhere to all conventions established in the codebase when generating, correcting, or refactoring code.

---

## Project at a Glance

| Concern              | Solution                                                             |
| -------------------- | -------------------------------------------------------------------- |
| State management     | `flutter_bloc` (Bloc + Cubit)                                        |
| State immutability   | `Equatable` — all states and events extend it                        |
| Dependency injection | Constructor injection in Blocs; repos use `DioUtil()` singleton      |
| Navigation           | `NavigationService` singleton + `CustomRouter.generateRoute`         |
| Networking           | `DioUtil` (Dio wrapper) only — never use the `http` package          |
| Secure storage       | `SecureStorageHelper` for tokens; `SharedPreferenceHelper` for flags |
| Routing constants    | `RouteConstants` (not inline strings or `static const id`)           |
| String constants     | `AppStrings`                                                         |
| Icon/asset constants | `AppIcons`                                                           |
| Colour constants     | `AppColors`                                                          |
| Dimension constants  | `AppDimensions`                                                      |
| Result type          | `Result<T>` / `Success<T>` / `Failure` from `core/types/`            |
| Barrel import        | `import '../../../../core/router/export.dart';`                      |

---

## Dart / Flutter General Rules

### Language

- Write all code and documentation in **English**.
- Always declare explicit types for variables, parameters, and return values.  
  Avoid `dynamic` or `var` unless unavoidable.
- Use `final` for everything that does not reassign.
- Prefer `const` constructors and widget instantiations wherever possible.

### Naming

| Entity                          | Convention                             | Example                 |
| ------------------------------- | -------------------------------------- | ----------------------- |
| Classes                         | PascalCase                             | `HomeBloc`              |
| Variables / functions / methods | camelCase                              | `fetchUserProfile()`    |
| Files and directories           | snake_case                             | `home_bloc.dart`        |
| Constants (compile-time)        | SCREAMING_SNAKE_CASE in constant class | `AppStrings.APP_NAME`   |
| Boolean variables               | Verb prefix                            | `isLoading`, `hasError` |

- Start every method/function name with a descriptive **verb**.
- Use complete words; abbreviate only established terms (`API`, `URL`, `OTP`, `FCM`).

### Functions & Methods

- Keep functions **< 20 statements**; extract helpers when a function grows beyond that.
- Use **early returns** instead of deeply nested `if/else` blocks.
- Prefer **named parameters** with `required` for functions that accept multiple arguments.

### Classes

- Follow SOLID principles; one responsibility per class.
- Keep classes **< 200 lines** with **< 10 public methods**.
- Prefer **composition** over inheritance.

---

## Feature-First Architecture

Every feature lives under `lib/features/<feature_name>/` and follows this structure:

```
features/
└── feature_name/
    ├── bloc/          # Bloc + Event + State (split into separate part files)
    ├── model/         # Request/response DTOs  (fromJson / toJson)
    ├── repo/          # Repository — wraps DioUtil calls
    ├── view/          # Page widgets (one per route)
    └── widget/        # Small, reusable widgets scoped to this feature
```

Shared code lives in `lib/core/`:

```
core/
├── components/        # Reusable UI widgets (buttons, dialogs…)
├── constants/         # AppStrings, AppColors, AppIcons, AppDimensions, enums
├── data/
│   ├── network/       # DioUtil, ApiConfig, ApiResponse, SecureStorageHelper
│   └── shared_preferences/
├── models/            # Cross-feature domain models (e.g. UserModel)
├── router/            # CustomRouter, RouteConstants, export.dart (barrel)
├── services/          # NavigationService
├── theme/             # AppTheme
├── types/             # Result<T>, Success<T>, Failure<T>
└── utils/             # DebugUtils, ApplicationUtils, AppBlocObserver, mixins
```

> **Important:** always import via the barrel file:
>
> ```dart
> import '../../../../core/router/export.dart';
> ```

---

## Bloc Pattern

### File Layout

Each Bloc is split into three `part` files:

```
feature/bloc/
├── feature_bloc.dart   ← Bloc class
├── feature_event.dart  ← Events  (part of 'feature_bloc.dart')
└── feature_state.dart  ← State   (part of 'feature_bloc.dart')
```

### State

- Base state extends `Equatable`.
- Use an **enum** for status: at minimum `initial`, `loading`, `success`, `error`.
- Provide a full **`copyWith`** method.
- Override `List<Object?> get props` — include every field.
- `@immutable final class` for the state.
- Do **not** store passwords, tokens, or credentials in state.

### Events

- `@immutable final class` for every event.
- Events extend the sealed base `extends <Feature>Event`.
- Name events `<Verb><Noun>Event`: `SubmitLoginEvent`, `LoadHomeDataEvent`.
- `props` must **never** include passwords or credentials.

### Bloc Class

- Constructor-inject repo with a nullable fallback:
  `FeatureBloc({FeatureRepo? repo}) : _repo = repo ?? FeatureRepo()`.
- Register handlers: `on<EventType>(_onEventName)`.
- Handler signature: `Future<void> _onEventName(Event event, Emitter<State> emit) async`.
- Always emit `loading` **first** inside an async handler.
- Always emit a terminal state (`success` or `error`) in **every** code path including `catch`.
- Log errors: `DebugUtils.showPrint('FeatureBloc._onX error: $e')`.
- **Never** perform navigation or show dialogs inside a Bloc.

---

## Repository Pattern

- Constructor-inject `DioUtil` with a nullable fallback:
  `FeatureRepo({DioUtil? dioUtil}) : _dioUtil = dioUtil ?? DioUtil()`.
- **Reads** → return `Future<T?>` (call `showSnackBar` on failure, return `null`).
- **Writes** → return `Future<Result<T>>` (`Success<T>` / `Failure`).
- All endpoint paths must come from `ApiConfig` constants.
- Wrap every network call in `try/catch`; log with `DebugUtils.showPrint`.
- Never `rethrow` unless the caller is designed to handle it.

---

## Models (DTOs)

- `@immutable`, all fields `final`.
- `factory fromJson(Map<String, dynamic>)` + `toJson()`.
- Null-safe defaults: `?? ''`, `?? 0`, `?? false`, `?? []`.
- Never declare a field as `dynamic` — cast JSON values explicitly.
- Request models in `feature/model/` named `<Noun>RequestModel`.
- Response models in `feature/model/` named `<Noun>ResponseModel`.

---

## Navigation

- Use `NavigationService` for all programmatic navigation.
- Use `RouteConstants` for all route name strings — never inline strings.
- Pass typed argument objects, not raw `Map`s.

---

## UI / Widgets

- All widgets must use `const` constructors where possible.
- Use `flutter_screenutil` (`16.w`, `20.h`, `14.sp`) for all dimensions.
  **Never** use `MediaQuery` for sizing; **never** use raw doubles.
- Use `AppColors` for every colour — never hardcode hex/RGB.
  Never use `.withOpacity()` — encode alpha into the constant.
- Use `AppStrings` for every user-facing string.
- Use `AppIcons` for every asset path.
- Use `AppDimensions` for all spacing/size constants.
- All `TextStyle`s must use named `FontStyles` getters from
  `lib/core/constants/fonts/font_styles.dart` — no inline `TextStyle(...)`.
- Use `TextWidget` for all text display — no raw `Text(...)` in screen widgets.
- Use `CustomButton` for primary actions.
- Use `BlocBuilder` for state-driven UI; `BlocListener` for side-effects.
- Use `BlocConsumer` only when both building and listening are needed.
- Never use `print()` — use `DebugUtils.showPrint()`.
- Feedback to users: `showSnackBar(message, SnackType.success/failed)`.
- Extract logical sub-trees into `StatelessWidget` classes in `widget/`.
- Always pass a `key` to list items.

---

## Error Handling & Result Type

Use `Result<T>` / `Success<T>` / `Failure` from `core/types/result.dart` for any write
operation. Switch on the result in the Bloc handler:

```dart
switch (result) {
  case Success(:final data):
    emit(state.copyWith(status: FeatureStatus.success, data: data));
  case Failure(:final message):
    emit(state.copyWith(status: FeatureStatus.error, message: message));
}
```

---

## Security

### Secrets & Keys

- **Never** hardcode API keys, tokens, or secrets in Dart source.
- Inject all secrets at build time via `--dart-define=KEY=value`; read with
  `String.fromEnvironment('KEY')`.
- **`AppConfig.environment`** is set at build time — never change it at runtime.

### Local Storage

- Use `SecureStorageHelper` for **all** security-sensitive values: auth token,
  refresh token, FCM token.
- Use `SharedPreferenceHelper` only for non-sensitive app state (theme, onboarding).
- **Never** store a password or credential in `SharedPreferences` or Bloc state.

### Bloc State

- `props` must never include passwords, tokens, or payment secrets.
- Clear any credential field immediately after the handler that consumed it:
  `emit(state.copyWith(password: ''))`.

### WebView

- Validate every URL loaded against `ApiConfig` domain constants before loading.
- Reject URLs whose host is not in the allowlist.

### Network

- All API calls must be HTTPS. Never disable certificate validation.
- URL selection is done via `AppConfig.environment` + `ApiConfig.baseUrl` only.

---

## Reliability

- Every repository method must terminate at a `DioUtil` call — no self-recursion.
- Every `async` function calling the network must have a `try/catch`.
- Emit `ApiMsgStrings.somethingWentWrong` to the UI — never expose raw exception messages.
- Always emit `loading` first; always emit a terminal state in every code path.

### Linting & Formatting
- All code must conform exactly to the project's `analysis_options.yaml` (e.g., `flutter_lints` or `very_good_analysis`).
- Never generate code that requires `// ignore:` comments.
- Always append trailing commas `,` to Flutter widget properties to ensure proper `dart format` structure.

### Observability
- `DebugUtils.showPrint` is for debug strings only. For `catch(e, stackTrace)` blocks, pass the exception and stack trace to your designated crash reporter service facade.

### Cubit vs Bloc
- Default to `Cubit` for straightforward state changes. Only use `Bloc` when you specifically need `Event` transformers (e.g., debounce, throttle, droppable).

### UI & E2E Testing
- **Maestro (E2E Flows)**: Use Maestro instead of Flutter integration tests for testing full user journeys, especially those involving native OS permission dialogs (Camera, Biometrics, Vault).
- **Semantics**: Every intractable UI component must be wrapped in `Semantics(identifier: 'specific_name')` to ensure Maestro can reliably tap it during E2E flows. Do not rely on fixed X/Y coordinates.
- **Widget Tests**: Create isolated Widget Tests strictly for custom UI components mapping to Mock Blocs. Mock network images or animations during tests to prevent flaky failures.

---


---

## App Store & Play Store Compliance
To ensure the app passes Apple App Store and Google Play Store reviews, strictly adhere to the following when generating code:

### Permissions & Privacy
- **Just In Time Requesting**: Never request permissions (Camera, Location, Contacts, etc.) at app launch. Request them only when the user initiates a feature that requires them.
- **Rationale Strings**: Ensure that `Info.plist` and `AndroidManifest.xml` have clear, user-friendly rationale strings explaining *why* the permission is needed.
- **Account Deletion**: If generating user profile settings, always include an option to **Delete Account** natively within the app.

### UI / UX & Platform Behaviors
- **Safe Areas**: Always wrap top-level scaffold contents or floating elements in a `SafeArea` to avoid overlaps with the iOS notch, dynamic island, or Android system bars.
- **Back Navigation**: Ensure proper `WillPopScope` or `PopScope` implementations for Android hardware back buttons so users are not trapped on a screen.
- **Accessibility**: Use `Semantics` widgets for custom controls. Ensure text scales correctly with the system text size without overflowing (use `TextOverflow.ellipsis` or scalable flexible layouts).
- **Dark Mode**: Never hardcode colors that would break in Dark Mode. Always use `Theme.of(context).colorScheme` or the predefined `AppColors`.

### Authentication & Payments
- **Sign in with Apple**: If adding any third-party SSO (Google, Facebook), you *must* concurrently implement Sign in with Apple for iOS.
- **Digital Goods**: If generating features that unlock digital content or subscriptions, do not use external payment gateways (like Stripe); use native In-App Purchases (IAP).

## CI/CD & Git Hooks (Pre-Commit)
To enforce code formatting, linting, and reliability before code reaches the repository, a strict pre-commit hook is active.
- **Formatting**: Generated code and tests must output cleanly via `dart format . --set-exit-if-changed`.
- **Linting**: Code must pass `flutter analyze` without any warnings.
- **Tests**: Code must not break existing test cases and must pass `flutter test`.
Never generate code that attempts to circumvent or ignore these rules, as the native git hooks will block all non-compliant commits.

---

## Maintainability

### Single Source of Truth

| Class            | File                                     | Purpose                     |
| ---------------- | ---------------------------------------- | --------------------------- |
| `AppStrings`     | `core/res/strings/app_strings.dart`      | All user-facing strings     |
| `AppColors`      | `core/res/colors/colors.dart`            | All colour values           |
| `AppIcons`       | `core/res/drawables/icons.dart`          | SVG/PNG asset paths         |
| `AppDimensions`  | `core/res/size/size_config.dart`         | Spacing/size constants      |
| `RouteConstants` | `core/routes/app_router.dart`            | Navigation route strings    |
| `ApiConfig`      | `core/data/network/api_config.dart`      | API base URLs and endpoints |
| `ApiMsgStrings`  | `core/res/strings/api_msg_strings.dart`  | API error/success messages  |

### Dependency Direction

- Features must not import each other. Shared models and services live in `core/`.
- Blocs must not import views or widgets.
- Repos must not import Blocs or views.
- `core/` must not import from `features/`.

### Code Size Limits

- Files: **< 300 lines**.
- Functions / methods: **< 20 statements**.
- Classes: **< 200 lines**, **< 10 public methods**.

### Environment Switching

Control via `--dart-define=ENV=dev|uat|prod` at build time.
Never merge code where `baseUrl` is hardcoded to a dev or UAT value.

---

## Coverage

- Every public Bloc event handler must have a corresponding `blocTest`.
- Minimum per handler: (1) happy path, (2) null/empty response, (3) exception thrown.
- Use `flutter_test` + `bloc_test` + `mocktail`. Mock repos via constructor injection.
- Never use real network calls in unit tests.
- Follow **Arrange–Act–Assert**; prefix test variables `input`, `mock`, `actual`, `expected`.

---

## Automation – Prompt Files

Use the prompt files in `.github/prompts/` as the entry point for structured work:

| Task                                         | Prompt file                          |
| -------------------------------------------- | ------------------------------------ |
| **Full e2e (Figma + AC + API → prod-ready)** | `create-feature-e2e.prompt.md`       |
| Maestro E2E Test Flows                       | `create-maestro-flow.prompt.md`      |
| Deep Link / Routing Architecture             | `create-deep-link.prompt.md`         |
| New feature (full stack, no Figma)           | `create-feature.prompt.md`           |
| Screen from Figma only                       | `create-screen-from-figma.prompt.md` |
| Bloc only                                    | `create-bloc.prompt.md`              |
| Repository only                              | `create-repo.prompt.md`              |
| Model DTOs from JSON                         | `create-model.prompt.md`             |
| Tests for a Bloc                             | `write-tests.prompt.md`              |
| Security / quality audit                     | `security-review.prompt.md`          |
| App Store / Play Store compliance check      | `app-store-compliance.prompt.md`     |
