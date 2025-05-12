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
    └── PaymentRequiredError
```

---

## Why Build a Custom Error Hierarchy?

-  **Granular Type-Safe Handling**  
  - Catch specific error types
  - Handle them differently
  - Group related errors logically

-  **Consistent Structured Errors**  
  - Same design pattern for all errors
  - Include relevant context in each error
  - Standardized properties (`message`, `code`, `stackTrace`, etc.)

-  **Improved Debugging and Maintenance**  
  - More context
  - Preserved stack traces
  - Clear categorization

---

## Building an Error Hierarchy in Dart

### Base Error Class

```dart
class AppException implements Exception {
  final String message;
  final String? code;
  final dynamic stackTrace;

  AppException(this.message, {this.code, this.stackTrace});

  @override
  String toString() => 'AppException: $code - $message';
}
```

### Specific Error Types

```dart
class NetworkException extends AppException {
  final int? statusCode;
  NetworkException(String message, {
    String? code,
    this.statusCode,
    dynamic stackTrace,
  }) : super(message, code: code ?? 'NETWORK_ERROR', stackTrace: stackTrace);

  @override
  String toString() => 'NetworkException: $code - $message (Status: $statusCode)';
}

class AuthException extends AppException {
  AuthException(String message, {
    String? code,
    dynamic stackTrace,
  }) : super(message, code: code ?? 'AUTH_ERROR', stackTrace: stackTrace);
}

class DatabaseException extends AppException {
  DatabaseException(String message, {
    String? code,
    dynamic stackTrace,
  }) : super(message, code: code ?? 'DB_ERROR', stackTrace: stackTrace);
}

class ValidationException extends AppException {
  final Map<String, String>? fieldErrors;
  ValidationException(String message, {
    this.fieldErrors,
    String? code,
    dynamic stackTrace,
  }) : super(message, code: code ?? 'VALIDATION_ERROR', stackTrace: stackTrace);
}
```

### More Specific Network Exceptions

```dart
class ServerException extends NetworkException {
  ServerException(String message, {
    int? statusCode,
    String? code,
    dynamic stackTrace,
  }) : super(message,
             statusCode: statusCode,
             code: code ?? 'SERVER_ERROR',
             stackTrace: stackTrace);
}

class ConnectionException extends NetworkException {
  ConnectionException(String message, {
    String? code,
    dynamic stackTrace,
  }) : super(message,
             statusCode: null,
             code: code ?? 'CONNECTION_ERROR',
             stackTrace: stackTrace);
}
```

---

## Throwing and Catching Custom Exceptions

### Throwing Custom Exceptions

```dart
Future<Map<String, dynamic>> fetchData(String url) async {
  try {
    if (url.isEmpty) {
      throw ValidationException('URL cannot be empty');
    }
    if (!url.startsWith('https://')) {
      throw ValidationException('URL must use HTTPS protocol');
    }
    if (url.contains('offline')) {
      throw ConnectionException('Failed to connect to server');
    }
    if (url.contains('private')) {
      throw AuthException('Not authorized to access this resource');
    }
    if (url.contains('error')) {
      throw ServerException('Internal server error', statusCode: 500);
    }
    return {'status': 'success', 'data': 'Some data'};
  } catch (e) {
    if (e is! AppException) {
      throw AppException('Unexpected error: ${e.toString()}');
    }
    rethrow;
  }
}
```

### Catching and Handling Exceptions

```dart
void main() async {
  try {
    final data = await fetchData('https://api.example.com/data');
    print('Received data: $data');
  } on ValidationException catch (e) {
    print('Validation error: ${e.message}');
    if (e.fieldErrors != null) {
      for (final entry in e.fieldErrors!.entries) {
        print('- ${entry.key}: ${entry.value}');
      }
    }
  } on AuthException catch (e) {
    print('Authentication error: ${e.message}');
    login();
  } on ConnectionException catch (e) {
    print('Connection error: ${e.message}');
    checkNetworkConnection();
  } on NetworkException catch (e) {
    print('Network error (${e.statusCode}): ${e.message}');
    if (e.statusCode == 500) {
      retry();
    }
  } on AppException catch (e) {
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

```dart
Future<void> loadUserData(String userId) async {
  try {
    final userData = await fetchUserFromDatabase(userId);
    processUserData(userData);
  } on DatabaseException catch (e) {
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

```dart
class ErrorHandler {
  static void handle(Object error, {StackTrace? stackTrace}) {
    _logError(error, stackTrace);

    if (error is ValidationException) {
      _handleValidationError(error);
    } else if (error is AuthException) {
      _handleAuthError(error);
    } else if (error is NetworkException) {
      _handleNetworkError(error);
    } else if (error is AppException) {
      _handleAppError(error);
    } else {
      _handleUnknownError(error);
    }
  }

  static void _logError(Object error, StackTrace? stackTrace) {
    print('ERROR: $error');
    if (stackTrace != null) print(stackTrace);
  }

  static void _handleValidationError(ValidationException error) =>
      print('Handling validation error: ${error.message}');

  static void _handleAuthError(AuthException error) =>
      print('Handling auth error: ${error.message}');

  static void _handleNetworkError(NetworkException error) =>
      print('Handling network error: ${error.message}');

  static void _handleAppError(AppException error) =>
      print('Handling application error: ${error.message}');

  static void _handleUnknownError(Object error) =>
      print('Handling unknown error: $error');
}
```

---

## Zone-Based Error Handling

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
