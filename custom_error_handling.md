# Error Handling in Dart

## Introduction

Error handling is a crucial aspect of building robust applications in any programming language, and Dart is no exception. Well-structured error handling can make your code more maintainable, easier to debug, and provide better feedback when things go wrong.

In this section, we'll explore how to:

- Create a custom error hierarchy in Dart  
- Implement effective error handling strategies  

---
## Why Error Handling Matters in Mobile Apps
In mobile apps, the environment is unpredictable: network can drop, data might be malformed, APIs may fail, and user actions can trigger unexpected behavior. If we don’t handle exceptions, the app may crash, display a blank screen, or hang — directly impacting user trust and app reviews.

- **Network Variability**: Users switch between WiFi, cellular data, and offline modes constantly. Network requests can fail due to poor connectivity, timeouts, or server issues.
  
- **Resource Constraints**: Mobile devices have limited memory, battery, and processing power. Errors related to resource management need specific handling.
  
- **User Experience Expectations**: Mobile users expect instant responses and graceful degradation when services are unavailable.
Platform-Specific Issues: Different mobile platforms (iOS, Android) may have unique error scenarios that need specialized handling.

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

## Why we need a Custom Error Hierarchy?
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

Also if you need specific UI behavior based on error type then you should choose Custom Error Hierarchy.

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
Imagine a user trying to fetch data from the server:

- Device is offline → ConnectionError

- API takes too long → TimeoutError

- Server returns 500 → ServerError

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

**AuthError**:
When a user is logged out due to token expiration or unauthorized access, a custom AuthError helps identify and handle it cleanly.
Without it, you’re stuck parsing vague 401s that can crash or silently fail. With AuthError, you can detect the issue, redirect to login, and show a clear message like “Session expired.” It ensures secure, user-friendly auth flow in your app.

```dart
class AuthError extends AppError {
  AuthError(String message, {
    String? code,
    StackTrace stackTrace,
  }) : super(message, code: code ?? 'AUTH_ERROR', stackTrace: stackTrace);
}
```
**DatabaseError**:
When users submit invalid or incomplete form data, ValidationError helps provide field-specific feedback.
Without it, you rely on generic exceptions or messy hardcoded messages.
With ValidationError, you can highlight specific fields, localize messages, and guide users to fix issues easily.
This leads to a smoother, more intuitive form experience.

```dart
class DatabaseError extends AppError {
  DatabaseError(String message, {
    String? code,
    StackTrace stackTrace,
  }) : super(message, code: code ?? 'DB_ERROR', stackTrace: stackTrace);
}
```

### Validation error
Sometime user can try to submits a form with missing or wrong data. What should you do in this sceneri? You might want to give specific feedback:

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
This helps you:
- Highlight exactly which fields are invalid
- Provide targeted error messages
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

## Error Types & Responsibilities
In a real-world mobile application, not all errors should be treated the same way. Some need user feedback, others require silent retries or redirection. Once you've defined a custom error hierarchy, it’s important to know where each error typically occurs and who should handle it — the UI layer, a service, or a global handler.

| **Error Type**       | **Where It Happens**             | **Handled In...**             |
|----------------------|----------------------------------|-------------------------------|
| `ConnectionError`    | Network failure                  | UI or error handler           |
| `TimeoutError`       | Slow or delayed API responses    | Error handler                 |
| `AuthError`          | Token expired, unauthorized      | Middleware or UI              |
| `ValidationError`    | Form submission or bad input     | UI (form/controller)          |
| `DatabaseError`      | Local DB failure (SQLite, Hive)  | Storage service or error handler |
| `AppError`           | Unexpected app-level failure     | Global error handler          |


## Best Practices

### 1. Using `rethrow` for Error Propagation
The rethrow statement is useful when you want to catch an exception, perform some action, but still propagate the exception up the call stack.

**When to Use rethrow?**

Use it when:

- You want to log or process the error temporarily
- You don’t want to swallow it—just pass it along

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

**When to Use finally**

Use it to:
- Clean up resources (close DB, stop loading indicator)
- Execute code regardless of success or error

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
