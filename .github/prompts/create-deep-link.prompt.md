---
agent: agent
description: Generate deep-linking logic, string parsing, and route constants to securely link external URLs natively into the application's BLoC architecture.
argument-hint: "Deep link path structure (e.g., /document/:id), target route, expected parameters, and trigger actions"
---

# Create Deep Link & Route Engine

Deep Link Path: `${input:deepLinkPath}` | Target Feature: `${input:targetFeature}`
Parameters Expected: `${input:parameters}`

## File Locations

Update the following files to register the deep link:
```
lib/core/routes/app_router.dart
```

---

## Step 1 – Strict Route Constants

Update the routing constants string to explicitly define the parameter structure. Never hardcode routing strings across the app.

```dart
// Inside lib/core/routes/app_router.dart (or equivalent constants file)
static const String ${input:targetFeature}Route = '/${input:targetFeature}';
static const String ${input:targetFeature}DeepLink = '/${input:targetFeature}/:id';
```

---

## Step 2 – Strong-Typed Arguments

When passing parameters from a Deep Link to a screen, **never** pass raw `Map<String, dynamic>`. Always scaffold a strongly-typed Arguments class to prevent runtime crashes.

```dart
@immutable
final class ${input:targetFeature}Args extends Equatable {
  const ${input:targetFeature}Args({required this.id});

  final String id;

  // Defensive parsing from deep-link intent strings
  factory ${input:targetFeature}Args.fromUri(Uri uri) {
    return ${input:targetFeature}Args(
      id: uri.pathSegments.last, // Example extraction
    );
  }

  @override
  List<Object?> get props => [id];
}
```

---

## Step 3 – The Routing Interceptor

Inside the `onGenerateRoute` or `CustomRouter.generateRoute` method, add the matching case. If the link is malformed, securely redirect the user to an Error or Home page.

```dart
case RouteConstants.${input:targetFeature}Route:
  final args = settings.arguments;
  
  // 1. Validate the arguments strictly
  if (args is! ${input:targetFeature}Args) {
    DebugUtils.showPrint('DeepLink Error: Invalid or missing arguments for ${input:targetFeature}');
    return MaterialPageRoute(builder: (_) => const GenericErrorView());
  }
  
  // 2. Wrap the destination view in the Feature Bloc and fire the Load event immediately
  return MaterialPageRoute(
    builder: (_) => BlocProvider(
      create: (context) => ${input:targetFeature}Bloc(
         repo: ${input:targetFeature}Repo(),
      )..add(Load${input:targetFeature}Event(id: args.id)), // Automatically triggers data fetch
      child: const ${input:targetFeature}View(),
    ),
  );
```

---

## Rules

- **Strong Typing**: URL parameters are notoriously unreliable. Treat every deep-link parameter as a potentially malicious/malformed string. Always parse securely using `.tryParse` if looking for integers.
- **Fail Gracefully**: If a user clicks a broken or expired deep link, it must **never crash**. Use a fallback route (e.g. `GenericErrorView` or Home) and log the failure using `DebugUtils`.
- **Pre-fire BLoC Events**: When deep-linking directly into a specific feature (like a Document View), standard practice is to inject the ID into the `BlocProvider` and immediately `.add(LoadEvent(id))` so the screen loads seamlessly.
- **`dynamic` Ban**: Never extract routing settings using `as dynamic`. Use strict `if (args is MyArgs)` type-checking.
- **Observability**: Any failed deep link attempts must be logged to Crashlytics via `DebugUtils.showPrint()`.
- **Testing**: Instruct the developer to test this link using `adb shell am start -W -a android.intent.action.VIEW -d "url"` or `xcrun simctl openurl booted "url"`.
