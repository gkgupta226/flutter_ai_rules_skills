---
agent: agent
description: Create a Repository class for a feature following the project's DioUtil + Result<T> conventions.
argument-hint: "Feature name in snake_case, PascalCase variant, and list of API methods needed with HTTP verb and endpoint"
---

# Create Repository

Feature: `${input:featureName}` | PascalCase: `${input:featurePascalName}`

API methods needed:

```
${input:apiMethods}
```

## File location

```
lib/features/<parentFolder>/${input:featureName}/repo/${input:featureName}_repo.dart
```

## Template

```dart
import '../../../../core/router/export.dart';

class ${input:featurePascalName}Repo {
  ${input:featurePascalName}Repo({DioUtil? dioUtil}) : _dioUtil = dioUtil ?? DioUtil();

  final DioUtil _dioUtil;

  // WRITE operation (POST/PUT/DELETE) — returns Result<T>
  Future<Result<ResponseType>> someWriteMethod({
    required RequestModel requestModel,
  }) async {
    try {
      final Map<String, dynamic>? response = await _dioUtil.postApi(
        url: ApiConfig.someEndpoint,
        body: requestModel.toJson(),
        showMessage: false,
      );
      if (response == null) {
        return const Failure(message: ApiMsgStrings.somethingWentWrong);
      }
      
      // Wrap parsing in try/catch to gracefully handle JSON mapping crashes
      try {
        return Success(data: ResponseType.fromJson(response));
      } catch (e, stackTrace) {
        DebugUtils.showPrint('${input:featurePascalName}Repo.someWriteMethod JSON Parsing error: $e\n$stackTrace');
        return const Failure(message: ApiMsgStrings.somethingWentWrong);
      }
    } catch (e, stackTrace) {
      DebugUtils.showPrint('${input:featurePascalName}Repo.someWriteMethod network error: $e\n$stackTrace');
      return const Failure(message: ApiMsgStrings.somethingWentWrong);
    }
  }

  // READ operation (GET) — returns Result<T> (DECOUPLED FROM UI)
  Future<Result<ResponseType>> someReadMethod() async {
    try {
      final Map<String, dynamic>? response = await _dioUtil.getApi(
        url: ApiConfig.someEndpoint,
      );
      if (response == null) {
        return const Failure(message: ApiMsgStrings.somethingWentWrong);
      }
      
      // Wrap parsing in try/catch to gracefully handle JSON mapping crashes
      try {
        return Success(data: ResponseType.fromJson(response));
      } catch (e, stackTrace) {
        DebugUtils.showPrint('${input:featurePascalName}Repo.someReadMethod JSON Parsing error: $e\n$stackTrace');
        return const Failure(message: ApiMsgStrings.somethingWentWrong);
      }
    } catch (e, stackTrace) {
      DebugUtils.showPrint('${input:featurePascalName}Repo.someReadMethod network error: $e\n$stackTrace');
      return const Failure(message: ApiMsgStrings.somethingWentWrong);
    }
  }
}
```

## Rules

- **Constructor-inject** `DioUtil` with `?? DioUtil()` fallback for testability.
- **Unified Return Type**: ALL operations (READs & WRITEs) must exclusively return `Future<Result<T>>`.
- **API Response Parsing Safety**: Always wrap `ResponseType.fromJson(response)` in an inner `try/catch` to gracefully catch and report `TypeError` or `FormatException`.
- All endpoint paths come from `ApiConfig` constants — never inline URL strings.
- **Catch Blocks & Observability**: Always catch `(e, stackTrace)` so that the full context is logged and can be handled by a crash reporter. Use `DebugUtils.showPrint('ClassName.methodName error: $e\n$stackTrace')`.
- **Never** `rethrow` exceptions into the void. Always convert exceptions to `Failure(message)`.
- **Clean Architecture Ban**: You must **never** import `Bloc`, `event`, `state`, `snackbars`, `dialogs`, or `view` files into the repository. The data layer must remain 100% headless and completely decoupled from UI execution.
- Use barrel import `import '../../../../core/router/export.dart'`.
- Every repo method must terminate at a `DioUtil` call — no self-recursion or retry loops.
