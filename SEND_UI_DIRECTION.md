# Send UI direction

## Product boundary

Send is a Matrix messenger inspired by the clarity and conversation-first hierarchy of Google Messages on Pixel. Do not copy Google names, logos, proprietary assets, exact artwork, or source. Retain Element X's Rust SDK, E2EE, sync, authentication, recovery and protocol behavior; replace the Element presentation, not the security model.

## First redesign target: inbox

- A single prominent “Messages” header, search affordance, and account/avatar access.
- Conversation-first vertical list: large contact/room avatar, clear name, one-line latest-message preview, relative/short time, unread emphasis/count and restrained E2EE status where useful.
- Filter chips support useful Matrix inbox subsets but should not expose Element's Spaces-first navigation; simplify to Messages/Unread/Invites as functions actually supported.
- One clear bottom-end “New conversation” action; remove the floating toolbar and deprioritize Spaces, while preserving invite/recovery/security entry points in context/menu destinations.
- Empty, loading, offline, invite, encryption/recovery-needed states remain real data-driven screens; never fill UI with pretend conversations.

## Conversation target

- Compact top bar with back, avatar/name, encryption/verification affordance, and voice-call action only when the selected MatrixRTC implementation can execute it.
- Chronological timeline with readable incoming/outgoing bubbles, restrained timestamps, delivery status, reply/reaction/context actions and accessible long-press semantics.
- Composer remains a single compact text field with accessible send/action controls and draft preservation. Media/voice composition follows only when actual SDK paths are ready.

## Visual system

- Use a quiet warm/neutral surface, strong readable ink, Material 3 dynamic color where available, generous spacing and rounded conversation surfaces; avoid faux Google branding and excessive gradients.
- Match Android edge-to-edge, IME, predictive back, large text, TalkBack and adaptive layout behavior.
- Screen-level system, user and Matrix state must remain visible and truthful. Do not trade encryption warnings, identity verification, failures or permissions for visual minimalism.

## Source references

- [AOSP Messaging](https://android.googlesource.com/platform/packages/apps/Messaging) is an open-source Android SMS/MMS client, useful for studying platform integration, but it is not Google Messages and its telephony behavior is not a Matrix implementation.
- [PVOT Messages](https://github.com/PVOT-OSS/Messages) continues QKSMS with an opinionated messaging UI and native SMS/MMS features. Use only as a UX reference; its GPL-3.0 code and telephony-specific architecture are not copied into this AGPL/commercial Element X fork.
- The user-provided Google Messages XAPK is the direct visual reference. Its decoded resources include a newer named inbox variant with an 80dp row, 52dp contact icon and 20dp/4dp large/grouped bubble corners; these dimensions inform Send's layout proportions only. No Google code, assets, branding or private APIs are reused.

## Migration method

1. Replace the inbox shell while retaining existing state, room-opening callback, invite/recovery/settings/context pathways.
2. Rework conversation visual components without removing timeline event types or SDK action semantics.
3. Add native notifications/shortcuts and call UI only via the same session/room repository and permission/crypto checks.
4. Remove old surface/routes only after all deep links, notifications, settings, recovery, accessibility and tests have migrated. Use GitHub CI for builds/tests/lint; do not run them locally.

The supplied Google Messages XAPK is `messages.android_20261002_00_RC00` (target SDK 37). Static decompilation is permitted for visual/resource inspection only; do not copy its code, assets, branding, or private APIs, and do not install or execute it.
