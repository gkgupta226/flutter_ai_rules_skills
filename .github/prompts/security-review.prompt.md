---
agent: agent
description: Perform a 6-pillar security and quality audit on a feature or file. Returns a finding table with severity ratings.
argument-hint: "Feature name or file path to audit"
---

# Security & Quality Review

Target: `${input:target}`

## Audit pillars

Evaluate every finding against all six pillars. Assign severity using the colour key below,
then emit a consolidated finding table followed by a prioritised fix list.

### 1 — Security

Check against OWASP Top-10 adapted for mobile/Flutter:

| Check                    | What to look for                                                                  |
| ------------------------ | --------------------------------------------------------------------------------- |
| Secrets in source        | Hardcoded API keys, tokens, passwords, base URLs in `.dart` files                 |
| Insecure storage         | Auth tokens / credentials in `SharedPreferences` instead of `SecureStorageHelper` |
| Credential in Bloc state | Password or token field in `props` or stored beyond the handler that consumed it  |
| Plaintext credentials    | Passwords or secrets logged with `print`, `debugPrint`, or `DebugUtils`           |
| WebView URL validation   | URL loaded without checking against `ApiConfig` domain allowlist                  |
| HTTP instead of HTTPS    | Any `http://` in endpoint constants                                               |
| Certificate validation   | `badCertificateCallback` set to `true` or similar bypass                          |
| Injection surface        | Dynamic SQL / shell strings built from user input                                 |
| `dart-define` bypass     | `AppConfig.environment` mutated at runtime                                        |

### 2 — Reliability

| Check                    | What to look for                                                 |
| ------------------------ | ---------------------------------------------------------------- |
| Missing `try/catch`      | `async` function that calls network without `catch`              |
| No terminal state        | Handler that can exit without emitting `success` or `error`      |
| `loading` not first      | Handler emits API call before emitting `loading`                 |
| Raw exception to UI      | `e.toString()` or exception message exposed in state / snackbar  |
| Self-recursion in repo   | Repo method that calls itself                                    |
| `rethrow` without reason | `rethrow` where caller is not designed to handle                 |
| Unawaited futures        | `async` call without `await` or explicit discard (`unawaited()`) |
| Missing stackTrace       | `catch (e)` block that fails to capture and log the `stackTrace` |
| `BuildContext` async gap | Using `context` after an `await` without `if (!context.mounted) return;` |
| Memory Leaks             | Missing `dispose()` method for `TextEditingController`, `ScrollController`, or Streams |

### 3 — Maintainability

| Check                      | What to look for                                                |
| -------------------------- | --------------------------------------------------------------- |
| Hardcoded strings          | User-visible strings not in `AppStrings`                        |
| Hardcoded colours / hex    | Colour literals not in `AppColors`                              |
| Raw `Text(...)` widget     | `Text(...)` used instead of `TextWidget`                        |
| Inline `TextStyle`         | `TextStyle(...)` not from `FontStyles` getters                  |
| `MediaQuery` for sizing    | `MediaQuery.of(context).size` used for layout dimensions        |
| Raw doubles for dimensions | `SizedBox(height: 16)` etc. not from `AppDimensions`/ScreenUtil |
| Inline route strings       | Route name strings not from `RouteConstants`                    |
| Cross-feature imports      | Feature A importing directly from Feature B (not via `core/`)   |
| Circular dependency        | `core/` importing from `features/`                              |
| File size                  | Any file exceeding 300 lines                                    |
| Function size              | Any function exceeding 20 statements                            |
| `print` / `debugPrint`     | Raw print calls not using `DebugUtils`                          |
| `dynamic` field            | Model or state fields typed as `dynamic`                        |
| UI in Data Layer           | Repositories returning `Future<T?>` instead of `Result<T>` or importing UI elements like Snackbars |
| Deprecated Widgets         | Usage of widgets triggering `flutter analyze` warnings (e.g. `WillPopScope` instead of `PopScope`) |
| Routing in BLoC            | `Navigator` or `NavigationService` calls happening inside a `_bloc.dart` instead of UI     |
| Missing Semantics          | Custom `GestureDetector` buttons lacking `Semantics` wrappers for screen readers           |
| Obscured / Tiny Targets    | Clickable areas that do not meet 48x48 dp minimum guidelines                               |

### 4 — Test coverage

| Check                 | What to look for                                                     |
| --------------------- | -------------------------------------------------------------------- |
| Missing bloc test     | Public event handler without corresponding `blocTest`                |
| Incomplete test cases | Handler tested with fewer than 3 cases (happy / failure / exception) |
| Real network in tests | Unit tests that instantiate real `DioUtil` or call actual endpoints  |
| `var` / missing types | Test variables not explicitly typed                                  |

### 5 — Duplication

| Check                  | What to look for                                                     |
| ---------------------- | -------------------------------------------------------------------- |
| Repeated UI snippets   | Identical widget sub-trees not extracted to `widget/`                |
| Duplicate API logic    | Same endpoint called from multiple repos                             |
| Copy-paste state/event | State or event boilerplate copy-pasted and not DRY'd via shared base |

### 6 — Performance & Architecture Purity

| Check                  | What to look for                                                     |
| ---------------------- | -------------------------------------------------------------------- |
| Missing `const`        | Neglecting `const` constructors for UI elements, causing useless widget rebuilds |
| `build()` pollution    | Expensive loops, synchronous JSON parsing, or heavy logic inside the `build()` method |
| Main Isolate blocking  | Parsing massive JSON strings natively instead of safely using `Isolate.run()` |
| Massive ListViews      | Using `Column` or `ListView` instead of `ListView.builder` for unbounded lists |

---

## Output format

### Finding table

| #   | File                | Line | Pillar   | Severity    | Finding           | Fix                     |
| --- | ------------------- | ---- | -------- | ----------- | ----------------- | ----------------------- |
| 1   | `path/to/file.dart` | 42   | Security | 🔴 Critical | Hardcoded API key | Move to `--dart-define` |

Severity key:

| Icon | Level    | Definition                                            |
| ---- | -------- | ----------------------------------------------------- |
| 🔴   | Critical | Immediate security breach or data loss risk           |
| 🟠   | High     | Likely failure in production or data exposure         |
| 🟡   | Medium   | Degrades reliability, maintainability, or testability |
| 🟢   | Low      | Style / convention divergence; low impact             |

### Fix list (prioritised)

List fixes in descending severity order. For each fix provide:

- **File path** (relative to `lib/`)
- **What to change** (concise instruction or code snippet)
- **Why** (one sentence referencing the relevant rule from the copilot instructions)
