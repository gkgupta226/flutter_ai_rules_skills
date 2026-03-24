---
agent: agent
description: Create a Bloc class, Event file, and State file for an existing or new feature following project BLoC conventions.
argument-hint: "Feature name in snake_case, PascalCase variant, and description of events/state needed"
---

# Create Bloc

Feature: `${input:featureName}` | PascalCase: `${input:featurePascalName}`

Events / state described:

```
${input:description}
```

## File layout

```
lib/features/<parentFolder>/${input:featureName}/bloc/
├── ${input:featureName}_bloc.dart   ← Bloc class (main file)
├── ${input:featureName}_event.dart  ← part of bloc
└── ${input:featureName}_state.dart  ← part of bloc
```

## Bloc class — `${input:featureName}_bloc.dart`

```dart
import '../../../../core/router/export.dart';

part '${input:featureName}_event.dart';
part '${input:featureName}_state.dart';

class ${input:featurePascalName}Bloc extends Bloc<${input:featurePascalName}Event, ${input:featurePascalName}State> {
  ${input:featurePascalName}Bloc({${input:featurePascalName}Repo? repo})
      : _repo = repo ?? ${input:featurePascalName}Repo(),
        super(const ${input:featurePascalName}State()) {
    on<SubmitXxxEvent>(_onSubmitXxx);
    // add one on<> per event
  }

  final ${input:featurePascalName}Repo _repo;

  Future<void> _onSubmitXxx(
    SubmitXxxEvent event,
    Emitter<${input:featurePascalName}State> emit,
  ) async {
    emit(state.copyWith(status: ${input:featurePascalName}Status.loading));
    try {
      final Result<void> result = await _repo.someWriteMethod();
      switch (result) {
        case Success():
          emit(state.copyWith(status: ${input:featurePascalName}Status.success));
        case Failure(:final message):
          emit(state.copyWith(status: ${input:featurePascalName}Status.error, message: message));
      }
    } catch (e, stackTrace) {
      DebugUtils.showPrint('${input:featurePascalName}Bloc._onSubmitXxx error: $e\n$stackTrace');
      // Forward the exception and stackTrace to your crash reporter here
      emit(state.copyWith(
        status: ${input:featurePascalName}Status.error,
        message: ApiMsgStrings.somethingWentWrong,
      ));
    }
  }
}
```

## Event file — `${input:featureName}_event.dart`

```dart
part of '${input:featureName}_bloc.dart';

@immutable
sealed class ${input:featurePascalName}Event extends Equatable {
  const ${input:featurePascalName}Event();
}

@immutable
final class SubmitXxxEvent extends ${input:featurePascalName}Event {
  const SubmitXxxEvent({required this.someField});

  final String someField;

  @override
  List<Object?> get props => [someField]; // never include passwords/tokens
}
```

## State file — `${input:featureName}_state.dart`

```dart
part of '${input:featureName}_bloc.dart';

enum ${input:featurePascalName}Status { initial, loading, success, error }

@immutable
final class ${input:featurePascalName}State extends Equatable {
  const ${input:featurePascalName}State({
    this.status = ${input:featurePascalName}Status.initial,
    this.message = '',
    // add domain fields here
  });

  final ${input:featurePascalName}Status status;
  final String message;

  ${input:featurePascalName}State copyWith({
    ${input:featurePascalName}Status? status,
    String? message,
    bool clearMessage = false,
  }) =>
      ${input:featurePascalName}State(
        status: status ?? this.status,
        message: clearMessage ? '' : (message ?? this.message),
      );

  @override
  List<Object?> get props => [status, message];
}
```

## Rules

- Emit `loading` first in every async handler. Ensure you reset/clear previous error messages when doing so using the `clearMessage` flag.
- Always emit a terminal state (`success` or `error`) in **every** code path including `catch`.
- **Catch Blocks & Observability**: Always catch `(e, stackTrace)` so that the full context is logged and can be caught by a crash reporter.
- **Event Transformers**: If the event handles rapid user input (like searching or submitting a form button), explicitly apply `droppable()`, `restartable()`, or `debounceTime()` transformers from `bloc_concurrency`.
- **Form Validation**: If the feature handles many form fields, validate inputs inside the state before the API call to avoid unnecessary networking.
- Log errors: `DebugUtils.showPrint('${input:featurePascalName}Bloc.<handler> error: $e')`.
- **Never** perform navigation or show dialogs inside the Bloc.
- `props` must **never** include passwords, tokens, or credentials.
- Use barrel import `import '../../../../core/router/export.dart'` — not individual imports.
