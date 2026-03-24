---
agent: agent
description: Generate Dart model DTOs (request and/or response) from a JSON payload or field description, following project model conventions.
argument-hint: "Model name in PascalCase, request/response/both, raw JSON or field list"
---

# Create Model

Model: `${input:modelName}` | Type: `${input:modelType}` (request / response / both)

JSON or field list:

```json
${input:jsonOrFields}
```

## File location

```
lib/features/<parentFolder>/<featureName>/model/
├── ${input:modelFileName}_request_model.dart   (if request or both)
└── ${input:modelFileName}_response_model.dart  (if response or both)
```

## Request model template

```dart
import '../../../../core/router/export.dart';

@immutable
final class ${input:modelName}RequestModel extends Equatable {
  const ${input:modelName}RequestModel({
    required this.field1,
    // add all required fields
  });

  final String field1;

  Map<String, dynamic> toJson() => {
        'field1': field1,
      };

  @override
  List<Object?> get props => [field1];
}
```

## Response model template

```dart
import '../../../../core/router/export.dart';

@immutable
final class ${input:modelName}ResponseModel extends Equatable {
  const ${input:modelName}ResponseModel({
    required this.id,
    this.createdAt,
    // add all fields
  });

  factory ${input:modelName}ResponseModel.fromJson(Map<String, dynamic> json) =>
      ${input:modelName}ResponseModel(
        // parsed defensively:
        id: json['id']?.toString() ?? '',
        createdAt: DateTime.tryParse(json['created_at']?.toString() ?? ''),
        // map every JSON key
      );

  final String id;
  final DateTime? createdAt;

  Map<String, dynamic> toJson() => {
        'id': id,
        'created_at': createdAt?.toIso8601String(),
      };

  ${input:modelName}ResponseModel copyWith({
    String? id,
    DateTime? createdAt,
  }) =>
      ${input:modelName}ResponseModel(
        id: id ?? this.id,
        createdAt: createdAt ?? this.createdAt,
      );

  @override
  List<Object?> get props => [id, createdAt];
}
```

## Rules

- `@immutable` on every model class.
- All fields are `final`.
- Every response model has a `factory fromJson(Map<String, dynamic> json)`.
- Every model has a `toJson()` → `Map<String, dynamic>`.
- Null-safe defaults: `?? ''`, `?? 0`, `?? false`, `?? []`, `?? {}`.
- **Never** declare a field as `dynamic` — cast and parse JSON values defensively to prevent `TypeError` crashes:
  - String: `json['key']?.toString() ?? ''`
  - int: `int.tryParse(json['key']?.toString() ?? '') ?? 0`
  - double: `double.tryParse(json['key']?.toString() ?? '') ?? 0.0`
  - bool: `json['key'] is bool ? json['key'] as bool : (json['key']?.toString().toLowerCase() == 'true')`
  - DateTime: `DateTime.tryParse(json['key']?.toString() ?? '')`
  - List: `(json['key'] as List<dynamic>? ?? []).map((e) => ItemModel.fromJson(e as Map<String, dynamic>)).toList()`
- **Equatable**: All models must extend `Equatable` and implement `get props => [...]` so they trigger reliable UI rebuilds when embedded in a BLoC State.
- Request models live in `<feature>/model/` named `<Noun>RequestModel`.
- Response models live in `<feature>/model/` named `<Noun>ResponseModel`.
- Provide `copyWith` on response models that are stored in Bloc state.
- Use barrel import `import '../../../../core/router/export.dart'` only if needed for shared types.
