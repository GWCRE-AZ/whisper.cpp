# V1 Voice-to-Text App Plan (iPhone + Mac)

## Goal
Deliver a practical version 1 that lets a user speak naturally and insert accurate text with optional cleanup (filler-word removal and grammar polishing).

## Platforms
- iPhone 16 (iOS keyboard extension)
- MacBook Air (macOS menu bar app + global hotkey)

## V1 Scope

### In scope
- **Mac app**
  - Global hotkey to start/stop dictation
  - Local transcription using `whisper.cpp`
  - Insert text into focused app
  - Modes:
    - Verbatim
    - Clean (remove fillers + punctuation)
    - Polish (light grammar rewrite)
- **iPhone app + keyboard extension**
  - Keyboard mic button for dictation in text fields
  - Basic on-device transcription path
  - Same 3 output modes
- Shared settings sync (mode, custom replacements)

### Out of scope for V1
- Full conversational AI assistant behavior
- Multi-speaker diarization
- Advanced domain adaptation training
- Enterprise policy/compliance console

## Functional Requirements

### FR-1 Capture and transcribe speech
- User can start dictation in <= 1 action.
- Partial text updates every 300-800 ms when feasible.
- Final text produced within 1.5 s after stop for short utterances.

### FR-2 Insert text into active app
- macOS: insert at current cursor location.
- iOS: insert through keyboard extension into focused field.

### FR-3 Cleanup pipeline
- Remove common fillers: `um`, `uh`, `erm`, repeated ellipses.
- Normalize punctuation and capitalization.
- Preserve intent; never fabricate facts.

### FR-4 Modes
- Verbatim: minimal transforms.
- Clean: filler stripping + punctuation fixes.
- Polish: grammar and clarity rewrite while preserving meaning.

### FR-5 User controls
- Toggle mode quickly from menu bar (macOS) and keyboard toolbar (iOS).
- Undo last insertion.
- Custom replacement dictionary (e.g., names, product terms).

## Non-Functional Requirements
- Privacy-first default: local inference where possible.
- Offline capable on Mac for core dictation.
- P50 transcription latency under 2 s for <= 15 s clips on target hardware.
- Crash-free rate target >= 99.5% weekly sessions.

## High-Level Architecture

### Shared core
- Audio pre-processing (VAD + chunking)
- Whisper inference wrapper
- Text post-processing pipeline
  - deterministic cleanup rules
  - optional polish step

### macOS app
- Menu bar UI
- Global shortcut handler
- Accessibility-based text insertion

### iOS app
- Host app for onboarding/settings
- Keyboard extension for in-field dictation
- Shared app group storage for settings

## Post-Processing Rules (V1)
1. Remove standalone fillers bounded by punctuation/space.
2. Collapse repeated spaces and repeated punctuation.
3. Sentence-case first token and `i` -> `I` in obvious contexts.
4. Keep domain keywords from custom dictionary unchanged.

## Milestones

### M1: Core transcription prototype (Week 1)
- Mac CLI demo: record -> transcribe -> print
- Rule-based cleanup implemented

### M2: macOS app alpha (Week 2)
- Menu bar + hotkey + text insertion
- Mode toggle + undo insertion

### M3: iOS keyboard alpha (Week 3)
- Keyboard dictation button + insertion
- Shared settings and mode parity

### M4: Hardening and beta (Week 4)
- Latency tuning
- Crash fixes
- Basic telemetry (opt-in)

## Acceptance Criteria for V1
- User can dictate into Messages, Mail, and Notes on Mac via global hotkey.
- User can dictate into common text inputs on iPhone via custom keyboard.
- Clean mode removes obvious fillers in >= 90% of sampled cases.
- Polish mode improves readability without changing intent in manual QA set.

## Suggested Initial Task Breakdown
1. Build shared C++ transcription service API around `whisper.cpp`.
2. Create deterministic cleanup module with unit tests.
3. Implement macOS text insertion and hotkey flow.
4. Implement iOS keyboard extension dictation flow.
5. Add settings sync and mode controls.
6. Add QA corpus and regression tests for cleanup quality.

## Risks and Mitigations
- **iOS keyboard memory/perf constraints**
  - Mitigation: smaller model option + chunked inference.
- **Text insertion edge cases on macOS apps**
  - Mitigation: fallback pasteboard insertion path.
- **Over-aggressive cleanup altering meaning**
  - Mitigation: conservative rules + easy revert + verbatim mode.

## Definition of Done
- End-to-end dictation works on both devices for V1 scope.
- Modes behave as documented.
- Baseline tests pass.
- Release notes and onboarding tips completed.
