---
agent: agent
description: Write bloc_test unit tests for a Bloc class covering happy path, null/empty response, and exception thrown for each handler.
argument-hint: "Feature name, Bloc class path, and list of event handlers to cover"
---

# Write Bloc Tests

Feature: `${input:featureName}` | Bloc: `${input:featurePascalName}Bloc`

Handlers to cover:

```
${input:handlers}
```

## File location

```
test/features/${input:featureName}/bloc/${input:featureName}_bloc_test.dart
```

## Test template

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

import 'package:flutter_base_project/features/${input:featureName}/bloc/${input:featureName}_bloc.dart';
import 'package:flutter_base_project/features/${input:featureName}/repo/${input:featureName}_repo.dart';
import 'package:flutter_base_project/core/types/result.dart';

// ── Mocks & Fakes ──────────────────────────────────────────────────────────

class Mock${input:featurePascalName}Repo extends Mock implements ${input:featurePascalName}Repo {}
class FakeRequestModel extends Fake implements RequestModel {} // Replace with actual request model type

// ── Helpers ────────────────────────────────────────────────────────────────

${input:featurePascalName}Bloc buildBloc(${input:featurePascalName}Repo repo) =>
    ${input:featurePascalName}Bloc(repo: repo);

// ── Tests ──────────────────────────────────────────────────────────────────

void main() {
  late Mock${input:featurePascalName}Repo mockRepo;

  setUpAll(() {
    // CRITICAL: Prevent mocktail crash when passing custom models through any()
    registerFallbackValue(FakeRequestModel());
  });

  setUp(() {
    mockRepo = Mock${input:featurePascalName}Repo();
  });

  group('${input:featurePascalName}Bloc', () {
    group('SubmitXxxEvent', () {
      // ── arrange ──
      const inputEvent = SubmitXxxEvent(someField: 'value');

      blocTest<${input:featurePascalName}Bloc, ${input:featurePascalName}State>(
        'emits [loading, success] on happy path',
        setUp: () {
          when(() => mockRepo.someWriteMethod(
                requestModel: any(named: 'requestModel'),
              )).thenAnswer((_) async => const Success(data: null));
        },
        build: () => buildBloc(mockRepo),
        act: (bloc) => bloc.add(inputEvent),
        expect: () => [
          const ${input:featurePascalName}State(status: ${input:featurePascalName}Status.loading, message: ''),
          const ${input:featurePascalName}State(status: ${input:featurePascalName}Status.success, message: ''),
        ],
      );

      blocTest<${input:featurePascalName}Bloc, ${input:featurePascalName}State>(
        'emits [loading, error] when repo returns Failure',
        setUp: () {
          when(() => mockRepo.someWriteMethod(
                requestModel: any(named: 'requestModel'),
              )).thenAnswer(
            (_) async => const Failure(message: 'Something went wrong.'),
          );
        },
        build: () => buildBloc(mockRepo),
        act: (bloc) => bloc.add(inputEvent),
        expect: () => [
          const ${input:featurePascalName}State(status: ${input:featurePascalName}Status.loading, message: ''),
          const ${input:featurePascalName}State(
            status: ${input:featurePascalName}Status.error,
            message: 'Something went wrong.',
          ),
        ],
      );

      blocTest<${input:featurePascalName}Bloc, ${input:featurePascalName}State>(
        'emits [loading, error] when repo throws exception',
        setUp: () {
          when(() => mockRepo.someWriteMethod(
                requestModel: any(named: 'requestModel'),
              )).thenThrow(Exception('network error'));
        },
        build: () => buildBloc(mockRepo),
        act: (bloc) => bloc.add(inputEvent),
        expect: () => [
          const ${input:featurePascalName}State(status: ${input:featurePascalName}Status.loading, message: ''),
          isA<${input:featurePascalName}State>()
              .having((s) => s.status, 'status', ${input:featurePascalName}Status.error),
        ],
      );
    });

    // Repeat group per handler listed in ${input:handlers}
  });
}
```

## Rules

- **Minimum 3 tests per handler:** happy path, failure response, exception thrown.
- Use `bloc_test` `blocTest<B, S>` — never test `.state` after `add` manually.
- Mock repos with `mocktail` via constructor injection: `${input:featurePascalName}Bloc(repo: mockRepo)`.
- **Never** use real network calls in tests.
- Follow **Arrange–Act–Assert**; prefix variables: `inputEvent`, `mockRepo`, `actualState`, `expectedState`.
- Variable names: `input*` for inputs, `mock*` for mocks, `actual*` for captured values, `expected*` for expected values.
- Each `blocTest` description must start with: `'emits [<states>] when <condition>'`.
- Test file: `test/features/${input:featureName}/bloc/${input:featureName}_bloc_test.dart`.
- Register fallback values with `registerFallbackValue` in `setUpAll` if needed by mocktail.
- **Git Hooks Compliance**: The generated test file MUST strictly pass `dart format .` and `flutter analyze` without any custom deprecation warnings.
