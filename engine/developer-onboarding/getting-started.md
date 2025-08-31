# Getting Started

[← Back to Documentation Home](../README.md)

Welcome to the Soccer Engine! This guide will help you set up your development environment and get your first code running.

## 📋 Prerequisites

Before you begin, ensure you have the following installed:

### Required
- **Dart SDK 3.0.0 or higher** 
  - Download from [dart.dev](https://dart.dev/get-dart)
  - Verify installation: `dart --version`
- **Git** 
  - Download from [git-scm.com](https://git-scm.com/)
  - Verify installation: `git --version`

### Recommended
- **Visual Studio Code** with Dart extension
- **Flutter SDK** (if planning to build UI components)
- **Android Studio** (for mobile development)

## 🚀 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/srogers909/soccer-engine-dart.git
cd soccer-engine-dart
```

### 2. Install Dependencies

```bash
dart pub get
```

This will install all required packages including:
- `json_annotation` - For JSON serialization
- `equatable` - For value equality
- `test` - For unit testing
- `build_runner` - For code generation

### 3. Generate Code

The project uses code generation for JSON serialization:

```bash
dart run build_runner build
```

You should see output like:
```
Built with build_runner in 3s; wrote X outputs.
```

### 4. Verify Installation

Run the test suite to ensure everything is working:

```bash
dart test
```

Expected output:
```
✓ All tests passed!
```

## 🎮 Your First Demo

### CLI Demo

Try the interactive CLI demo to explore the engine:

```bash
dart run bin/demo.dart help
```

Available commands:
- `player-demo` - Explore player creation and management
- `benchmark` - Performance testing
- `match-demo` - Match simulation example

Example:
```bash
dart run bin/demo.dart player-demo
```

### AI Demo

Experience the AI system in action:

```bash
dart run bin/ai_demo.dart
```

This showcases different GM personalities making decisions about transfers and squad management.

## 📁 Project Structure Overview

```
soccer-engine-dart/
├── lib/                    # Main library code
│   ├── soccer_engine.dart  # Public API exports
│   └── src/
│       ├── models/         # Data models (Player, Team, etc.)
│       ├── ai/            # AI systems and decision engines
│       ├── systems/       # Game systems (match simulation, etc.)
│       └── utils/         # Utility functions
├── test/                  # Test files (mirrors lib structure)
├── bin/                   # Executable demos
├── documentation/         # This documentation
└── pubspec.yaml          # Project configuration
```

## 🧪 Test-Driven Development

This project follows strict TDD methodology. Here's the basic workflow:

### 1. Red Phase - Write Failing Test
```dart
test('should calculate player overall rating correctly', () {
  final player = Player(
    id: 'test',
    name: 'Test Player',
    age: 25,
    position: PlayerPosition.midfielder,
    technical: 80,
    physical: 70,
    mental: 75,
  );
  
  expect(player.overallRating, equals(75)); // (80+70+75)/3 = 75
});
```

### 2. Green Phase - Make Test Pass
```dart
class Player {
  // ... other code
  
  int get overallRating => ((technical + physical + mental) / 3).round();
}
```

### 3. Refactor Phase - Improve Code
Optimize, clean up, and improve the implementation while keeping tests green.

## 📊 Testing Commands

```bash
# Run all tests
dart test

# Run specific test file
dart test test/models/player_test.dart

# Run with coverage
dart test --coverage=coverage
dart run coverage:format_coverage --lcov --in=coverage --out=coverage/lcov.info --packages=.dart_tool/package_config.json --report-on=lib

# Run tests in watch mode (requires `dart test` extension)
dart test --reporter=expanded
```

## 🔧 Development Tools

### Code Generation

When you modify models with `@JsonSerializable()`, regenerate code:

```bash
# Watch for changes and auto-generate
dart run build_runner watch

# One-time generation with conflict resolution
dart run build_runner build --delete-conflicting-outputs
```

### Linting

The project follows Dart style guidelines:

```bash
# Analyze code
dart analyze

# Format code
dart format .
```

## 🚨 Common Issues

### Issue: Build Runner Conflicts
**Problem**: Conflicts during code generation
**Solution**: 
```bash
dart run build_runner build --delete-conflicting-outputs
```

### Issue: Import Errors
**Problem**: Cannot resolve imports
**Solution**: 
```bash
dart pub get
dart pub deps
```

### Issue: Test Failures
**Problem**: Tests failing after changes
**Solution**: 
1. Check that all dependencies are up to date: `dart pub get`
2. Regenerate code: `dart run build_runner build`
3. Run specific failing test to debug: `dart test test/path/to/test.dart`

## 🎯 Next Steps

Now that you have the engine running:

1. **Explore the Code**: Start with `lib/src/models/player.dart` to understand the core concepts
2. **Read the Tests**: Test files in `test/` provide excellent usage examples
3. **Check Out AI Systems**: Explore `lib/src/ai/` for advanced AI functionality
4. **Learn the Workflow**: Read [Development Workflow](development-workflow.md)
5. **Understand Architecture**: Check [Project Structure](project-structure.md)

## 📞 Getting Help

- **Documentation**: Browse the complete [API Reference](../api-reference/models.md)
- **Issues**: Report bugs on [GitHub Issues](https://github.com/srogers909/soccer-engine-dart/issues)
- **Examples**: Check the `bin/` directory for comprehensive examples

---

**Navigation**: [Documentation Home](../README.md) → Developer Onboarding → Getting Started
