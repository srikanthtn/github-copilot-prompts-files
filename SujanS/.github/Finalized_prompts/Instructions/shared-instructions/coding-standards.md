# Coding Standards – Global (Scala)

## Language
- Scala is the mandated language.
- Idiomatic Scala practices are required.

## Core Standards
- Immutability by default
- Explicit handling of Option, Either, Try
- No usage of null
- Clear separation of concerns

## Determinism
- No non-deterministic constructs
- No reliance on system time without abstraction
- No hidden side effects

## Prohibited
- Global mutable state
- Hidden control flow
- Silent error handling
