# Streaks — Bot specification

**Archetype:** workflow

**Voice:** supportive and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for tracking personal habits with streak mechanics. Users create habits with custom schedules, receive local-time reminders with one-tap check-ins, and view progress metrics. Supports manual edits, weekly summaries, and milestone celebrations—all while keeping data strictly private.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individual Telegram users
- habit-tracking enthusiasts

## Success criteria

- Users can create and manage habits with zero data loss across sessions
- 100% accurate streak tracking with idempotent check-ins
- Weekly summaries delivered at user-configured times
- All user data remains private with no third-party sharing

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with habit list and quick actions
- **Create habit** (button, actor: user, callback: habit:create) — Start habit creation flow
- **View habits** (button, actor: user, callback: habits:list) — Show compact habit overview
- **/week** (command, actor: user, command: /week) — Trigger weekly summary view

## Flows

### Onboarding
_Trigger:_ /start

1. Confirm timezone
2. Set language
3. Show quick-start options

_Data touched:_ User

### Habit creation
_Trigger:_ habit:create

1. Name habit
2. Choose recurrence type
3. Set reminder time
4. Confirm start date

_Data touched:_ Habit

### Reminder check-in
_Trigger:_ scheduled reminder

1. Send reminder message
2. Show 'Mark done' button
3. Record completion on tap

_Data touched:_ Occurrence

### Weekly summary
_Trigger:_ weekly schedule

1. Generate 7-day summary
2. Display streak stats
3. Show encouraging milestone message

_Data touched:_ Metrics

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account metadata
  - fields: telegram_id, timezone, language, summary_schedule
- **Habit** _(retention: persistent)_ — User-defined habit configuration
  - fields: title, recurrence_type, reminder_time, start_date, status
- **Occurrence** _(retention: persistent)_ — Daily habit tracking record
  - fields: date, status, completion_time, source, note
- **Metrics** _(retention: persistent)_ — Streak and completion statistics
  - fields: current_streak, best_streak, completion_rate

## Integrations

- **Telegram** (required) — Bot API messaging and scheduling
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure timezone and language
- Schedule weekly summary timing
- Create/edit/pause/delete habits
- Manually mark occurrences

## Notifications

- Scheduled reminders at user-chosen local time
- Weekly summary with progress visualization
- Milestone celebration messages

## Permissions & privacy

- All data is user-private
- No habit data sharing between users
- Local-time calculations respect user timezone

## Edge cases

- Timezone changes mid-flow
- Editing past habits without retroactive changes
- Double-tap prevention on check-in buttons

## Required tests

- End-to-end habit creation to streak tracking
- Idempotent check-in button behavior
- Weekly summary formatting across locales

## Assumptions

- Users will keep Telegram active for reminder delivery
- Default timezone is correctly inferred from Telegram metadata
