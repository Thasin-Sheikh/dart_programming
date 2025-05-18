
# ডার্টে এরর হ্যান্ডলিং

## Introduction
যেকোনো প্রোগ্রামিং ভাষায় রোবাস্ট অ্যাপ্লিকেশন তৈরি করার ক্ষেত্রে এরর হ্যান্ডলিং অত্যন্ত গুরুত্বপূর্ণ। ডার্টও এর ব্যতিক্রম নয়। সঠিকভাবে পরিকল্পিত এরর হ্যান্ডলিং কোডকে আরও রক্ষণযোগ্য এবং সহজবোধ্য করার পাশাপাশি সমস্যা দেখা দিলে কার্যকরভাবে সাড়া দেওয়া এবং উন্নত ফিডব্যাক প্রদান করে।

এই অংশে আমরা আলোচনা করবো:

- কীভাবে ডার্টে কাস্টম এরর হায়ারার্কি তৈরি করবেন
- কার্যকর এরর হ্যান্ডলিং কৌশল প্রয়োগ করব

## Understanding Dart's Error Types
ডার্টে মূলত দুই ধরনের এরর রয়েছে:

- **Exceptions:** এগুলো রানটাইমে ধরা যায় এবং হ্যান্ডেল করা যায়। এগুলো Exception ইন্টারফেস ইমপ্লিমেন্ট করে।
- **Errors:** এগুলো সাধারণত গুরুতর প্রোগ্রাম ব্যর্থতা নির্দেশ করে এবং Error ইন্টারফেস ইমপ্লিমেন্ট করে। এগুলো বাগ নির্দেশ করে যেগুলো ধরার পরিবর্তে ঠিক করা উচিত।

## Example of Built-in Error Handling
ডার্ট পূর্ব থেকেই এরর হ্যান্ডলিংয়ের জন্য কিছু সুবিধা প্রদান করে:

```dart
try {
  // এমন কোড যা এক্সেপশন ঘটাতে পারে
} on FormatException {
  // ফরম্যাট এক্সেপশন হ্যান্ডলিং
} on Exception {
  // অন্যান্য এক্সেপশন হ্যান্ডলিং
} on Error {
  // গুরুতর এরর হ্যান্ডলিং (সাধারণত সুপারিশকৃত নয়)
} catch (e) {
  // যেকোনো থ্রো করা অবজেক্ট ধরবে
}
```

## Custom Error Hierarchy

### Why Build a Custom Error Hierarchy?

কাস্টম এরর হায়ারার্কি তৈরি করার কিছু সুবিধা রয়েছে:

- **Granular Type-Safe Handling:** নির্দিষ্ট এরর ধরতে পারবেন, ভিন্নভাবে হ্যান্ডেল করতে পারবেন, এবং সম্পর্কিত এররগুলোকে যৌক্তিকভাবে গোষ্ঠীভুক্ত করতে পারবেন।
- **Consistent Structured Errors:** সকল এররের জন্য একই ডিজাইন প্যাটার্ন, প্রয়োজনীয় তথ্য অন্তর্ভুক্ত, স্ট্যান্ডার্ড প্রপার্টি (মেসেজ, কোড, স্ট্যাক ট্রেস ইত্যাদি) থাকবে।
- **Improved Debugging and Maintenance:** কী ভুল হলো এবং কেন, তার বেশি প্রসঙ্গ থাকবে, স্ট্যাক ট্রেস এবং এরর চেইন সংরক্ষিত থাকবে, এবং এরর উৎস স্পষ্টভাবে শ্রেণিবদ্ধ হবে।

## Building an Error Hierarchy in Dart

### Base Error Class

```dart
abstract class AppError implements Exception {
  final String message;
  final String? code;
  final StackTrace? stackTrace;

  AppError(this.message, {this.code, this.stackTrace});

  @override
  String toString() => 'AppError: $code - $message';
}
```

### More Specific Network Errors

```dart
class NetworkError extends AppError {
  final int? statusCode;

  NetworkError(String message, {
    this.statusCode,
    String? code,
    StackTrace? stackTrace,
  }) : super(message, code: code ?? 'NETWORK_ERROR', stackTrace: stackTrace);
}
```

### Domain Specific Errors

```dart
class AuthError extends AppError {
  AuthError(String message, {
    String? code,
    StackTrace? stackTrace,
  }) : super(message, code: code ?? 'AUTH_ERROR', stackTrace: stackTrace);
}
```

### ValidationError

```dart
class ValidationError extends AppError {
  final Map<String, String>? fieldErrors;

  ValidationError(String message, {
    this.fieldErrors,
    String? code,
    StackTrace? stackTrace,
  }) : super(message, code: code ?? 'VALIDATION_ERROR', stackTrace: stackTrace);
}
```

### Best Practices

1. Using `rethrow` for Error Propagation

```dart
Future<void> processData(String data) async {
  try {
    await parseData(data);
  } catch (e) {
    print('ডেটা প্রসেসিংয়ে এরর: $e');
    rethrow;
  }
}
```

2. Async Error Handling with try-catch-finally

```dart
Future<void> loadUserData(String userId) async {
  try {
    final userData = await fetchUserFromDatabase(userId);
    processUserData(userData);
  } catch (e) {
    print('অজানা এরর: $e');
  } finally {
    closeConnection();
  }
}
```

## Conclusion

ডার্টে কাস্টম এরর হায়ারার্কি তৈরি করলে কোডের সংগঠন উন্নত হয়, ডিবাগিং সহজ হয়, এবং ডেভেলপারদের জন্য ব্যবহারযোগ্যতা বৃদ্ধি পায়।

একটি সুষ্ঠু এরর হ্যান্ডলিং পদ্ধতি:

- বিভিন্ন এরর পরিস্থিতিতে উপযুক্ত প্রতিক্রিয়া প্রদানের সুযোগ দেয়
- স্পষ্ট পুনরুদ্ধার পথ তৈরি করে
- রক্ষণাবেক্ষণ ও সামঞ্জস্যতা বাড়ায়

মনে রাখবেন, ভালো এরর হ্যান্ডলিং শুধুমাত্র এক্সেপশন ধরে রাখা নয়, এটি এমন একটি ব্যবস্থা তৈরি করা যা সৃষ্টিশীল উপায়ে ব্যর্থতা সামলাতে পারে এবং সমস্যা সম্পর্কে পরিষ্কার তথ্য প্রদান করতে পারে।
