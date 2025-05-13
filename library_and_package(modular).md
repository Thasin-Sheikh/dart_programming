# Dart Libraries and Packages (Modular)

## What Are Libraries and Packages?

- **Library**: A collection of functions, classes, and variables grouped together in a file or set of files.
- **Package**: A bundle that can include multiple libraries along with metadata like version and dependencies.

### Why Use Libraries and Packages?
- **Code Organization**: Group related code together
- **Reusability**: Reuse code across multiple apps
- **Maintainability**: Easier to manage and debug
- **Modularity**: Enables separation of concerns
- **Team Collaboration**: Supports distributed development

---

## Creating and Using Libraries

### 1. Creating a Basic Library
Every Dart file is implicitly a library. Example:

`lib/temperature_converter.dart`
```dart
// temperature_converter.dart
// This file acts as a self-contained library

/// Converts Celsius to Fahrenheit
double celsiusToFahrenheit(double celsius) => (celsius * 9 / 5) + 32;

/// Converts Fahrenheit to Celsius
double fahrenheitToCelsius(double fahrenheit) => (fahrenheit - 32) * 5 / 9;

/// A class that provides detailed temperature conversions
class TemperatureConverter {
  double toFahrenheit(double celsius) => celsiusToFahrenheit(celsius);
  double toCelsius(double fahrenheit) => fahrenheitToCelsius(fahrenheit);

  String describeConversion(double value, String fromUnit) {
    if (fromUnit.toLowerCase() == 'c') {
      return '$value°C is ${toFahrenheit(value)}°F';
    } else if (fromUnit.toLowerCase() == 'f') {
      return '$value°F is ${toCelsius(value)}°C';
    } else {
      return 'Unknown unit';
    }
  }
}
```

### 2. Importing and Using a Library

`bin/main.dart`
```dart
import 'package:your_project/temperature_converter.dart';

void main() {
  print(celsiusToFahrenheit(0));       // Output: 32.0
  print(fahrenheitToCelsius(100));     // Output: 37.777...

  final converter = TemperatureConverter();
  print(converter.describeConversion(100, 'f'));
}
```

---

## Library Directives
Dart provides several directives to manage dependencies and visibility. These are powerful but should be used judiciously:

- **`part`/`part of`**:  
  Can make code harder to maintain if overused. Use sparingly.

- **`export`**:  
  Useful for creating facade libraries that simplify APIs.

- **Prefixes (`as`)**:  
  Essential when dealing with conflicting names.

### 1. `import`
Import libraries from:
```dart
import 'package:your_project/library.dart'; // Local
import 'dart:math';                         // Dart core
import 'package:http/http.dart';           // External
```

### 2. `part` and `part of`
Split libraries across multiple files:
```dart
// main_library.dart
library math_utils;
part 'geometry.dart';
part 'algebra.dart';

double square(double x) => x * x;
```
```dart
// geometry.dart
part of math_utils;
double calculateCircleArea(double radius) => 3.14 * square(radius);
```

### 3. `export`
Combine multiple files into a unified API:
```dart
// lib/math_library.dart
export 'src/geometry.dart';
export 'src/algebra.dart';
export 'src/statistics.dart';
```

---

## Library Prefixes
Avoid name clashes:
```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

void main() {
  Element element1 = Element();
  lib2.Element element2 = lib2.Element();
}
```

---

## Show and Hide
Control what to import:
```dart
import 'temperature_converter.dart' show celsiusToFahrenheit, fahrenheitToCelsius;
import 'temperature_converter.dart' hide TemperatureConverter;
```

---

## Creating a Reusable Dart Package (`dart_utils`)

### Directory Structure
```
dart_utils/
├── lib/
│   ├── src/
│   │   ├── string_utils.dart
│   │   └── date_utils.dart
│   └── dart_utils.dart
├── pubspec.yaml
└── README.md
```

### pubspec.yaml
```yaml
name: dart_utils
description: A collection of utility functions for Dart projects
version: 1.0.0
environment:
  sdk: '>=2.19.0 <3.0.0'
dependencies:
  intl: ^0.18.0
dev_dependencies:
  lints: ^2.0.0
  test: ^1.21.0
```

### string_utils.dart
```dart
class StringUtils {
  static String capitalize(String text) =>
    text.isEmpty ? text : text[0].toUpperCase() + text.substring(1);

  static String truncate(String text, int maxLength, {String ellipsis = '...'}) =>
    text.length <= maxLength ? text : text.substring(0, maxLength - ellipsis.length) + ellipsis;

  static String normalizeWhitespace(String text) =>
    text.trim().replaceAll(RegExp(r'\s+'), ' ');

  static bool isValidEmail(String email) {
    final regex = RegExp(r'^[\w\.-]+@([\w\-]+\.)+[\w\-]{2,4}\$');
    return regex.hasMatch(email);
  }
}
```

### date_utils.dart
```dart
import 'package:intl/intl.dart';

class DateUtils {
  static String formatDate(DateTime date, {String format = 'MMM d, yyyy'}) =>
    DateFormat(format).format(date);

  static String getRelativeDate(DateTime date) {
    final now = DateTime.now();
    final today = DateTime(now.year, now.month, now.day);
    final yesterday = today.subtract(const Duration(days: 1));
    final tomorrow = today.add(const Duration(days: 1));
    final input = DateTime(date.year, date.month, date.day);

    if (input == today) return 'Today';
    if (input == yesterday) return 'Yesterday';
    if (input == tomorrow) return 'Tomorrow';
    return formatDate(date);
  }

  static bool isPast(DateTime date) => date.isBefore(DateTime.now());

  static int daysBetween(DateTime from, DateTime to) {
    final start = DateTime(from.year, from.month, from.day);
    final end = DateTime(to.year, to.month, to.day);
    return end.difference(start).inDays;
  }

  static DateTime startOfWeek() {
    final now = DateTime.now();
    return now.subtract(Duration(days: now.weekday - 1));
  }
}
```

### Main Export File - `dart_utils.dart`
```dart
library dart_utils;

export 'src/string_utils.dart';
export 'src/date_utils.dart';
```

---

## Using the Package in Another Project

### Option 1: Local Path Reference
`pubspec.yaml`
```yaml
dependencies:
  dart_utils:
    path: ../dart_utils
  flutter:
    sdk: flutter
```
### Option 2: Publishing to pub.dev (For Real Production Use)

Our discussion is mainly based on option 1.

### Example Usage - `main.dart`
```dart
import 'package:dart_utils/dart_utils.dart';

void main() {
  final name = 'john doe';
  print('Capitalized: ${StringUtils.capitalize(name)}');

  final longText = 'This is a long text needing truncation.';
  print('Truncated: ${StringUtils.truncate(longText, 20)}');

  final email = 'test@example.com';
  print('Valid email: ${StringUtils.isValidEmail(email)}');

  final today = DateTime.now();
  print('Formatted date: ${DateUtils.formatDate(today)}');
  print('Relative: ${DateUtils.getRelativeDate(today)}');
}
```

### Running
```sh
dart pub get
dart run bin/main.dart
```

---

Now we are at the end of our discussion. In this discussion, we have learned about the necessity of libraries and packages in Dart, and how to create and use them effectively for code modularity, reusability, and maintainability.
