# Contributing Guide

[← Back to Documentation](../README.md) | [← Back to Developer Onboarding](../README.md#developer-onboarding)

## Welcome Contributors!

Thank you for your interest in contributing to the Soccer Engine project! This guide will help you get started with contributing code, reporting issues, and improving the project.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Contribution Types](#contribution-types)
- [Code Standards](#code-standards)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)
- [Pull Request Process](#pull-request-process)
- [Issue Reporting](#issue-reporting)
- [Code Review Guidelines](#code-review-guidelines)

---

## Code of Conduct

We are committed to providing a welcoming and inclusive environment for all contributors. Please be respectful, constructive, and collaborative in all interactions.

### Expected Behavior

- Use welcoming and inclusive language
- Respect differing viewpoints and experiences
- Accept constructive criticism gracefully
- Focus on what's best for the community and project
- Show empathy towards other contributors

### Unacceptable Behavior

- Harassment, discrimination, or intimidation
- Trolling, insulting, or derogatory comments
- Publishing private information without permission
- Any other conduct that would be inappropriate in a professional setting

---

## Getting Started

### Prerequisites

Before contributing, ensure you have:

1. **Dart SDK** (3.0.0 or later)
2. **Git** for version control
3. **IDE** (VS Code with Dart extension recommended)
4. **Flutter** (if working on UI components)

### Initial Setup

```bash
# Fork and clone the repository
git clone https://github.com/YOUR_USERNAME/soccer-engine-dart.git
cd soccer-engine-dart

# Install dependencies
dart pub get

# Run tests to ensure everything works
dart test

# Run the CLI demo
dart run bin/demo.dart

# Run the AI demo
dart run bin/ai_demo.dart
```

---

## Development Workflow

We follow a Test-Driven Development (TDD) approach. See [Development Workflow](development-workflow.md) for detailed TDD guidelines.

### Branch Strategy

```bash
# Create feature branch from main
git checkout main
git pull origin main
git checkout -b feature/your-feature-name

# Work on your changes...

# Push and create pull request
git push origin feature/your-feature-name
```

### Branch Naming Convention

- **Features**: `feature/description-of-feature`
- **Bug fixes**: `fix/description-of-fix`
- **Documentation**: `docs/description-of-changes`
- **Refactoring**: `refactor/description-of-refactor`
- **Tests**: `test/description-of-test-improvements`

---

## Contribution Types

### 🚀 Feature Development

Adding new functionality to the Soccer Engine:

- New player models or attributes
- Enhanced AI decision-making
- Additional match simulation features
- New tactical systems
- Financial management improvements

### 🐛 Bug Fixes

Fixing issues in existing code:

- Logic errors in simulations
- Data model inconsistencies
- Performance problems
- Memory leaks
- Calculation errors

### 📚 Documentation

Improving project documentation:

- API documentation
- Code examples
- Tutorial content
- Developer guides
- README improvements

### 🧪 Testing

Enhancing test coverage and quality:

- Unit tests for new features
- Integration tests
- Performance tests
- Edge case testing
- Test utilities

### 🔧 Infrastructure

Improving development infrastructure:

- Build system improvements
- CI/CD enhancements
- Development tooling
- Code quality tools
- Performance monitoring

---

## Code Standards

### Dart Style Guide

We follow the [official Dart style guide](https://dart.dev/guides/language/effective-dart) with these specific requirements:

#### Naming Conventions

```dart
// Classes: PascalCase
class PlayerTransfer { }

// Variables and functions: camelCase
final String playerName = 'Messi';
void calculateRating() { }

// Constants: camelCase with const
const int maxPlayers = 25;

// Enums: PascalCase with camelCase values
enum PlayerPosition {
  goalkeeper,
  defender,
  midfielder,
  forward,
}

// Files: snake_case
// player_transfer.dart
// match_simulator.dart
```

#### Code Organization

```dart
// Import order: dart, flutter, packages, relative
import 'dart:math';

import 'package:equatable/equatable.dart';
import 'package:json_annotation/json_annotation.dart';

import '../models/player.dart';
import 'team.dart';
```

#### Documentation

```dart
/// Brief description of the class or function.
/// 
/// Longer description if needed, explaining the purpose,
/// usage patterns, and important considerations.
/// 
/// Example:
/// ```dart
/// final player = Player(
///   id: 'player_001',
///   name: 'Lionel Messi',
///   position: PlayerPosition.forward,
/// );
/// ```
class Player {
  /// The unique identifier for this player.
  final String id;
  
  /// The player's display name.
  final String name;
}
```

### Architecture Patterns

#### Immutable Data Models

```dart
@JsonSerializable()
class Player extends Equatable {
  final String id;
  final String name;
  
  const Player({
    required this.id,
    required this.name,
  });
  
  /// Creates a copy with updated values
  Player copyWith({
    String? id,
    String? name,
  }) {
    return Player(
      id: id ?? this.id,
      name: name ?? this.name,
    );
  }
  
  @override
  List<Object> get props => [id];
}
```

#### Validation Patterns

```dart
class Player {
  factory Player({
    required String id,
    required String name,
    required int rating,
  }) {
    if (id.isEmpty) {
      throw ArgumentError('Player ID cannot be empty');
    }
    if (name.isEmpty) {
      throw ArgumentError('Player name cannot be empty');
    }
    if (rating < 1 || rating > 100) {
      throw ArgumentError('Player rating must be between 1 and 100');
    }
    
    return Player._internal(
      id: id,
      name: name,
      rating: rating,
    );
  }
  
  const Player._internal({
    required this.id,
    required this.name,
    required this.rating,
  });
}
```

#### Error Handling

```dart
// Use specific exception types
class InvalidPlayerRatingException implements Exception {
  final String message;
  const InvalidPlayerRatingException(this.message);
  
  @override
  String toString() => 'InvalidPlayerRatingException: $message';
}

// Handle errors gracefully
Result<Player> createPlayer(Map<String, dynamic> data) {
  try {
    return Result.success(Player.fromJson(data));
  } on FormatException catch (e) {
    return Result.failure('Invalid player data format: $e');
  } on ArgumentError catch (e) {
    return Result.failure('Invalid player arguments: $e');
  }
}
```

---

## Testing Requirements

### Test Coverage

We maintain **100% test coverage**. Every contribution must include comprehensive tests.

#### Required Test Types

1. **Unit Tests**: Test individual methods and classes
2. **Integration Tests**: Test component interactions
3. **Property-Based Tests**: Test with random inputs
4. **Edge Case Tests**: Test boundary conditions

#### Test Structure

```dart
import 'package:test/test.dart';
import 'package:soccer_engine/soccer_engine.dart';

void main() {
  group('Player', () {
    group('constructor', () {
      test('creates player with valid data', () {
        final player = Player(
          id: 'player_001',
          name: 'Lionel Messi',
          position: PlayerPosition.forward,
          overallRating: 95,
        );
        
        expect(player.id, equals('player_001'));
        expect(player.name, equals('Lionel Messi'));
        expect(player.position, equals(PlayerPosition.forward));
        expect(player.overallRating, equals(95));
      });
      
      test('throws ArgumentError for empty id', () {
        expect(
          () => Player(
            id: '',
            name: 'Messi',
            position: PlayerPosition.forward,
            overallRating: 95,
          ),
          throwsA(isA<ArgumentError>()),
        );
      });
      
      test('throws ArgumentError for invalid rating', () {
        expect(
          () => Player(
            id: 'player_001',
            name: 'Messi',
            position: PlayerPosition.forward,
            overallRating: 101, // Invalid rating
          ),
          throwsA(isA<ArgumentError>()),
        );
      });
    });
    
    group('copyWith', () {
      test('creates copy with updated values', () {
        final original = Player(
          id: 'player_001',
          name: 'Messi',
          position: PlayerPosition.forward,
          overallRating: 95,
        );
        
        final updated = original.copyWith(overallRating: 96);
        
        expect(updated.id, equals(original.id));
        expect(updated.name, equals(original.name));
        expect(updated.position, equals(original.position));
        expect(updated.overallRating, equals(96));
      });
    });
    
    group('JSON serialization', () {
      test('serializes to JSON correctly', () {
        final player = Player(
          id: 'player_001',
          name: 'Messi',
          position: PlayerPosition.forward,
          overallRating: 95,
        );
        
        final json = player.toJson();
        
        expect(json['id'], equals('player_001'));
        expect(json['name'], equals('Messi'));
        expect(json['position'], equals('forward'));
        expect(json['overallRating'], equals(95));
      });
      
      test('deserializes from JSON correctly', () {
        final json = {
          'id': 'player_001',
          'name': 'Messi',
          'position': 'forward',
          'overallRating': 95,
        };
        
        final player = Player.fromJson(json);
        
        expect(player.id, equals('player_001'));
        expect(player.name, equals('Messi'));
        expect(player.position, equals(PlayerPosition.forward));
        expect(player.overallRating, equals(95));
      });
    });
  });
}
```

#### Property-Based Testing

```dart
import 'package:test/test.dart';
import 'package:soccer_engine/soccer_engine.dart';

void main() {
  group('Player property tests', () {
    test('rating is always within valid range', () {
      final random = Random(12345);
      
      for (int i = 0; i < 100; i++) {
        final rating = random.nextInt(100) + 1; // 1-100
        
        final player = Player(
          id: 'player_$i',
          name: 'Player $i',
          position: PlayerPosition.forward,
          overallRating: rating,
        );
        
        expect(player.overallRating, greaterThanOrEqualTo(1));
        expect(player.overallRating, lessThanOrEqualTo(100));
      }
    });
  });
}
```

### Running Tests

```bash
# Run all tests
dart test

# Run specific test file
dart test test/models/player_test.dart

# Run tests with coverage
dart test --coverage=coverage
dart run coverage:format_coverage --lcov --in=coverage --out=coverage/lcov.info --report-on=lib

# Run tests in watch mode (if using test_process)
dart test --reporter=expanded
```

---

## Documentation

### Code Documentation

Every public API must be documented:

```dart
/// Calculates the overall team rating based on player ratings and chemistry.
/// 
/// The calculation considers:
/// - Average player ratings weighted by position importance
/// - Team chemistry factor (0.9-1.1 multiplier)
/// - Formation suitability bonus
/// 
/// Returns a rating between 1 and 100, where:
/// - 90-100: World-class team
/// - 80-89: Elite team
/// - 70-79: Strong team
/// - 60-69: Average team
/// - Below 60: Weak team
/// 
/// Example:
/// ```dart
/// final team = Team(players: players, formation: Formation.f442);
/// final rating = team.calculateOverallRating();
/// print('Team rating: $rating');
/// ```
/// 
/// Throws [StateError] if the team has fewer than 11 players.
int calculateOverallRating() {
  // Implementation...
}
```

### README Updates

When adding significant features, update the main README:

- Add to feature list
- Update usage examples
- Include in API overview
- Add to roadmap if applicable

### API Documentation

Update relevant API documentation files:

- `documentation/api-reference/models.md`
- `documentation/api-reference/ai-systems.md`
- `documentation/api-reference/game-systems.md`

---

## Pull Request Process

### Before Creating a PR

1. **Ensure all tests pass**: `dart test`
2. **Check code formatting**: `dart format .`
3. **Run static analysis**: `dart analyze`
4. **Update documentation** if needed
5. **Rebase on latest main** if necessary

### PR Description Template

```markdown
## Description
Brief description of the changes and why they were made.

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [ ] Code refactoring
- [ ] Performance improvement

## Changes Made
- List of specific changes
- Another change
- Yet another change

## Testing
- [ ] Added unit tests for new functionality
- [ ] Added integration tests if applicable
- [ ] All existing tests pass
- [ ] Manual testing completed

## Documentation
- [ ] Updated code documentation
- [ ] Updated API documentation
- [ ] Updated README if necessary

## Screenshots (if applicable)
Include screenshots for UI changes or demo outputs.

## Checklist
- [ ] My code follows the project's style guidelines
- [ ] I have performed a self-review of my code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
```

### Review Process

1. **Automated Checks**: CI/CD pipeline runs tests and analysis
2. **Code Review**: At least one maintainer reviews the code
3. **Discussion**: Address any feedback or questions
4. **Approval**: Maintainer approves the changes
5. **Merge**: Changes are merged to main branch

---

## Issue Reporting

### Bug Reports

Use the bug report template:

```markdown
**Bug Description**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Create a team with '...'
2. Run simulation with '...'
3. Observe the error

**Expected Behavior**
A clear and concise description of what you expected to happen.

**Actual Behavior**
What actually happened instead.

**Code Example**
```dart
// Minimal code example that reproduces the issue
final team = Team(players: players);
final result = team.simulateMatch(); // Throws error here
```

**Environment**
- Dart version: [e.g., 3.0.0]
- Operating System: [e.g., Windows 11, macOS 13, Ubuntu 22.04]
- Project version: [e.g., 1.2.0]

**Additional Context**
Add any other context about the problem here.
```

### Feature Requests

Use the feature request template:

```markdown
**Feature Description**
A clear and concise description of the feature you'd like to see.

**Problem Statement**
What problem would this feature solve? What use case does it address?

**Proposed Solution**
Describe the solution you'd like to see implemented.

**Alternatives Considered**
Describe any alternative solutions or features you've considered.

**Additional Context**
Add any other context, examples, or screenshots about the feature request.

**Implementation Ideas**
If you have ideas about how this could be implemented, share them here.
```

---

## Code Review Guidelines

### For Contributors

When your code is being reviewed:

- **Be responsive** to feedback
- **Ask questions** if feedback is unclear
- **Explain your reasoning** for design decisions
- **Be open** to alternative approaches
- **Update your PR** promptly based on feedback

### For Reviewers

When reviewing code:

- **Be constructive** and helpful
- **Explain the reasoning** behind suggestions
- **Praise good practices** when you see them
- **Focus on the code**, not the person
- **Be specific** in your feedback

#### Review Checklist

**Functionality**
- [ ] Does the code do what it's supposed to do?
- [ ] Are edge cases handled properly?
- [ ] Is error handling appropriate?
- [ ] Are there any obvious bugs?

**Design**
- [ ] Is the code well-structured?
- [ ] Does it follow project patterns?
- [ ] Is it maintainable and extensible?
- [ ] Are abstractions appropriate?

**Testing**
- [ ] Are there sufficient tests?
- [ ] Do tests cover edge cases?
- [ ] Are tests well-written and clear?
- [ ] Is test coverage maintained?

**Performance**
- [ ] Are there any obvious performance issues?
- [ ] Is memory usage reasonable?
- [ ] Are algorithms efficient?
- [ ] Are there unnecessary computations?

**Documentation**
- [ ] Is the code self-documenting?
- [ ] Are complex sections explained?
- [ ] Is API documentation complete?
- [ ] Are examples provided where helpful?

---

## Getting Help

### Resources

- **Documentation**: Start with our comprehensive docs
- **Code Examples**: Check `bin/demo.dart` and `bin/ai_demo.dart`
- **Tests**: Look at existing tests for patterns and examples
- **Discussions**: Use GitHub Discussions for questions

### Communication Channels

- **GitHub Issues**: For bug reports and feature requests
- **GitHub Discussions**: For questions and general discussion
- **Pull Request Comments**: For code-specific discussions

### Mentoring

New contributors are welcome! Don't hesitate to:

- Ask questions in your pull requests
- Request guidance on implementation approaches
- Seek help with testing strategies
- Ask for code review feedback

---

## Recognition

We value all contributions to the project! Contributors will be:

- **Credited** in the project contributors list
- **Mentioned** in release notes for significant contributions
- **Thanked** publicly for their efforts

### Types of Recognition

- **First-time contributor**: Special mention in PR
- **Regular contributor**: Added to contributors list
- **Significant feature**: Highlighted in release notes
- **Long-term maintainer**: Added to core team

---

Thank you for contributing to the Soccer Engine project! Your efforts help make this project better for everyone in the soccer simulation community.

For questions about contributing, please open a GitHub Discussion or reach out in your pull request.
