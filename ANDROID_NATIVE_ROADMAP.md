# Send: Android-native experience roadmap

Product goal: deliver a low-clutter Google Messages-inspired Compose UI without Google branding/assets, on top of Element X's proven Matrix Rust SDK, crypto, sync, account and call architecture. This is a backlog, not a claim that integrations are implemented. Preserve essential login/session restore, verification/key backup, room membership, E2EE, timeline, reaction and call behavior while replacing the visual/navigation product surface. Upstream Element X is dual licensed AGPL-3.0-only OR Element commercial; retain notices and review distribution/license obligations before release. This is not legal advice.

## Architecture rule

Rust SDK/session/timeline/crypto is the sole source of truth. Android surfaces project the same conversation state and actions; they are not a second message database. Every system action resolves account + room, restores SDK and crypto state, rechecks room membership/effective encryption and user permission, then uses the same SDK operation path. Handle process death, logout, account switching, stale rooms, locked crypto, offline state and send errors. Do not claim delivery/read before SDK/server confirmation.

## P0 — Android conversations and notification correctness

- Post one notification per relevant room using AndroidX `NotificationCompat.MessagingStyle`, sender `Person`, bounded recent history and stable long-lived dynamic conversation shortcuts. Associate shortcut ID, notification, and deep link; IDs must be opaque/app-local, stable on rename, and never contain message/room/user plaintext.
- Add mark-read and inline `RemoteInput` reply only through authenticated SDK/crypto operations. Use distinct `PendingIntent` identity per account/room; mutable PendingIntent only where Android RemoteInput requires it. Deduplicate send/retries by transaction identity. Do not mark read merely because a notification was posted.
- Respect API 33+ runtime notification permission, stable channels, DND, priority and all user/system controls. A denied notification permission means no notification surface, never permission to bypass Android.
- Default lock-screen content to generic redacted “New message”; let users explicitly opt into sender/body previews. Notification shade/history, companion surfaces, Auto, widgets, TTS, STT, Assistant, AI and screenshots can expose decrypted plaintext outside Matrix E2EE. No message bodies/tokens/keys in logs, analytics, shortcut IDs or push extras. Push payload metadata is also a disclosure boundary.
- RemoteInput text is ephemeral plaintext: send directly into SDK crypto pipeline without persistence/logging. If session/crypto is unavailable, safely route to unlock/open app; never fallback to unencrypted send.

Sources: [People and conversations](https://developer.android.com/develop/ui/views/notifications/conversations) · [Matrix Push Gateway API](https://spec.matrix.org/v1.19/push-gateway-api/)

## P1 — Android Auto and native calling

### Android Auto

- Start with notification-powered Android Auto: `MessagingStyle`, `RemoteInput` reply and mark-read actions handled in a service/receiver, not by launching the Activity. Auto voice dictation arrives as RemoteInput text and must use the same SDK E2EE path.
- Add templated Android Auto conversation browsing only after background session and notification actions are robust. Test with an actual supported Auto/head-unit environment. Keep car-facing history/actions minimal; passengers, car display and voice services can see previews. Matrix is not SMS/RCS: no SMS role or SMS/MMS permission.

[Android Auto messaging](https://developer.android.com/training/cars/communication/notification-messaging)

### MatrixRTC call surfaces

- First integrate a real compatible MatrixRTC call implementation. `CallStyle` is presentation, not media/signaling. Use incoming/ongoing/ended `NotificationCompat.CallStyle` with answer/decline/hang-up actions.
- Evaluate AndroidX Core-Telecom for self-managed VoIP (call lifecycle, audio focus/routing, Bluetooth, concurrent calls and system integration) only after confirming compatibility with the chosen MatrixRTC implementation. Follow current self-managed call, `MANAGE_OWN_CALLS`, foreground-service type/permission, Android 14+ restrictions, background-start limits and notification deadlines. Full-screen intents are restricted and cannot be the only incoming-call surface. Stop call foreground work promptly on end/decline/revocation.
- Ask for microphone permission only on explicit call start/accept. Keep room identity, tokens, membership secrets and call credentials out of notification extras. Telecom does not implement Matrix signaling/media/E2EE.

[CallStyle](https://developer.android.com/develop/ui/compose/notifications/call-style) · [Core-Telecom](https://developer.android.com/develop/connectivity/telecom/voip-app/telecom) · [FGS types](https://developer.android.com/develop/background-work/services/fgs/service-types)

## P2 — shortcuts, bubbles, widgets, sharing

- Launcher shortcuts: New message and recent/pinned conversation; direct-link into correct account/room. Direct-share/Chooser targets for intentional text/media sharing; confirm recipient before send.
- Bubbles: optional and user-controlled, never the primary UI. Require valid conversation shortcut and deep link, use the same full-security conversation screen, and retain standard notification fallback. Never auto-expand.
- Android system Conversation Widgets are eligible from API 31+ via conversation shortcuts; custom widgets are separate. Widget refresh is system-constrained, not a real-time sync mechanism. Default to unread/activity metadata or explicitly chosen preview; never assume widgets are private or fresh. Do not send from widgets until SDK process-death/crypto/action handling is complete.
- Keep PendingIntent identity unique per room/account and mutable only for RemoteInput where required.

[Bubble API](https://developer.android.com/develop/ui/compose/notifications/bubbles) · [App widgets](https://developer.android.com/develop/ui/views/appwidgets)

## P2 — dictation, read-aloud, Assistant and Gemini Nano

- Dictation: start Android `SpeechRecognizer` or supported ML Kit on-device speech only after explicit composer tap. Test recognition language/network/device availability; keep normal keyboard and manual correction. Transcript remains a draft until reviewed and explicitly sent. No always-on capture.
- TTS: Android `TextToSpeech` for user-selected messages/conversation; explicit start/stop, respect audio focus/route; never automatically speak sensitive text.
- Google Assistant App Actions are declared through supported `shortcuts.xml` capabilities/BIIs; there is no general private Gemini-app plugin API. Limit to navigation/open compose/draft; disambiguate in app, require confirmation before send, and do not expose messages or bypass unlock. Availability depends on Google services, locale and policy.
- Optional ML Kit GenAI/Gemini Nano on-device: user-invoked proofreading, rewrite, summary, or transcription when model/device supports it. Minimize selected context, label generated text, retain as editable draft, never auto-send. Cloud AI on decrypted message content must be separate explicit consent; it transmits plaintext outside Matrix E2EE. All AI remains optional with a no-AI path.

[SpeechRecognizer](https://developer.android.com/reference/android/speech/SpeechRecognizer) · [TextToSpeech](https://developer.android.com/reference/android/speech/tts/TextToSpeech) · [App Actions](https://developer.android.com/develop/devices/assistant/intents) · [Gemini Nano](https://developer.android.com/ai/gemini-nano)

## P3 — API 37 and cross-device

- Android Contact Picker (API 37) instead of broad `READ_CONTACTS`; on API 34–36 use system `ACTION_PICK` or manual address entry. A selected phone/email is not a Matrix identity. Identity lookup may disclose identifiers to a homeserver/service; explain, minimize and ask before lookup/upload.
- Android 17 Handoff/Continue On (API 37) only for minimal non-sensitive navigation state (opaque room key/scroll anchor). Never transfer event bodies, names, tokens, crypto keys or session. Receiving device authenticates and authorizes independently; retain normal sign-in/deep-link fallback.
- Guard newer APIs and maintain API 34+ path; review target-37 behavior changes before raising target.

[Android 17 features](https://developer.android.com/about/versions/17/features) · [Continue On](https://developer.android.com/develop/better-together/continue-on) · [Contact Picker](https://developer.android.com/about/versions/17/features/contact-picker)

## Native polish, all screens

- Edge-to-edge/insets and IME; dynamic color; adaptive/themed icon; predictive back; font/display scaling; TalkBack/switch access labels/order/announcements; contrast/touch targets; responsive tablet/foldable list-detail layout; native share/clipboard/permissions and restrained haptics.
- `MessagingStyle` for messages, `CallStyle` for calls. Do not use Live Updates/ProgressStyle for normal messages.
- Redact recents/task snapshots and lock-screen notifications by user preference; never log plaintext; secure-screen capture only where justified because it harms usability and cannot prevent all capture.
- Test each OS entry path through process death, account switch, stale room, offline send, permission denial, crypto restore failure, Doze, lock screen and user-customized notification settings. Bubbles/widgets/Auto/Assistant/STT/TTS/Gemini are optional and OS/OEM controlled, not guaranteed on every device.

## Delivery order

1. Preserve Element X SDK/auth/session/recovery/sync/call foundations; redesign inbox and timeline UI.
2. Conversation shortcuts + redacted MessagingStyle notifications.
3. Safe RemoteInput reply/read through Rust SDK.
4. Real MatrixRTC implementation + CallStyle; add Core-Telecom only after live call verification.
5. Android Auto, direct share, widgets and opt-in bubbles.
6. Explicit dictation/TTS and narrow App Actions; later optional Gemini Nano.
7. Contact Picker/Handoff on API 37 after API 34+ behavior is stable.

Every item requires functional behavior and device/API verification, not placeholder controls or UI-only mocks.
