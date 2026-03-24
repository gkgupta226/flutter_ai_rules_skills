---
agent: agent
description: Generate a Flutter screen and widgets from a Figma node, mapping Figma design tokens to project constants (AppColors, FontStyles, AppDimensions, AppIcons, AppStrings).
argument-hint: "Figma file key, Figma node ID, feature name, output path"
---

# Create Screen from Figma

Feature: `${input:featureName}` | PascalCase: `${input:featurePascalName}`
Figma file key: `${input:figmaFileKey}`
Figma node ID: `${input:figmaNodeId}`

---

## Step 1 – Fetch Figma data

Call Figma MCP `get_figma_data`:

- `fileKey`: `${input:figmaFileKey}`
- `nodeId`: `${input:figmaNodeId}`
- Retrieve the full node tree including children, styles, and component variants.

Parse the response and extract every design token present:

| Token type               | Figma value      | Notes                                |
| ------------------------ | ---------------- | ------------------------------------ |
| fill colour              | `#RRGGBB` / rgba | map to `AppColors.*`                 |
| stroke colour            | `#RRGGBB` / rgba | map to `AppColors.*`                 |
| font family              | string           | must match `FontFamily.*`            |
| font size                | number           | map to nearest `FontStyles.*` getter |
| font weight              | number           | map to nearest `FontStyles.*` getter |
| line height              | number/%         | map to nearest `FontStyles.*` getter |
| corner radius            | number           | map to `AppDimensions.radius*`       |
| horizontal padding       | number           | map to `AppDimensions.*`             |
| vertical padding/spacing | number           | map to `AppDimensions.*`             |
| icon/image node          | nodeId           | schedule for download                |
| visible text             | string           | map to `AppStrings.*`                |

---

## Step 2 – Build Design Token Map & download assets

For each token extracted in Step 1, produce:

| Token type | Figma value    | Project constant         | Action                 |
| ---------- | -------------- | ------------------------ | ---------------------- |
| colour     | `#2196F3`      | `AppColors.primary`      | use                    |
| colour     | `#FF0000`      | —                        | **add** to `AppColors` |
| text       | "Welcome back" | `AppStrings.welcomeBack` | use / **add**          |
| icon       | node `1234:56` | `AppIcons.someIcon`      | **download** + add     |

**Rules:**

- A token already present in a project constant → **use** it.
- A token not yet in project constants → **add** it to the relevant constants file, following naming conventions (see project `copilot-instructions.md`). Never hardcode values in widget files.
- Colours: never use `.withOpacity()` — encode alpha into the constant hex value.
- For icon/image nodes: call Figma MCP `download_figma_images` with those node IDs and save to `assets/icons/` or `assets/images/`. Register new asset paths under `flutter: assets:` in `pubspec.yaml` and add a constant to `AppIcons`.

---

## Step 3 – Apply constants

Update the following files as needed (only add what is new — never modify existing constants):

- `lib/core/res/colors/colors.dart` — new `AppColors.*` entries
- `lib/core/res/strings/app_strings.dart` — new `AppStrings.*` entries
- `lib/core/res/drawables/icons.dart` — new `AppIcons.*` entries
- `lib/core/res/fonts/font_style.dart` — new `FontStyles.*` getter (only if a genuinely new style is needed; prefer composing `.copyWith()` on existing getters in widget code)
- `lib/core/res/size/size_config.dart` — new `AppDimensions.*` getter (only if spacing value has no existing equivalent)
- `pubspec.yaml` — new asset entries

---

## Step 4 – Generate the view

Create `lib/features/${input:parentFolder}/${input:featureName}/view/${input:featureName}_view.dart`.

**Layout rules (derived from Figma node tree):**

- **SafeArea & Scrolling**: Wrap the outermost screen scaffold body in a `SafeArea` to respect iPhone Notches and Android system bars. Assume ALL screens need to scroll on smaller devices; wrap the primary `Column` in a `SingleChildScrollView` to prevent pixel overflows.
- **Input Intelligence**: If a Figma component visually represents an input area (a border box with placeholder text), intelligently replace it with a native `TextFormField` rather than generating a static painted `Container`.
- Replicate the Figma layout hierarchy: frames → `Column`/`Row`/`Stack`; auto-layout direction → `Column`/`Row`; auto-layout gap → `SizedBox(height: ...)` / `SizedBox(width: ...)`.
- **Text Overflow**: Wrap text-heavy `Row` children in `Expanded` or use `TextOverflow.ellipsis` to gracefully handle large system font scalings.
- All sizes use ScreenUtil: widths → `.w`, heights → `.h`, font sizes → `.sp`, radii → direct `AppDimensions.*` getter value.
- All colours → `AppColors.*`. All text → `TextWidget(text: ..., style: FontStyles.*)`. All buttons → `CustomButton(...)`.
- Complex or repeated sub-trees (≥ 3 widgets) → extract to `widget/` (Step 5).
- Strictly follow all _View_ rules from `create-feature.prompt.md`.

**BLoC wiring:**

- Wrap screen root in `BlocProvider(create: (_) => FeatureBloc())`.
- State-driven UI → `BlocBuilder<FeatureBloc, FeatureState>`.
- Side effects (navigation, snackbars) → `BlocListener<FeatureBloc, FeatureState>`.
- Both → `BlocConsumer`.
- Dispatch events matching the Feature Design Table from `create-feature.prompt.md`.

---

## Step 5 – Extract widgets

For each identified sub-tree:

1. Create `lib/features/${input:parentFolder}/${input:featureName}/widget/<widget_name>_widget.dart`.
2. Widget is `StatelessWidget` wherever possible; `StatefulWidget` only for locally animated or form-field sub-trees.
3. **Accessibility & Maestro**: Every extracted button, custom interactive container, or touch target MUST be wrapped in a `Semantics(identifier: 'specific_btn_name', button: true)` widget to comply with screen readers AND allow Maestro E2E tests to locate them. It must also guarantee a tap target of at least 48x48.
4. Accept only the minimal typed parameters needed — no passing of entire state objects.
5. **Performance Purity**: Aggressively apply `const` keywords to every generic widget to ensure 60fps scrolling.
6. Pass a `key` parameter.

---

## Final UI Check

- **Git Hooks Compliance**: The final generated code must unconditionally pass `flutter analyze` and `dart format`.
- **No Deprecations**: NEVER output deprecated layout widgets (e.g., use `PopScope` over `WillPopScope`).
- No `TODO` comments in generated files — implement completely or leave the slot empty with a `// placeholder` comment explaining what goes there.
