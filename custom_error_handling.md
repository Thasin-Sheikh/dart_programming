# Error Handling in Dart

## Introduction

Error handling is a crucial aspect of building robust applications in any programming language, and Dart is no exception. Well-structured error handling can make your code more maintainable, easier to debug, and provide better feedback when things go wrong.

In this section, we'll explore how to:

- Create a custom error hierarchy in Dart  
- Implement effective error handling strategies  

---

## Understanding Dart's Error Types

Dart has two main categories of errors:

- **Exceptions**: Errors that can be caught and handled at runtime. These implement the `Exception` interface.
- **Errors**: Represent serious program failures and implement the `Error` interface. They usually indicate bugs that should be fixed rather than caught.

### Example of Built-in Error Handling
Before we create our own error hierarchy, let's understand what Dart provides out of the box:

```dart
try {
  // Code that might cause an exception
} on FormatException {
  // Handle format exceptions
} on Exception {
  // Handle other exceptions
} on Error {
  // Handle serious errors (generally not recommended)
} catch (e) {
  // Handle any other thrown object
}
```

---

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

## Why Build a Custom Error Hierarchy?
Creating your own error hierarchy offers several benefits:

- **Granular Type-Safe Handling**  
  - Catch specific error types
  - Handle them differently
  - Group related errors logically

- **Consistent Structured Errors**  
  - Same design pattern for all errors
  - Include relevant context in each error
  - Standardized properties (`message`, `code`, `stackTrace`, etc.)

- **Improved Debugging and Maintenance**  
  - More context about what went wrong and why
  - Preserved stack traces and error chains
  - Clear categorization of error sources

---

## Building an Error Hierarchy in Dart

### Base Error Class
Dart's support for class inheritance makes it perfect for creating a structured error hierarchy. Let's start by creating a base error class:

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

### More Specific Network Error

```dart
class NetworkError extends AppError {
  final int? statusCode;

  NetworkError(String message, {
    this.statusCode,
    String? code,
    StackTrace? stackTrace,
  }) : super(message, code: code ?? 'NETWORK_ERROR', stackTrace: stackTrace);

  @override
  String toString() => 'NetworkError: $code - $message (Status: $statusCode)';
}
class ServerError extends NetworkError {
  ServerError(String message, {
    int? statusCode,
    String? code,
    dynamic stackTrace,
  }) : super(message,
             statusCode: statusCode,
             code: code ?? 'SERVER_ERROR',
             stackTrace: stackTrace);
}

class ConnectionError extends NetworkError {
  ConnectionError(String message, {
    String? code,
    StackTrace stackTrace,
  }) : super(message,
             statusCode: null,
             code: code ?? 'CONNECTION_ERROR',
             stackTrace: stackTrace);
}
```
### Domain Specific Errors

```dart
class AuthError extends AppError {
  AuthError(String message, {
    String? code,
    StackTrace stackTrace,
  }) : super(message, code: code ?? 'AUTH_ERROR', stackTrace: stackTrace);
}

class DatabaseError extends AppError {
  DatabaseError(String message, {
    String? code,
    StackTrace stackTrace,
  }) : super(message, code: code ?? 'DB_ERROR', stackTrace: stackTrace);
}
```

### Validation error

```dart
class ValidationError extends AppError {
  final Map<String, String>? fieldErrors;
  ValidationError(String message, {
    this.fieldErrors,
    String? code,
    StackTrace stackTrace,
  }) : super(message, code: code ?? 'VALIDATION_ERROR', stackTrace: stackTrace);
}
```

---


### Throwing and Catching Custom Error
Now that we have our error hierarchy, let's look at how to use it effectively in Dart applications.

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
Here's how you can handle different error types:

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

## Best Practices

### 1. Using `rethrow` for Error Propagation
The rethrow statement is useful when you want to catch an exception, perform some action, but still propagate the exception up the call stack.

```dart
Future<void> processData(String data) async {
  try {
    await parseData(data);
  } catch (e) {
    print('Error while processing data: $e');
    rethrow;
  }
}
```

### 2. Async Error Handling with `try-catch-finally`
When working with asynchronous code in Dart, error handling works the same way as with synchronous code:

```dart
Future<void> loadUserData(String userId) async {
  try {
    final userData = await fetchUserFromDatabase(userId);
    processUserData(userData);
  } on DatabaseError catch (e) {
    print('Database error: ${e.message}');
  } catch (e) {
    print('Unknown error: $e');
  } finally {
    closeConnection();
  }
}
```

---

## Centralized Error Handler
For more complex applications, you might want to implement a centralized error handler:

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
Zones are useful for isolating **uncaught errors**, especially in Flutter apps, CLI tools, or isolates. They can catch any unhandled error globally and report/log them centrally.

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

Implementing a custom error hierarchy in Dart improves code organization, enhances debugging, and provides a better developer experience.  

By creating a structured approach to error handling, you can:

- Respond appropriately to different error scenarios  
- Provide clear paths to recovery  
- Improve maintainability and consistency

Remember that good error handling is not just about catching exceptions—it's about creating a system that gracefully handles failures and provides clear information about what went wrong
