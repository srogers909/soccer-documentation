# Models API Reference

[← Back to Documentation Home](../README.md)

This document provides comprehensive API reference for all data models in the Soccer Engine.

## 📋 Overview

The Soccer Engine models represent core entities in soccer simulation:
- **Player**: Individual player with attributes and statistics
- **Team**: Squad management and team organization
- **League**: Competition management and tournament organization
- **Gameweek**: Match scheduling and round organization
- **Match**: Match data, events, and results
- **Financial Account**: Budget management and Financial Fair Play
- **Youth Academy**: Youth player development
- **Contracts & Transfers**: Player movement and negotiations

## 👤 Player

The core entity representing a soccer player.

### Constructor

```dart
Player({
  required String id,
  required String name,
  required int age,
  required PlayerPosition position,
  int? technical,
  int? physical,
  int? mental,
  int? form,
  int? fitness,
})
```

### Properties

| Property | Type | Description | Range |
|----------|------|-------------|-------|
| `id` | `String` | Unique player identifier | Non-empty |
| `name` | `String` | Player's full name | Non-empty |
| `age` | `int` | Player's age in years | 16-45 |
| `position` | `PlayerPosition` | Primary playing position | Enum value |
| `technical` | `int` | Technical skill rating | 1-100 (default: 50) |
| `physical` | `int` | Physical attributes rating | 1-100 (default: 50) |
| `mental` | `int` | Mental attributes rating | 1-100 (default: 50) |
| `form` | `int` | Current form rating | 1-10 (default: 7) |
| `fitness` | `int` | Current fitness percentage | 0-100 (default: 100) |

### Computed Properties

#### `overallRating` → `int`
Simple average of all attributes:
```dart
int get overallRating => ((technical + physical + mental) / 3).round();
```

#### `positionOverallRating` → `int`
Position-weighted overall rating:
- **Goalkeepers**: Mental (50%), Physical (30%), Technical (20%)
- **Defenders**: Physical (40%), Mental (35%), Technical (25%)
- **Midfielders**: Technical (40%), Mental (35%), Physical (25%)
- **Forwards**: Technical (45%), Physical (30%), Mental (25%)

### Methods

#### `updateForm(int newForm)` → `Player`
Updates player's form rating.
```dart
final player = Player(id: 'p1', name: 'Test', age: 25, position: PlayerPosition.midfielder);
final updatedPlayer = player.updateForm(9);
print(updatedPlayer.form); // 9
```

#### `updateFitness(int newFitness)` → `Player`
Updates player's fitness level.
```dart
final updatedPlayer = player.updateFitness(85);
print(updatedPlayer.fitness); // 85
```

### JSON Serialization

```dart
// To JSON
final json = player.toJson();

// From JSON
final player = Player.fromJson(json);
```

### Cross-References

- **Used in [Team](#-team)**: Players form team squads and starting lineups
- **Part of [Match Events](#match-events)**: Players are involved in match events (goals, cards, etc.)
- **Youth Development**: See [Youth Academy](#-youth-academy) for player development

### Example Usage

```dart
// Create a player
final player = Player(
  id: 'messi-1',
  name: 'Lionel Messi',
  age: 36,
  position: PlayerPosition.forward,
  technical: 95,
  physical: 75,
  mental: 98,
  form: 9,
  fitness: 90,
);

print('Overall: ${player.overallRating}'); // 89
print('Position Rating: ${player.positionOverallRating}'); // 91

// Update player condition
final updatedPlayer = player
    .updateForm(8)
    .updateFitness(95);
```

## 🏆 League

Represents a football league with teams, rules, and metadata for organizing competitions.

### Constructor

```dart
League({
  required String id,
  required String name,
  required String country,
  LeagueTier tier = LeagueTier.tier1,
  LeagueFormat format = LeagueFormat.roundRobin,
  List<Team>? teams,
  LeagueRules? rules,
  int? foundedYear,
  int maxTeams = 20,
  int minTeams = 8,
})
```

### Properties

| Property | Type | Description | Default/Range |
|----------|------|-------------|---------------|
| `id` | `String` | Unique league identifier | Non-empty |
| `name` | `String` | League name | Non-empty |
| `country` | `String` | Country where league is based | Non-empty |
| `tier` | `LeagueTier` | League tier/division level | `tier1` |
| `format` | `LeagueFormat` | League format | `roundRobin` |
| `teams` | `List<Team>` | Participating teams | Empty list |
| `rules` | `LeagueRules` | League rules and scoring | Default rules |
| `foundedYear` | `int` | Year league was founded | Current year (1850-current) |
| `maxTeams` | `int` | Maximum teams allowed | 20 |
| `minTeams` | `int` | Minimum teams for competition | 8 |

### Enums

#### `LeagueFormat`
```dart
enum LeagueFormat {
  roundRobin,         // Each team plays every other team twice
  singleRoundRobin,   // Each team plays every other team once
  playoff,            // Elimination rounds
  groupAndKnockout,   // Group stage + knockout rounds
}
```

#### `LeagueTier`
```dart
enum LeagueTier {
  tier1,      // Top tier (Premier League, La Liga)
  tier2,      // Second tier (Championship, La Liga 2)
  tier3,      // Third tier
  tier4,      // Fourth tier
  tier5Plus,  // Fifth tier and below
}

// Each tier has a display name
String get displayName; // "1st Tier", "2nd Tier", etc.
```

### League Rules

```dart
class LeagueRules {
  final int promotionSpots;    // Teams promoted (default: 2)
  final int relegationSpots;   // Teams relegated (default: 3)
  final int playoffSpots;      // Teams in playoffs (default: 4)
  final int pointsForWin;      // Points for win (default: 3)
  final int pointsForDraw;     // Points for draw (default: 1)
  final int pointsForLoss;     // Points for loss (default: 0)
  final bool useGoalDifference; // Use goal difference for tiebreaking (default: true)
  final bool useHeadToHead;    // Use head-to-head for tiebreaking (default: false)
}
```

### Computed Properties

#### `isCompetitive` → `bool`
Whether league has minimum teams for competitive play:
```dart
bool get isCompetitive => teams.length >= minTeams;
```

#### `canStartSeason` → `bool`
Whether league can start with fixture generation:
```dart
bool get canStartSeason {
  if (!isCompetitive) return false;
  if ((format == LeagueFormat.roundRobin || format == LeagueFormat.singleRoundRobin) 
      && teams.length % 2 != 0) return false;
  return true;
}
```

#### `requiredGameweeks` → `int`
Number of gameweeks needed for the season:
- **Round-robin**: `(teams.length - 1) * 2`
- **Single round-robin**: `teams.length - 1`
- **Other formats**: Simplified calculation

#### `statistics` → `Map<String, dynamic>`
League statistics including team count, player totals, averages, and status.

### Methods

#### `addTeam(Team team)` → `League`
Adds a team to the league.
```dart
final league = League(id: 'pl', name: 'Premier League', country: 'England');
final updatedLeague = league.addTeam(manchesterUnited);
```

**Validation**:
- Team not already in league
- League not at maximum capacity
- Even number of teams for round-robin format

**Throws**: `ArgumentError` for validation failures.

#### `removeTeam(String teamId)` → `League`
Removes a team from the league.
```dart
final updatedLeague = league.removeTeam('man-utd');
```

**Validation**: Won't fall below minimum team count.

#### `updateRules(LeagueRules newRules)` → `League`
Updates league rules.
```dart
final newRules = LeagueRules(promotionSpots: 3, relegationSpots: 2);
final updatedLeague = league.updateRules(newRules);
```

#### `getTeam(String teamId)` → `Team?`
Gets a team by ID, returns `null` if not found.
```dart
final team = league.getTeam('liverpool');
```

### Validation Rules

- **ID, name, country**: Non-empty strings
- **Founded year**: 1850 to current year
- **Team count**: 0 to `maxTeams`, minimum `minTeams` for competitive play
- **Round-robin format**: Requires even number of teams
- **No duplicate teams**: Each team can only appear once

### Cross-References

- **Contains [Team](#-team)**: Leagues organize teams into competitions
- **Uses [Gameweek](#-gameweek)**: Season is divided into gameweeks
- **Generates [Match](#-match)**: Fixture generation creates matches between teams

### Example Usage

```dart
// Create a league
final premierLeague = League(
  id: 'premier-league',
  name: 'Premier League',
  country: 'England',
  tier: LeagueTier.tier1,
  format: LeagueFormat.roundRobin,
  maxTeams: 20,
  minTeams: 16,
);

// Add teams
final updatedLeague = premierLeague
    .addTeam(manchesterUnited)
    .addTeam(liverpool)
    .addTeam(chelsea);

// Check league status
print('Competitive: ${updatedLeague.isCompetitive}');
print('Can start: ${updatedLeague.canStartSeason}');
print('Gameweeks needed: ${updatedLeague.requiredGameweeks}');

// Custom rules
final customRules = LeagueRules(
  promotionSpots: 3,
  relegationSpots: 2,
  pointsForWin: 3,
  useGoalDifference: true,
);
final leagueWithRules = updatedLeague.updateRules(customRules);
```

## 📅 Gameweek

Represents a gameweek/matchday in a league season, organizing matches into rounds.

### Constructor

```dart
Gameweek({
  required String id,
  required int number,
  required String seasonId,
  required DateTime scheduledDate,
  List<Match>? matches,
  GameweekStatus status = GameweekStatus.scheduled,
  String? name,
  bool isSpecial = false,
})
```

### Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `id` | `String` | Unique gameweek identifier | Required |
| `number` | `int` | Gameweek number in season (1-based) | Required |
| `seasonId` | `String` | League season identifier | Required |
| `scheduledDate` | `DateTime` | Scheduled date for gameweek | Required |
| `matches` | `List<Match>` | Matches in this gameweek | Empty list |
| `status` | `GameweekStatus` | Current status | `scheduled` |
| `name` | `String?` | Optional name/description | `null` |
| `isSpecial` | `bool` | Whether this is a special gameweek | `false` |

### Gameweek Status

```dart
enum GameweekStatus {
  scheduled,   // Scheduled but not started
  inProgress,  // Currently in progress
  completed,   // Completed
  postponed,   // Postponed
  cancelled,   // Cancelled
}
```

### Computed Properties

#### Match Status Counts
```dart
int get completedMatchesCount;   // Number of completed matches
int get scheduledMatchesCount;   // Number of scheduled matches  
int get inProgressMatchesCount;  // Number of in-progress matches
```

#### Gameweek Status
```dart
bool get isCompleted;  // All matches completed
bool get hasStarted;   // Any matches started
bool get canStart;     // Can be started (valid matches, no conflicts)
```

#### Match Timing
```dart
DateTime? get earliestMatchDate;  // Earliest match kickoff
DateTime? get latestMatchDate;    // Latest match kickoff
```

#### Team Information
```dart
Set<String> get participatingTeams;  // All team IDs in gameweek
```

#### Display
```dart
String get displayName;  // Name or "Gameweek X"
```

#### Statistics
```dart
Map<String, dynamic> get statistics;  // Match counts, goals, etc.
```

### Methods

#### `addMatch(Match match)` → `Gameweek`
Adds a match to the gameweek.
```dart
final gameweek = Gameweek(
  id: 'gw1',
  number: 1,
  seasonId: 'season-2024',
  scheduledDate: DateTime.now(),
);
final updatedGameweek = gameweek.addMatch(match);
```

**Validation**: Prevents team conflicts (same team in multiple matches).

#### `removeMatch(String matchId)` → `Gameweek`
Removes a match from the gameweek.
```dart
final updatedGameweek = gameweek.removeMatch('match-1');
```

#### `updateStatus(GameweekStatus newStatus)` → `Gameweek`
Updates the gameweek status.
```dart
final completedGameweek = gameweek.updateStatus(GameweekStatus.completed);
```

#### `updateScheduledDate(DateTime newDate)` → `Gameweek`
Updates the scheduled date.
```dart
final rescheduledGameweek = gameweek.updateScheduledDate(DateTime.parse('2024-01-15'));
```

#### `getMatchesForTeam(String teamId)` → `List<Match>`
Gets all matches for a specific team.
```dart
final teamMatches = gameweek.getMatchesForTeam('liverpool');
```

#### `getMatchBetweenTeams(String team1Id, String team2Id)` → `Match?`
Gets the match between two teams, returns `null` if not found.
```dart
final match = gameweek.getMatchBetweenTeams('liverpool', 'manchester-united');
```

### Validation Rules

- **ID and seasonId**: Non-empty strings
- **Number**: Must be positive (≥ 1)
- **Team conflicts**: Each team can only appear in one match per gameweek
- **Match validation**: All matches must be valid with proper team assignments

### Example Usage

```dart
// Create a gameweek
final gameweek = Gameweek(
  id: 'gw1-2024',
  number: 1,
  seasonId: 'premier-league-2024',
  scheduledDate: DateTime.parse('2024-08-17'),
  name: 'Opening Weekend',
);

// Add matches
final match1 = Match.create(
  id: 'match-1',
  homeTeam: manchesterUnited,
  awayTeam: liverpool,
  kickoffTime: DateTime.parse('2024-08-17T15:00:00'),
);

final updatedGameweek = gameweek.addMatch(match1);

// Check status
print('Can start: ${updatedGameweek.canStart}');
print('Teams: ${updatedGameweek.participatingTeams.length}');
print('Display: ${updatedGameweek.displayName}');

// Update status as matches progress
final inProgressGameweek = updatedGameweek.updateStatus(GameweekStatus.inProgress);

// Get statistics
final stats = updatedGameweek.statistics;
print('Total goals: ${stats['totalGoals']}');
print('Completed matches: ${stats['completedMatches']}');
```

## 🏟️ Team

Represents a soccer team with squad management capabilities.

### Constructor

```dart
Team({
  required String id,
  required String name,
  required String city,
  required int foundedYear,
  Stadium? stadium,
  List<Player>? players,
  Formation? formation,
  List<Player>? startingXI,
  int? morale,
})
```

### Properties

| Property | Type | Description | Constraints |
|----------|------|-------------|-------------|
| `id` | `String` | Unique team identifier | Non-empty |
| `name` | `String` | Team name | Non-empty |
| `city` | `String` | Team's home city | Non-empty |
| `foundedYear` | `int` | Year team was founded | 1850-current year |
| `stadium` | `Stadium` | Home stadium | Auto-generated if null |
| `players` | `List<Player>` | Squad players | Max 30 players |
| `formation` | `Formation` | Current formation | Default: 4-4-2 |
| `startingXI` | `List<Player>` | Starting lineup | Exactly 11 players |
| `morale` | `int` | Team morale | 0-100 (default: 75) |

### Computed Properties

#### `overallRating` → `int`
Average rating of all squad players.

#### `positionStrengths` → `Map<PlayerPosition, int>`
Average rating by position:
```dart
final strengths = team.positionStrengths;
print('Midfielder strength: ${strengths[PlayerPosition.midfielder]}');
```

#### `chemistry` → `int`
Team chemistry calculation (0-100) based on:
- Team morale (base)
- Squad balance bonus
- Squad size considerations

#### `isCompetitive` → `bool`
Whether team meets minimum squad requirements (16+ players).

### Methods

#### `addPlayer(Player player)` → `Team`
Adds a player to the squad.
```dart
final newPlayer = Player(id: 'new1', name: 'New Player', age: 22, position: PlayerPosition.midfielder);
final updatedTeam = team.addPlayer(newPlayer);
```

**Throws**: `ArgumentError` if player already exists or squad is full.

#### `removePlayer(String playerId)` → `Team`
Removes a player from squad and starting XI.
```dart
final updatedTeam = team.removePlayer('player-id');
```

#### `setFormation(Formation newFormation)` → `Team`
Changes team formation.
```dart
final updatedTeam = team.setFormation(Formation.f433);
```

#### `setStartingXI(List<Player> newStartingXI)` → `Team`
Sets the starting lineup.
```dart
final starters = team.players.take(11).toList();
final updatedTeam = team.setStartingXI(starters);
```

**Throws**: `ArgumentError` if lineup is invalid.

#### `updateMorale(int newMorale)` → `Team`
Updates team morale.
```dart
final updatedTeam = team.updateMorale(85);
```

#### `getPlayersByPosition(PlayerPosition position)` → `List<Player>`
Gets all players in a specific position.
```dart
final midfielders = team.getPlayersByPosition(PlayerPosition.midfielder);
```

### Formation Enum

```dart
enum Formation {
  f442,   // 4-4-2
  f433,   // 4-3-3
  f352,   // 3-5-2
  f541,   // 5-4-1
  f343,   // 3-4-3
  f532,   // 5-3-2
  f4231,  // 4-2-3-1
  f4141,  // 4-1-4-1
  f451,   // 4-5-1
  f3421,  // 3-4-2-1
}
```

Each formation has:
- `requirements` → `List<int>`: Player requirements [GK, DEF, MID, FWD]
- `displayName` → `String`: Human-readable name

### Example Usage

```dart
// Create a team
final team = Team(
  id: 'barcelona',
  name: 'FC Barcelona',
  city: 'Barcelona',
  foundedYear: 1899,
  formation: Formation.f433,
);

// Add players
final updatedTeam = team
    .addPlayer(goalkeeper)
    .addPlayer(defender)
    .addPlayer(midfielder);

// Check team status
print('Chemistry: ${updatedTeam.chemistry}');
print('Competitive: ${updatedTeam.isCompetitive}');
print('Overall: ${updatedTeam.overallRating}');
```

## ⚽ Match

Represents a soccer match with weather conditions, events, and match progression.

### Constructor

```dart
Match({
  required String id,
  required Team homeTeam,
  required Team awayTeam,
  required Weather weather,
  required DateTime kickoffTime,
  bool isNeutralVenue = false,
  bool isCompleted = false,
  int homeGoals = 0,
  int awayGoals = 0,
  MatchResult? result,
  int currentMinute = 0,
  List<MatchEvent> events = const [],
})
```

### Factory Constructor

```dart
Match.create({
  required String id,
  required Team homeTeam,
  required Team awayTeam,
  required Weather weather,
  required DateTime kickoffTime,
  bool isNeutralVenue = false,
})
```

### Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `id` | `String` | Unique match identifier | Required |
| `homeTeam` | `Team` | Home team object | Required |
| `awayTeam` | `Team` | Away team object | Required |
| `weather` | `Weather` | Weather conditions | Required |
| `kickoffTime` | `DateTime` | Match start time | Required |
| `isNeutralVenue` | `bool` | Whether played at neutral venue | `false` |
| `isCompleted` | `bool` | Whether match is finished | `false` |
| `homeGoals` | `int` | Home team goals | `0` |
| `awayGoals` | `int` | Away team goals | `0` |
| `result` | `MatchResult?` | Match outcome | `null` |
| `currentMinute` | `int` | Current match minute | `0` |
| `events` | `List<MatchEvent>` | Match events | Empty list |

### Weather System

```dart
class Weather {
  final WeatherCondition condition;
  final double temperature;  // Celsius (-40 to 50)
  final double humidity;     // Percentage (0-100)
  final double windSpeed;    // km/h (0-200)
  
  // Performance impact factor (0.8 to 1.2)
  double get performanceImpact;
}

enum WeatherCondition {
  sunny,    // +0.05 performance
  cloudy,   // No change
  rainy,    // -0.15 performance
  snowy,    // -0.20 performance
  windy,    // -0.10 performance
  foggy,    // -0.10 performance
}
```

### Match Results

```dart
enum MatchResult {
  homeWin,  // Home team victory
  draw,     // Tie game
  awayWin,  // Away team victory
}
```

### Match Events

```dart
enum MatchEventType {
  goal,         // Goal scored
  yellowCard,   // Yellow card issued
  redCard,      // Red card issued
  substitution, // Player substitution
  kickoff,      // Match start
  halfTime,     // Half-time break
  fullTime,     // Match end
  penalty,      // Penalty awarded
  ownGoal,      // Own goal
  assist,       // Goal assist
}

class MatchEvent {
  final String id;
  final MatchEventType type;
  final int minute;           // 0-120 minutes
  final String? playerId;
  final String? playerName;
  final String teamId;
  final String description;
  final Map<String, dynamic> metadata;
}
```

### Computed Properties

#### `homeAdvantage` → `double`
Home advantage factor (1.0 to 1.15) based on:
- **Neutral venue**: 1.0 (no advantage)
- **Stadium capacity**:
  - 80,000+: +0.15
  - 60,000+: +0.12
  - 40,000+: +0.10
  - 20,000+: +0.08
  - Under 20,000: +0.05

### Methods

#### `copyWith({...})` → `Match`
Creates updated match with new state.
```dart
final updatedMatch = match.copyWith(
  homeGoals: 2,
  awayGoals: 1,
  currentMinute: 90,
  isCompleted: true,
  result: MatchResult.homeWin,
);
```

### Validation Rules

- **ID**: Non-empty string
- **Teams**: Must be different teams
- **Kickoff time**: Cannot be more than 1 day in the past
- **Event minutes**: 0-120 range
- **Weather**: Temperature (-40 to 50°C), humidity (0-100%), wind speed (0-200 km/h)

### Cross-References

- **Related to [Team](#-team)**: Each match involves two teams
- **Used in [Gameweek](#-gameweek)**: Matches are organized into gameweeks
- **Part of [League](#-league)**: Matches contribute to league standings

### Example Usage

```dart
// Create weather conditions
final weather = Weather.create(
  condition: WeatherCondition.sunny,
  temperature: 22.0,
  humidity: 60.0,
  windSpeed: 10.0,
);

// Create a match
final match = Match.create(
  id: 'el-clasico-2024',
  homeTeam: barcelona,
  awayTeam: realMadrid,
  weather: weather,
  kickoffTime: DateTime.parse('2024-04-21T20:00:00'),
);

// Check conditions
print('Home advantage: ${match.homeAdvantage}'); // 1.12 (large stadium)
print('Weather impact: ${weather.performanceImpact}'); // 1.05 (sunny)

// Create match event
final goalEvent = MatchEvent.create(
  id: 'goal-1',
  type: MatchEventType.goal,
  minute: 25,
  teamId: barcelona.id,
  playerId: 'messi-1',
  playerName: 'Lionel Messi',
  description: 'Goal by Lionel Messi (assist: Xavi)',
  metadata: {'assistId': 'xavi-1'},
);

// Update match state
final updatedMatch = match.copyWith(
  homeGoals: 1,
  currentMinute: 25,
  events: [goalEvent],
);
```

## 💰 Financial Account

Manages team finances and Financial Fair Play compliance.

### Constructor

```dart
FinancialAccount({
  required String teamId,
  required int balance,
  required String currency,
  Map<BudgetCategory, int>? budgetLimits,
  List<FinancialTransaction>? transactions,
})
```

### Budget Categories

```dart
enum BudgetCategory {
  transfers,
  wages,
  youth,
  facilities,
  marketing,
  operations,
}
```

### Methods

#### `getAvailableBudget(BudgetCategory category)` → `int`
Gets remaining budget for a category.

#### `addTransaction(FinancialTransaction transaction)` → `FinancialAccount`
Records a financial transaction.

#### `checkFFPCompliance(DateTime startDate, DateTime endDate)` → `FFPComplianceResult`
Checks Financial Fair Play compliance over a period.

### Example Usage

```dart
final account = FinancialAccount(
  teamId: 'barcelona',
  balance: 100000000, // €100M
  currency: 'EUR',
  budgetLimits: {
    BudgetCategory.transfers: 50000000, // €50M
    BudgetCategory.wages: 200000000,    // €200M
  },
);

final transferBudget = account.getAvailableBudget(BudgetCategory.transfers);
print('Transfer budget: €${transferBudget ~/ 1000000}M');
```

## 🌱 Youth Academy

Manages youth player development and progression.

### Constructor

```dart
YouthAcademy({
  required String id,
  required String teamId,
  required String name,
  List<YouthPlayer>? players,
  Map<String, int>? facilities,
  List<YouthCoach>? coaches,
  int? reputation,
})
```

### Youth Player Development

```dart
class YouthPlayer extends Player {
  final int potential;           // Max possible rating
  final double developmentRate;  // Growth speed
  final List<String> traits;    // Special characteristics
  
  // Development methods
  YouthPlayer developSkills(int months);
  bool get readyForFirstTeam;
}
```

### Example Usage

```dart
final academy = YouthAcademy(
  id: 'barca-academy',
  teamId: 'barcelona',
  name: 'La Masia',
  reputation: 95,
);

// Develop youth players
final developedAcademy = academy.developPlayers(months: 6);

// Check for first-team ready players
final readyPlayers = academy.players
    .where((p) => p.readyForFirstTeam)
    .toList();
```

## 📄 Contracts & Transfers

### Contract

```dart
class Contract {
  final String id;
  final String playerId;
  final String teamId;
  final DateTime startDate;
  final DateTime endDate;
  final int weeklyWage;
  final Map<String, dynamic> clauses;
  
  bool get isActive;
  int get daysRemaining;
  int get totalValue;
}
```

### Transfer

```dart
class Transfer {
  final String id;
  final String playerId;
  final String fromTeamId;
  final String toTeamId;
  final int fee;
  final DateTime agreedDate;
  final DateTime? completedDate;
  final TransferStatus status;
}

enum TransferStatus {
  agreed,
  pending,
  completed,
  cancelled,
}
```

## 🔧 Validation Rules

All models include comprehensive validation:

### Player Validation
- Age: 16-45 years
- Attributes: 1-100 range
- Form: 1-10 range
- Fitness: 0-100 range
- ID and name: Non-empty strings

### Team Validation
- Squad size: 0-30 players
- Starting XI: Exactly 11 unique players from squad
- Founded year: 1850-current year
- Morale: 0-100 range

### Financial Validation
- Positive balance for transactions
- Budget limits must be positive
- FFP compliance calculations

## 📚 Next Steps

- **AI Systems**: Explore [AI Systems API](ai-systems.md)
- **Game Systems**: Learn about [Game Systems API](game-systems.md)
- **Technical Details**: Dive into [System Architecture](../technical/architecture.md)

---

**Navigation**: [Documentation Home](../README.md) → API Reference → Models
