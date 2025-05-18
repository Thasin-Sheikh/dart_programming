# Dart লাইব্রেরি এবং প্যাকেজ (মডুলার)

## লাইব্রেরি এবং প্যাকেজ কি?

**লাইব্রেরি:**  
ফাংশন, ক্লাস এবং ভেরিয়েবলের একটি সংগ্রহ যা এক বা একাধিক ফাইলে গোষ্ঠীবদ্ধ থাকে।

**প্যাকেজ:**  
একাধিক লাইব্রেরির সাথে সংস্করণ এবং নির্ভরতাসহ মেটাডেটা সম্বলিত একটি বান্ডেল।

### কেন লাইব্রেরি এবং প্যাকেজ ব্যবহার করবেন?

- **কোড সংগঠন:** সম্পর্কিত কোড একত্রে গোষ্ঠীবদ্ধ করা।  
- **পুনঃব্যবহারযোগ্যতা:** একাধিক অ্যাপ জুড়ে কোড পুনঃব্যবহার করা।  
- **রক্ষণাবেক্ষণযোগ্যতা:** সহজে পরিচালনা এবং ডিবাগ করা।  
- **মডুলারিটি:** উদ্বেগের বিচ্ছিন্নতা সক্ষম করা।  
- **দলগত সহযোগিতা:** বিতরণমূলক উন্নয়ন সমর্থন।  

---

## লাইব্রেরি তৈরি এবং ব্যবহার

### ১. একটি বেসিক লাইব্রেরি তৈরি

প্রতিটি Dart ফাইল নিজেই একটি লাইব্রেরি। উদাহরণস্বরূপ:

```dart
// lib/temperature_converter.dart
// এটি একটি স্বয়ংসম্পূর্ণ লাইব্রেরি হিসেবে কাজ করে

/// সেলসিয়াসকে ফারেনহাইটে রূপান্তর করে
double celsiusToFahrenheit(double celsius) => (celsius * 9 / 5) + 32;

/// ফারেনহাইটকে সেলসিয়াসে রূপান্তর করে
double fahrenheitToCelsius(double fahrenheit) => (fahrenheit - 32) * 5 / 9;

/// একটি ক্লাস যা বিস্তারিত তাপমাত্রা রূপান্তর প্রদান করে
class TemperatureConverter {
  double toFahrenheit(double celsius) => celsiusToFahrenheit(celsius);
  double toCelsius(double fahrenheit) => fahrenheitToCelsius(fahrenheit);

  String describeConversion(double value, String fromUnit) {
    if (fromUnit.toLowerCase() == 'c') {
      return '$value°C হল ${toFahrenheit(value)}°F';
    } else if (fromUnit.toLowerCase() == 'f') {
      return '$value°F হল ${toCelsius(value)}°C';
    } else {
      return 'অজানা একক';
    }
  }
}
```

### 2. একটি লাইব্রেরি আমদানি এবং ব্যবহার

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

## লাইব্রেরি ডিরেক্টিভ
Dart লাইব্রেরি নির্ভরতা এবং দৃশ্যমানতা পরিচালনার জন্য বেশ কিছু নির্দেশনা প্রদান করে। সেগুলি শক্তিশালী হলেও সংযমের সাথে ব্যবহার করা উচিত:

- **part/part of**: অতিরিক্ত ব্যবহার করলে কোড রক্ষণাবেক্ষণ কঠিন হতে পারে।

- **export**: API সরলীকরণের জন্য ফ্যাসাদ লাইব্রেরি তৈরি করতে কার্যকর।

- **উপসর্গ (as)**: নাম দ্বন্দ্ব এড়াতে অপরিহার্য।

### 1. `import`
লাইব্রেরি আমদানি করা যায়:
```dart
import 'package:your_project/library.dart'; // Local
import 'dart:math';                         // Dart core
import 'package:http/http.dart';           // External
```

### 2. `part` and `part of`
একাধিক ফাইলে লাইব্রেরি বিভক্ত করা:
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
একাধিক ফাইলকে একত্রিত করে একটি API হিসেবে প্রকাশ করা:
```dart
// lib/math_library.dart
export 'src/geometry.dart';
export 'src/algebra.dart';
export 'src/statistics.dart';
```

---

## Library Prefixes
নাম দ্বন্দ্ব এড়াতে:
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
নির্দিষ্ট উপাদান আমদানি বা আড়াল করতে:
```dart
import 'temperature_converter.dart' show celsiusToFahrenheit, fahrenheitToCelsius;
import 'temperature_converter.dart' hide TemperatureConverter;
```

---

## একটি পুনঃব্যবহারযোগ্য Dart প্যাকেজ তৈরি (`dart_utils`)

### ডিরেক্টরি গঠন
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

### প্রধান এক্সপোর্ট ফাইল - `dart_utils.dart`
```dart
library dart_utils;

export 'src/string_utils.dart';
export 'src/date_utils.dart';
```

---

## অন্য প্রকল্পে প্যাকেজ ব্যবহার করা

### Option 1: লোকাল পাথ রেফারেন্স
`pubspec.yaml`
```yaml
dependencies:
  dart_utils:
    path: ../dart_utils
  flutter:
    sdk: flutter
```
### Option 2: pub.dev এ প্রকাশ (প্রোডাকশন ব্যবহারের জন্য)
আমাদের আলোচনায় মূলত বিকল্প ১ ভিত্তিক

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

### চলুন এবার রান দিয়ে দেখা যাক।
```sh
dart pub get
dart run bin/main.dart
```

---

এই আলোচনায় আমরা Dart এ লাইব্রেরি এবং প্যাকেজের প্রয়োজনীয়তা, কীভাবে সেগুলো তৈরি এবং ব্যবহার করতে হয় তা শিখেছি। এর মাধ্যমে আমরা কোডকে মডুলার, পুনঃব্যবহারযোগ্য এবং রক্ষণাবেক্ষণযোগ্য করে তুলতে পারি।
