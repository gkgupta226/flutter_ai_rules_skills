---
agent: agent
description: End-to-end feature scaffold driven by acceptance criteria, a Figma design, and API request/response models. Orchestrates all sub-prompts in sequence to produce the complete bloc/model/repo/view/widget/test suite.
argument-hint: "Feature name, PascalCase variant, parent flow folder, Figma file key, Figma node ID, acceptance criteria, API request JSON, API response JSON"
---

# End-to-End Feature

Feature: `${input:featureName}` | PascalCase: `${input:featurePascalName}` | Flow: `${input:parentFolder}`
Figma file key: `${input:figmaFileKey}`
Figma node ID: `${input:figmaNodeId}`

## Acceptance Criteria

```
${input:acceptanceCriteria}
```

## API Contract

**Request:**

```json
${input:requestJson}
```

**Response:**

```json
${input:responseJson}
```

---

## How this prompt works

This is an **orchestrator**. Each phase below delegates to a dedicated sub-prompt that owns
the detailed rules for that concern. Read each linked sub-prompt **in full** before executing
its phase — the rules there are the authoritative source of truth. Do not re-interpret or
relax them here.

Work through phases **strictly in order**. Do not start Phase N+1 until Phase N is complete.

**CRITICAL:** To prevent token truncation and maintain code quality, you **MUST** naturally pause at the end of every Phase. Output your progress and explicitly ask the user to type "continue" before proceeding to the next Phase.

---

## Phase 1 – Parse & Plan (no code)

Before touching any file:

1. **Derive the Feature Design Table** from the acceptance criteria using the derivation rules
   in [create-feature.prompt.md](./create-feature.prompt.md) (the _Acceptance Criteria_ section).

   | #   | AC Clause | Event name | State status variant | Repo method | Edge case / guard |
   | --- | --------- | ---------- | -------------------- | ----------- | ----------------- |

2. **Fetch the Figma design** — call Figma MCP `get_figma_data`:
   - `fileKey`: `${input:figmaFileKey}`
   - `nodeId`: `${input:figmaNodeId}`

3. **Build the Design Token Map** following Steps 1–2 in
   [create-screen-from-figma.prompt.md](./create-screen-from-figma.prompt.md).

   | Token type | Figma value | Project constant | Action (use / add) |
   | ---------- | ----------- | ---------------- | ------------------ |

4. **Download assets** via Figma MCP `download_figma_images` for any icon/image nodes
   identified in the token map.

5. **Derive the model fields** from `${input:requestJson}` and `${input:responseJson}`.

Present the Feature Design Table, Design Token Map, and model field list. Do not proceed
until all three are complete.

---

## Phase 2 – Constants & Assets

Apply the token-mapping rules from
[create-screen-from-figma.prompt.md](./create-screen-from-figma.prompt.md) (Step 3).

Update only what is new — never modify existing constants:

- `lib/core/constants/colors/colors.dart`
- `lib/core/constants/strings/strings.dart`
- `lib/core/constants/drawable/icons.dart`
- `lib/core/constants/fonts/font_styles.dart`
- `lib/core/constants/app_dimensions.dart`
- `pubspec.yaml` (new asset paths only)

---

## Phase 3 – Models

Follow every rule in [create-model.prompt.md](./create-model.prompt.md).

Context:

- `modelName` → `${input:featurePascalName}`
- `modelType` → derive from API contract (request / response / both)
- `jsonOrFields` → use `${input:requestJson}` and `${input:responseJson}`
- Output path → `lib/features/${input:parentFolder}/${input:featureName}/model/`

**Code Generation Rule:** If using `json_serializable` or `freezed`, ensure the model declares the appropriate `part` file. After providing the code, remind the user to run `dart run build_runner build --delete-conflicting-outputs`.

---

## Phase 4 – Repository

Follow every rule in [create-repo.prompt.md](./create-repo.prompt.md).

Context:

- `featureName` → `${input:featureName}`
- `featurePascalName` → `${input:featurePascalName}`
- `apiMethods` → the _Repo method_ + _Edge case_ columns from the Feature Design Table
- API endpoint constant name → add to `lib/core/data/network/api_config.dart` if not present

---

## Phase 5 – Bloc

Follow every rule in [create-bloc.prompt.md](./create-bloc.prompt.md).

Context:

- `featureName` → `${input:featureName}`
- `featurePascalName` → `${input:featurePascalName}`
- `description` → the full _Event name_ + _State status variant_ + _Edge case_ columns
  of the Feature Design Table

The `${input:featurePascalName}Status` enum must include every variant listed in the Feature
Design Table in addition to the baseline `initial, loading, success, error`.

---

## Phase 6 – View & Widgets

Follow the view-generation rules in
[create-screen-from-figma.prompt.md](./create-screen-from-figma.prompt.md) (Steps 4–5)
**combined with** the _View_ bullet from
[create-feature.prompt.md](./create-feature.prompt.md) (the _Per-file rules_ → View section).

Additional wiring from AC:

- Every _"navigates to …"_ clause → `BlocListener` branch calling `NavigationService` with `RouteConstants.*`
- Every UI state clause → `BlocBuilder` / `BlocSelector` branch
- Bloc events dispatched from the UI must match the Feature Design Table exactly

Extract every sub-tree of ≥ 3 widgets or reused in siblings into a `widget/` file per
the rules in [create-screen-from-figma.prompt.md](./create-screen-from-figma.prompt.md) (Step 5).

---

## Phase 7 – Route & Exports

Follow the _Route_ and _Exports_ bullets from
[create-feature.prompt.md](./create-feature.prompt.md) (the _Per-file rules_ section):

- Add `static const String ${input:featureName} = '/${input:featureName}';` to `RouteConstants`
- Add a `case RouteConstants.${input:featureName}:` to `CustomRouter.generateRoute`
- Add cross-feature exports to `lib/core/router/export.dart`

---

## Phase 8 – Tests

Follow every rule in [write-tests.prompt.md](./write-tests.prompt.md).

Context:

- Bloc file → `lib/features/${input:parentFolder}/${input:featureName}/bloc/${input:featureName}_bloc.dart`
- Test file → `test/features/${input:featureName}/bloc/${input:featureName}_bloc_test.dart`
- Every row of the Feature Design Table must map to at least one `blocTest` case
- Minimum 3 tests per handler: happy path, failure/null response, exception thrown

---

## Phase 9 – Security, Quality & Compliance Gate

Run [security-review.prompt.md](./security-review.prompt.md) **AND** [app-store-compliance.prompt.md](./app-store-compliance.prompt.md) scoped to all files generated
or modified in Phases 2–8.

Additionally verify:

- [ ] Every AC clause is covered by at least one test from Phase 8
- [ ] Every Figma token from Phase 1 is accessed through a project constant — no inlined hex/rgba/dp values in widget code
- [ ] Every row of the Feature Design Table is implemented across Bloc, Repo, and View
- [ ] `props` in all events/states with passwords or tokens → those fields are excluded
- [ ] No `http` package usage — only `DioUtil`
- [ ] No `print()` — only `DebugUtils.showPrint()`
- [ ] All sensitive values stored in `SecureStorageHelper`, not `SharedPreferenceHelper`
- [ ] All routes use `RouteConstants.*` — no inline string literals
- [ ] All colours use `AppColors.*` — no hardcoded hex
- [ ] All strings use `AppStrings.*` — no hardcoded user-visible text
- [ ] All dimensions use `AppDimensions.*` + ScreenUtil — no raw doubles
- [ ] File sizes: views < 300 lines, blocs < 200 lines, repos < 200 lines
- [ ] **Git Hooks Check**: Ensure generated code strictly follows `dart format .` and passes `flutter analyze` unconditionally. Never output deprecated widgets.

Do not end until all nine phases are complete and every checklist item passes.
