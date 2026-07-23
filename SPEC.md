# Trivia Game Specification

## Version
- Initial draft v0.1
- Date: 2026-07-20

## 1. Overview
This document defines the initial specification for a customizable multiplayer trivia game. It serves as a foundation for implementation and can be extended as the game design evolves.

## 2. Goal
Create a trivia game that supports:
- multiplayer gameplay from 1 to n players
- configurable question presentation with a variable number of suggested answers
- turn-based progression with scoring and bonus mechanics
- a configurable win condition based on points

## 3. Core Assumptions
The following assumptions define the initial version of the game:
- The game is played by 1 to n players.
- Each player has a score that changes over the course of the game.
- A question may be presented with 0 to n suggested answers.
- A value of 0 suggested answers means the answer is open-ended or freeform.
- The game is turn-based, where one player is selected to answer the current question.
- The scoring algorithm is not defined yet and will be specified later.

## 4. Core Gameplay Rules
### 4.1 Question Flow
1. The game selects a current player.
2. The current player receives a question.
3. The question may include a configurable number of suggested answers.
4. The player attempts to provide the correct answer.

### 4.2 Correct Answer
- If the player gives the correct answer:
  - the player receives points
  - the player remains in control of the next question
  - the player’s correct-answer streak increases

### 4.3 Wrong Answer
- If the player gives an incorrect answer:
  - the player loses points
  - the turn passes to another player
  - the next question is assigned to the next player

### 4.4 Streak-Based Bonus Features
- When a player achieves a streak of consecutive correct answers, they may unlock a bonus feature.
- A bonus feature can be applied either:
  - to the player themselves, or
  - to an opponent
- Example bonus effects include:
  - protecting the player from point loss on an invalid answer
  - causing the opponent to lose double points on their next incorrect answer

### 4.5 Winning Condition
- A player wins when their score reaches or exceeds a configurable threshold.

## 5. Game Entities
### 5.1 Player
A player has:
- a unique identifier
- a name or display name
- a current score
- a current correct-answer streak
- a set of available bonus features or unlocked bonuses

### 5.2 Question
A question has:
- a prompt or text
- zero or more suggested answers
- one correct answer
- optional metadata such as difficulty, category, or difficulty level

### 5.3 Game Session
A game session contains:
- the list of players
- the current turn order
- the current active player
- the current question
- the current score state
- the win threshold
- the active or available bonus system

### 5.4 Bonus Feature
A bonus feature has:
- a name
- a target (self or opponent)
- an effect description
- an activation condition
- a duration or scope, if applicable

## 6. Configurable Parameters
The game should support configuration for the following initial parameters:
- number of players
- win threshold for victory
- number of suggested answers displayed per question
- scoring model
- bonus feature set and activation rules
- question source or pool

## 7. Initial Rules Summary
The game behavior in this version is defined as follows:
- A player answers a question.
- Correct answer: gain points and keep the turn.
- Wrong answer: lose points and pass the turn.
- Consecutive correct answers build a streak.
- A streak unlocks a bonus feature that can influence gameplay.
- The game ends when a player reaches the configured points threshold.

## 8. Open Questions / Future Extensions
The following items are intentionally left undefined for the initial version:
- the exact scoring formula
- the exact bonus feature mechanics and limits
- the turn order algorithm for players beyond the first
- the handling of ties or multiple winners
- the question difficulty and progression system
- whether bonus features are limited by number of uses

## 9. Implementation Notes
The initial implementation should prioritize:
- a clear separation between game rules and presentation
- a simple, extensible data model for players, questions, and bonus features
- configuration-driven behavior where possible
- future support for richer scoring and bonus systems
