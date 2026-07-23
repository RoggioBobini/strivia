# Trivia Game Specification

## Version
- Specification version: v1.0
- Status: implementation-ready MVP
- Date: 2026-07-23

## 1. Goal
Build a customizable local multiplayer trivia game that supports:
- 1 to n players
- configurable question prompts with 0 to n suggested answers
- fixed turn-based progression
- deterministic scoring and streak behavior
- a simple bonus system driven by streaks
- a configurable victory threshold

## 2. Scope
### 2.1 In scope for this version
- Local multiplayer gameplay only
- A single active question at a time
- Fixed round-robin turn order
- Question answer handling for both multiple-choice and freeform questions
- Scoring, streaks, and activation of a small bonus set
- A configurable win threshold

### 2.2 Out of scope for this version
- Networked multiplayer
- Persistent game state
- Ranking or matchmaking
- Difficulty progression or adaptive question selection
- Complex bonus stacking or economy systems

## 3. Core Gameplay Rules
### 3.1 Player and session setup
- A session is created with a list of players.
- Each player must have a unique identifier and display name.
- The starting score for all players is 0.
- The starting streak for all players is 0.
- The active player is selected from the player list using round-robin order.

### 3.2 Turn order
- Turn order is fixed and deterministic.
- The first player in the list becomes the active player at the start of the game.
- After each question, the next player in order becomes active when the current player answers incorrectly.
- If the current player answers correctly, they remain active and receive the next question.

### 3.3 Question flow
1. Select the active player.
2. Present the question prompt.
3. If the question has suggested answers, present them in the configured order.
4. Collect the player's answer.
5. Evaluate the answer against the question's accepted answers.
6. Apply scoring and streak updates.
7. Apply bonus resolution.
8. Advance the turn or keep the same player depending on the result.

### 3.4 Answer modes
A question may be one of two types:
- multiple-choice: the question has one or more suggested answers and one correct answer
- freeform: the question has zero suggested answers and relies on accepted answer strings

### 3.5 Correct answer behavior
If the player's answer matches a correct answer:
- add 10 points to the player's score
- increment the player's streak by 1
- keep the current player active for the next question

### 3.6 Wrong answer behavior
If the player's answer does not match a correct answer:
- subtract 5 points from the player's score
- reset the player's streak to 0
- pass the turn to the next player in order

### 3.7 Streak formula
The game uses a simple, deterministic streak score rule:
- base correct score = +10
- wrong answer penalty = -5
- cumulative streak bonus = +2 per consecutive correct answer, capped at +10

The effective score for a correct answer is therefore:
- score_delta = 10 + min(10, 2 x (current_streak_after_answer - 1))

Example:
- streak 0 → correct answer gives +10
- streak 1 → correct answer gives +12
- streak 2 → correct answer gives +14
- streak 3 → correct answer gives +16
- streak 4 or more → correct answer gives +18, capped by the maximum bonus policy

Implementation note: the streak bonus is evaluated after the streak is incremented for the current answer.

## 4. Scoring Rules
### 4.1 Base rules
- Correct answer: score +10, or +10 plus streak bonus
- Wrong answer: score -5
- Score is never below a configured minimum; the initial minimum is 0

### 4.2 Streak behavior
- A streak is reset to 0 on any wrong answer.
- A streak is not reset by a skipped or invalid answer.
- Invalid answers are treated as wrong answers for scoring purposes.

## 5. Answer Validation Rules
### 5.1 Multiple-choice validation
For multiple-choice questions:
- the player must select one suggested answer or submit an equivalent string input
- an answer is correct only if the normalized submitted value equals the normalized correct answer
- the question's suggested answers must be unique

### 5.2 Freeform validation
For freeform questions:
- the correct answer is stored as one canonical answer string
- the question may also define an array of accepted aliases
- all answer strings are normalized before comparison

### 5.3 Normalization rules
Answer normalization must follow these rules:
- trim leading and trailing whitespace
- collapse repeated internal whitespace to a single space
- convert to lowercase
- compare using the normalized form

Example:
- "  Paris " → "paris"
- "New   York" → "new york"

## 6. Bonus System
### 6.1 Bonus model
Each bonus feature has:
- a unique identifier
- a name
- a target: self or opponent
- an activation condition
- an effect description
- a duration or scope
- a usage limit, if any

### 6.2 Initial bonus set
The initial version supports two bonus features:

1. Shield
- unlock condition: streak reaches 3
- target: self
- effect: when the player is the active player and answers incorrectly, the penalty is reduced from -5 to 0 once per activation
- duration: one use only, consumed when it blocks the next wrong answer
- usage limit: 1 per unlock cycle

2. Sabotage
- unlock condition: streak reaches 5
- target: opponent
- effect: on the opponent's next wrong answer, the penalty is doubled from -5 to -10
- duration: immediate, single-use effect
- usage limit: 1 per unlock cycle

### 6.3 Bonus resolution order
When a player answers a question:
1. apply the answer result
2. evaluate any active bonus effects that trigger on the outcome
3. resolve the bonus effect before finalizing score state
4. expire any consumed temporary bonuses

## 7. Game Session Data Model
### 7.1 Player
A player has:
- id: string
- name: string
- score: integer
- streak: integer
- unlockedBonuses: list of bonus IDs
- activeBonuses: list of currently active bonus instances

### 7.2 Question
A question has:
- id: string
- prompt: string
- suggestedAnswers: array of strings
- correctAnswer: string
- acceptedAnswers: array of strings
- metadata: optional category/difficulty fields

### 7.3 Game Session
A game session has:
- id: string
- players: array of Player
- turnOrder: array of player ids
- activePlayerId: string
- questionPool: array of Question
- currentQuestionId: string or null
- winThreshold: integer
- bonusRules: configured bonus definitions

## 8. Configuration Parameters
The game must support the following initial configuration inputs:
- number of players
- player names
- win threshold
- number of suggested answers displayed for a question
- scoring model
- bonus feature set and activation rules
- question source or pool

## 9. Winning Condition
A player wins when their score reaches or exceeds the configured win threshold.

### 9.1 Tie behavior
- If multiple players reach the win threshold in the same round, the player with the higher score wins.
- If scores are tied, the game ends in a draw.

## 10. Failure and Edge Cases
The implementation must define the following behavior:
- empty or missing player list: reject session creation
- empty question pool: reject session creation or end the game with a no-questions state
- invalid answer input: treat as incorrect answer
- blank answer for multiple-choice question: treat as incorrect answer
- duplicate player ids or duplicate names: reject session creation
- question with no correct answer: reject question creation

## 11. Acceptance Criteria
The implementation is considered complete for the MVP when all of the following are true:
1. A session can be created for 1 to n players.
2. The turn order follows a deterministic round-robin sequence.
3. Correct answers increase score and keep the active player in control.
4. Wrong answers decrease score and pass control to the next player.
5. Streaks increment only on correct answers and reset on wrong answers.
6. A streak of 3 unlocks the Shield bonus and a streak of 5 unlocks Sabotage.
7. The win threshold ends the game when a player reaches or exceeds it.
8. Answer normalization and validation behave consistently for both multiple-choice and freeform questions.

## 12. Implementation Notes
The initial implementation should:
- keep game rules separate from UI rendering
- use a simple data model that can be expanded later
- make question evaluation deterministic
- keep bonus logic explicit and isolated from scoring logic
- support future expansion for richer scoring and additional bonus types
