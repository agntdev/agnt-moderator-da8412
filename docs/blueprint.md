# Group Moderation Bot — Bot specification

**Archetype:** community

**Voice:** professional and warm — write every user-facing message, button label, error, and empty state in this voice.

A Telegram group moderation bot that automates spam prevention, user verification, and action logging. New members must verify via a button within 3 minutes (configurable), while admins can customize rules, trust lists, and auto-action thresholds. The bot enforces configured moderation actions (warn/mute/kick/ban) and tracks metrics for group health.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group owners
- Group admins

## Success criteria

- New members verify within configured timeout
- Spam actions are logged and enforced
- Admins receive daily/weekly summaries

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main moderation menu
- **/help** (command, actor: user, command: /help) — Show admin command reference
- **I'm human** (button, actor: user, callback: verify:confirm) — Verify membership and remove posting restrictions
- **/setwelcome** (command, actor: admin, command: /setwelcome) — Configure custom welcome message
- **/setautoactions** (command, actor: admin, command: /setautoactions) — Configure automated moderation rules

## Flows

### Join verification
_Trigger:_ new_member_joined

1. Send welcome message with verification button
2. Wait for button tap or timeout
3. Apply verification result

_Data touched:_ Member, RuleSet

### Spam enforcement
_Trigger:_ message_posted

1. Check message against spam rules
2. Apply configured actions (warn/mute/kick/ban)
3. Log action event

_Data touched:_ Member, RuleSet, ActionLog

### Admin command execution
_Trigger:_ /admin_command

1. Validate admin permissions
2. Execute command (warn/mute/etc)
3. Update log and metrics

_Data touched:_ ActionLog, TrustList, RuleSet

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Member** _(retention: persistent)_ — Group member with verification status and join time
  - fields: user_id, join_timestamp, verified, trusted
- **RuleSet** _(retention: persistent)_ — Moderation rules and thresholds
  - fields: welcome_message, verification_timeout, spam_thresholds, auto_actions
- **TrustList** _(retention: persistent)_ — Users exempt from moderation
  - fields: user_ids, exemption_reasons
- **ActionLog** _(retention: persistent)_ — Recent moderation events (last 200 entries)
  - fields: event_type, target_user, timestamp, reason
- **Metrics** _(retention: persistent)_ — Daily/weekly group health statistics
  - fields: total_joins, verified_members, auto_removals

## Integrations

- **Telegram** (required) — Bot API messaging and group moderation
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Edit welcome message and rules
- Configure verification timeout and spam thresholds
- Manage trust list
- Adjust auto-action settings (warn/mute/kick/ban)

## Notifications

- In-group explanations for automated actions
- Daily/weekly admin summaries in private chat

## Permissions & privacy

- Only admins can access /setwelcome, /setautoactions, and trust list commands
- Action logs retain user IDs but no personal data beyond Telegram's API
- Verification timeout and spam thresholds are owner-configurable

## Edge cases

- Users who join but never tap verification button
- Admins attempting to use commands without proper permissions
- Spam messages in pinned posts or from admins

## Required tests

- Verify new member verification flow with timeout handling
- Test spam detection triggers against configured thresholds
- Confirm admin command execution updates logs and metrics

## Assumptions

- Default verification timeout is 3 minutes
- Default auto-action sequence is warn then 5-minute mute
- Trust list includes all admins by default
