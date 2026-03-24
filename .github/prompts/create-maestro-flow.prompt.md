---
agent: agent
description: Generate comprehensive, modular Maestro E2E test flows (YAML) for testing critical user journeys, strictly adhering to Flutter Semantics and native OS interaction rules.
argument-hint: "Flow name, target feature, and step-by-step user journey description"
---

# Create Maestro E2E Flow

Flow Name: `${input:flowName}` | Feature: `${input:targetFeature}`
Journey:
```
${input:journeyDescription}
```

## File Locations

All Maestro E2E flows belong in the root-level `.maestro/` directory.

```
.maestro/
├── common/             (For reusable sub-flows like login.yaml)
└── flows/
    └── ${input:flowName}.yaml
```

---

## Step 1 – The Boilerplate

Every Maestro YAML file must start with the `appId` declaration and basic setup, heavily relying on environment variables so tests can run on both Android and iOS dynamically.

```yaml
appId: ${APP_ID}
env:
  # Define default variables if they aren't passed by the CLI
  EMAIL: "test@example.com"
---
# ── Setup / Initialization ──
- clearState: true
- launchApp:
    clearState: true
```

---

## Step 2 – Reusable Flow Stitching (`runFlow`)

If the journey requires the user to be logged in, **never** manually rewrite the login steps. Always use `- runFlow` to stitch common modules together.

```yaml
# ── Pre-requisites ──
- runFlow:
    file: ../common/login_flow.yaml
    env:
      USER: ${EMAIL}
```

---

## Step 3 – The Journey Steps (The "Flutter" Rules)

Translate the `${input:journeyDescription}` into Maestro commands. 

### CRITICAL Flutter Rules:
1. **Never use brittle X/Y coordinates**: Do not use `- tapOn: point: 100,200`.
2. **Prefer Semantics over raw text**: Flutter paints text onto a canvas. Maestro often struggles to read custom fonts. ONLY use `- tapOn: "Text"` if it is standard. Otherwise, rely on Flutter `Semantics` labels: `- tapOn: id: "btn_submit_document"`.
3. **If targeting an ID**, you **MUST** ensure the corresponding Flutter widget actually has a semantic identifier. (e.g. `Semantics(identifier: 'btn_submit_document', child: ...)`).
4. **Native OS Dialogs**: FormFill Vault extensively uses OS permissions (Camera, Storage, Biometrics). If the journey triggers a permission, blindly assume the native OS dialog appears and accept it.

```yaml
# ── The Action ──
- tapOn: id: "btn_add_document"

# ── Handle OS Permissions (Android/iOS independent) ──
- tapOn: "Allow.*"   # Regex matches "Allow", "Allow Access", "While using the app", etc.

# ── Dynamic Input ──
- tapOn: id: "input_document_title"
- inputText: "Aadhaar Card"
- hideKeyboard

# ── Scrolling & Asserting ──
- scrollUntilVisible:
    element: id: "btn_save_vault"
    direction: DOWN
- tapOn: id: "btn_save_vault"

# ── Validate Success ──
- assertVisible: "Document saved successfully"
```

---

## Step 4 – Assertions and Teardown

Every E2E Flow must end with an incontrovertible visual assertion that the goal was achieved, followed by an optional teardown.

```yaml
# ── Conclusion ──
- extendedWaitUntil:
    visible: id: "vault_list_item_aadhaar"
    timeout: 5000 # Wait up to 5 seconds for the DB to save and UI to refresh
- assertVisible: "Aadhaar Card"
```

---

## Output Validation & Rules

Before generating the final YAML, ensure:
1. All `id:` targets use snake_case strings.
2. You explicitly output a reminder to the developer: *"Please ensure `Semantics(identifier: '...')` was added to your Flutter Widgets, otherwise Maestro cannot tap these IDs."*
3. You include the exact terminal command to run the flow: e.g., `maestro test .maestro/flows/${input:flowName}.yaml -e APP_ID=com.example.formfill`
