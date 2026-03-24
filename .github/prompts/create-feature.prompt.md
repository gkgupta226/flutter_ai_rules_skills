---
agent: agent
description: Scaffold a complete feature folder (bloc/model/repo/view/widget) from acceptance criteria following project architecture standards.
argument-hint: "Feature name in snake_case, PascalCase variant, parent flow folder, and acceptance criteria"
---

# Create Feature

Feature: `${input:featureName}` | PascalCase: `${input:featurePascalName}` | Flow: `${input:parentFolder}`

## Acceptance Criteria

```
${input:acceptanceCriteria}
```

> **Before writing any code**, parse the acceptance criteria and derive:
>
> | Concern                   | How to derive from AC                                                                      |
> | ------------------------- | ------------------------------------------------------------------------------------------ |
> | **Events**                | One event per user action or system trigger (`SubmitBookingEvent`, `LoadProfileDataEvent`) |
> | **State status variants** | One enum value per observable UI state beyond `initial/loading/success/error`              |
> | **API calls**             | One repo method per data exchange described                                                |
> | **Edge cases**            | Every "when X fails / empty / unauthorised" clause → error state + snackbar                |
> | **Navigation**            | Each "navigates to …" clause → `NavigationService` call in `BlocListener`                  |
> | **Validation**            | Each "must / cannot / required" clause → guard before API call                             |

**Read before writing any string/colour/icon/style/route:**
[app_strings.dart](../../lib/core/res/strings/app_strings.dart) ·
[colors.dart](../../lib/core/res/colors/colors.dart) ·
[icons.dart](../../lib/core/res/drawables/icons.dart) ·
[font_style.dart](../../lib/core/res/fonts/font_style.dart) ·
[size_config.dart](../../lib/core/res/size/size_config.dart) ·
[app_router.dart](../../lib/core/routes/app_router.dart) ·
[api_config.dart](../../lib/core/data/network/api_config.dart)

## Folder structure

```
lib/features/${input:parentFolder}/${input:featureName}/
├── bloc/  ${input:featureName}_{bloc,event,state}.dart
├── model/ ${input:featureName}_{request,response}_model.dart  (if API needed)
├── repo/  ${input:featureName}_repo.dart
├── view/  ${input:featureName}_view.dart
└── widget/
```

## Per-file rules

**Bloc** — barrel import only; `part` both sibling files; constructor-inject repo
(`?? ${input:featurePascalName}Repo()`); `on<E>(_handler)` per event; emit `loading` (with `clearMessage: true`) →
`try`/`catch (e, stackTrace)` → emit `success`/`error`; log with `DebugUtils.showPrint` passing `stackTrace`; no navigation or UI.

**Event** — `part of` bloc; `@immutable sealed class ${input:featurePascalName}Event extends Equatable`;
each event `@immutable final class <Verb><Noun>Event`; `props` for all non-sensitive fields.

**State** — `part of` bloc; `enum ${input:featurePascalName}Status { initial, loading, success, error }`;
`@immutable final class` state extends `Equatable`; all `final` fields; full `copyWith` with `clearMessage` flag; `props`.

**Repo** — barrel import; constructor-inject `DioUtil ?? DioUtil()`; ALL operations (reads & writes) → `Future<Result<T>>`
(strictly decoupled from UI — no snackbars); all paths from `ApiConfig`;
`try`/`catch (e, stackTrace)` + inner JSON parsing `try`/`catch` + `DebugUtils.showPrint`.

**Models** — `@immutable`, all `final`; `fromJson` factory + `toJson`; null-safe defaults;
never declare a field as `dynamic`.

**View** — wrap scaffold body in `SafeArea` + `SingleChildScrollView`; `BlocProvider` at root fires initial event if needed; `BlocListener` for
side-effects (navigation, snackbars); `BlocBuilder` for UI; every string → `AppStrings`,
colour → `AppColors`, text → `TextWidget`, button → `CustomButton` (wrap custom tap targets in `Semantics(button: true)`), dims → `AppDimensions`.

- ScreenUtil; extract sub-trees to `widget/`.

**Route** — add `static const` to `RouteConstants`; add `case` in `CustomRouter.generateRoute`.

**Exports** — add cross-feature exports to `export.dart`.

**AC coverage check** — after generating all files, verify every AC clause is handled.

**Git Hooks Compliance** — unconditionally pass `flutter analyze`; NEVER output deprecated widgets (e.g. `WillPopScope`).

No `TODO` comments — implement minimally but completely.
