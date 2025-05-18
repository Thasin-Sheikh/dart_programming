
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

```plaintext
AppError (abstract base)
├── NetworkError
│   ├── NoConnectionError
│   ├── TimeoutError
│   └── ApiError
├── DataError
│   ├── NotFoundError
│   ├── ValidationError
│   └── StorageError
└── BusinessError
    ├── UnauthorizedError
```

---

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

---
### Throwing and Catching Custom Error
এখন যেহেতু আমাদের এরর হায়ারার্কি তৈরি হয়ে গেছে, আসুন দেখি ডার্ট অ্যাপ্লিকেশনগুলিতে এটি কীভাবে আরও ভালোভাবে কাজে লাগানো যায়.

```dart
Future<Map<String, dynamic>> fetchData(String url) async {
  try {
    if (url.isEmpty) {
      throw ValidationError('URL cannot be empty');
    }
    if (!url.startsWith('https://')) {
      throw ValidationError('URL must use HTTPS protocol');
    }
    if (url.contains('offline')) {
      throw ConnectionError('Failed to connect to server');
    }
    if (url.contains('private')) {
      throw AuthError'Not authorized to access this resource');
    }
    if (url.contains('error')) {
      throw ServerError('Internal server error', statusCode: 500);
    }
    return {'status': 'success', 'data': 'Some data'};
  } catch (e) {
    if (e is! AppError) {
      throw AppError('Unexpected error: ${e.toString()}');
    }
    rethrow;
  }
}
```

### Catching and Handling Error
যেভাবে আপনি বিভিন্ন ধরনের এরর হ্যান্ডেল করতে পারেন:

```dart
void main() async {
  try {
    final data = await fetchData('https://api.example.com/data');
    print('Received data: $data');
  } on ValidationError catch (e) {
    print('Validation error: ${e.message}');
    if (e.fieldErrors != null) {
      for (final entry in e.fieldErrors!.entries) {
        print('- ${entry.key}: ${entry.value}');
      }
    }
  } on AuthError catch (e) {
    print('Authentication error: ${e.message}');
    login();
  } on ConnectionError catch (e) {
    print('Connection error: ${e.message}');
    checkNetworkConnection();
  } on NetworkError catch (e) {
    print('Network error (${e.statusCode}): ${e.message}');
    if (e.statusCode == 500) {
      retry();
    }
  } on AppError catch (e) {
    print('Application error: ${e.message}');
  } catch (e) {
    print('Unknown error: $e');
  }
}

void login() => print('Redirecting to login...');
void checkNetworkConnection() => print('Checking network connection...');
void retry() => print('Retrying operation...');
```

---

## Centralized Error Handler
বড় অ্যাপ্লিকেশনগুলোর জন্য centralized error handler তৈরি করা যেতে পারে:

```dart
import 'package:logger/logger.dart';

final logger = Logger();

class ErrorHandler {
  static void handle(Object error, {StackTrace? stackTrace}) {
    _logError(error, stackTrace);
    switch (error.runtimeType) {
      case ValidationError:
        _handleValidationError(error as ValidationError);
        break;
      case AuthError:
        _handleAuthError(error as AuthError);
        break;
      case NetworkError:
        _handleNetworkError(error as NetworkError);
        break;
      case AppError:
        _handleAppError(error as AppError);
        break;
      default:
        _handleUnknownError(error);
    }
  }

  static void _logError(Object error, StackTrace? stackTrace) {
    logger.e('Unhandled Error', error, stackTrace);
  }

  static void _handleValidationError(ValidationError error) {
    logger.w('Validation: ${error.message}');
  }

  static void _handleAuthError(AuthError error) {
    logger.w('Auth: ${error.message}');
  }

  static void _handleNetworkError(NetworkError error) {
    logger.w('Network: ${error.message}');
  }

  static void _handleAppError(AppError error) {
    logger.w('App: ${error.message}');
  }

  static void _handleUnknownError(Object error) {
    logger.w('Unknown error: $error');
  }
}

```

---

## Zone-Based Error Handling
Zones মূলত **uncaught errors** isolate করার জন্য ব্যবহার করা হয়, বিশেষ করে Flutter অ্যাপ, CLI tools বা isolates এ। এগুলো যেকোনো unhandled error গ্লোবালি ধরতে পারে এবং কেন্দ্রীয়ভাবে রিপোর্ট বা লগ করতে পারে।

```dart
import 'dart:async';

void main() {
  runZonedGuarded(() {
    throw Exception('Unhandled exception');
  }, (error, stackTrace) {
    print('Caught error in zone: $error');
    print(stackTrace);
    ErrorHandler.handle(error, stackTrace: stackTrace);
  });
}
```

---

## Conclusion
Dart-এ কাস্টম error hierarchy তৈরি করলে কোডের organization উন্নত হয়, debugging সহজ হয় এবং developer experience বাড়ে।

structured error handling-এর মাধ্যমে আপনি:

- বিভিন্ন error scenario-এ সঠিকভাবে প্রতিক্রিয়া জানাতে পারবেন
- recovery path স্পষ্ট করতে পারবেন
- maintainability ও consistency বাড়াতে পারবেন
  
মনে রাখবেন, ভালো error handling মানে শুধু exception ধরা নয়—এটি এমন একটি ব্যবস্থা তৈরি করা যা gracefully failure handle করে এবং কী ভুল হয়েছে তা স্পষ্ট ভাবে জানায়।
