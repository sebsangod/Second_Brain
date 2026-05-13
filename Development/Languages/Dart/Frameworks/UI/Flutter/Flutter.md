---
aliases:
  - Flutter
tags:
  - learning
  - dev/lang/dart
date: 2026-03-15
---
**Sources**: [Flutter](https://flutter.dev/)

**Related:** [[Dart]]

---

## Description

_Flutter_ is an **open-source UI software development kit (SDK)** created by **Google** that allows you to build natively compiled applications for multiple platforms from a single codebase.

---

## Key Concepts

- **Single Codebase**: Write code once and deploy it across iOS, Android, web, Windows, macOS, and Linux.

- **Dart Programming Language**: Flutter uses ``Dart``, an object-oriented language also developed by **Google**, designed for fast app performance and easy learning.

- **Hot Reload**: Developers can see code changes instantly in the running app without losing its state, which significantly speeds up development.

- **Everything is a Widget**: The core building blocks of a _Flutter_ UI are **widgets**. These describe what the view should look like given its current configuration and state.

- **Custom Rendering Engine**: Unlike other frameworks that use platform-specific components, _Flutter_ uses its own high-performance rendering engine (Skia or Impeller) to draw every pixel on the screen


---

## Examples

### Hello World!

```dart title:main.dart
import 'package:flutter/material.dart';

void main() {
  runApp(
    const MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text('Hello World!'),
        ),
      ),
    ),
  );
}

```

---

## Claude Sessions
