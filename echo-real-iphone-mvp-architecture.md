# Echo — Real iPhone MVP Technical Architecture

## 1. Role

Act as a senior iOS software architect and product engineer.

Design Echo as a realistic native iPhone application. Respect Apple’s iOS sandbox, privacy, security, App Intents, Siri Shortcuts, microphone, notification, and background-execution limitations.

Echo is not a password manager.

Do not include:

- Password autofill
- Login autofill
- Automatic login saving
- Credential capture
- Password breach monitoring
- Arbitrary control of other apps
- Persistent overlays above other apps
- Continuous background listening
- Hidden surveillance

## 2. Product goal

Echo is a premium, privacy-focused, voice-first personal assistant for iPhone.

Users can:

- Start a voice interaction
- Store secure personal notes
- Browse using a lightweight in-app browser
- Run supported phone actions
- Create reminders
- Manage personal preferences
- Delete saved data
- Lock the app instantly

Echo should feel calm, premium, private, fast, minimal, trustworthy, and non-intrusive.

## 3. Platform target

- iPhone
- Native iOS application
- SwiftUI
- iOS 17 or later
- Swift 5.9 or later
- Designed for TestFlight and App Store distribution

## 4. Visual design

- Deep navy: `#0A1628`
- Gold accent: `#C9A24B`
- Dark luxury aesthetic
- Sentence case
- Generous spacing
- Thin repeating gold pinstripes at the top and bottom of screens
- Simple outline icons
- No unnecessary gradients
- No heavy shadows
- Large touch targets
- Accessible contrast
- Subtle animations only

## 5. Recommended Apple technologies

Use:

- SwiftUI for the interface
- Observation or `ObservableObject` for state management
- SwiftData for non-sensitive structured data
- CryptoKit for encryption
- Keychain Services for encryption keys
- LocalAuthentication for Face ID, Touch ID, and passcode authentication
- AVFoundation for microphone and audio playback
- Speech framework for speech recognition
- `AVSpeechSynthesizer` for basic text-to-speech
- UserNotifications for reminders
- WebKit and `WKWebView` for Quick Browse
- App Intents for Shortcuts integration
- Siri Shortcuts for supported user-configured actions
- OSLog with redaction for safe diagnostics

## 6. Architecture style

Use a feature-based architecture separating presentation, application state, domain models, use cases, services, persistence, security, and Apple platform integrations.

Use:

- SwiftUI views
- Feature view models
- Protocol-based services
- Repository interfaces
- Dependency injection
- A central app router
- A central security/session manager
- Mock services for testing

The app must distinguish between real functionality, mock functionality, and unsupported iOS functionality. Do not present simulated behavior as real.

## 7. Project structure

```text
Echo/
├── App/
│   ├── EchoApp.swift
│   ├── AppRouter.swift
│   ├── AppState.swift
│   └── DependencyContainer.swift
├── DesignSystem/
│   ├── EchoColors.swift
│   ├── EchoTypography.swift
│   ├── EchoSpacing.swift
│   ├── EchoTheme.swift
│   ├── EchoButton.swift
│   ├── EchoCard.swift
│   ├── EchoIcons.swift
│   └── PinstripeBorder.swift
├── Core/
│   ├── Security/
│   │   ├── SecuritySession.swift
│   │   ├── BiometricAuthenticator.swift
│   │   ├── KeychainClient.swift
│   │   ├── EncryptionService.swift
│   │   └── SecureDataWiper.swift
│   ├── Audio/
│   │   ├── AudioSessionManager.swift
│   │   ├── MicrophonePermission.swift
│   │   └── AudioPlaybackService.swift
│   ├── Speech/
│   │   ├── SpeechRecognitionService.swift
│   │   ├── SpeechSynthesisService.swift
│   │   └── VoiceSessionCoordinator.swift
│   ├── Permissions/
│   │   ├── PermissionManager.swift
│   │   └── PermissionState.swift
│   ├── Networking/
│   │   ├── APIClient.swift
│   │   └── NetworkConfiguration.swift
│   └── Logging/
│       └── EchoLogger.swift
├── Domain/
│   ├── Models/
│   │   ├── SecureNote.swift
│   │   ├── ReminderItem.swift
│   │   ├── AssistantMemory.swift
│   │   ├── ActionHistoryItem.swift
│   │   ├── CallTranscript.swift
│   │   └── BrowserSession.swift
│   ├── Repositories/
│   │   ├── NotesRepository.swift
│   │   ├── RemindersRepository.swift
│   │   ├── MemoryRepository.swift
│   │   ├── HistoryRepository.swift
│   │   └── TranscriptRepository.swift
│   └── UseCases/
│       ├── CreateNoteUseCase.swift
│       ├── UpdateNoteUseCase.swift
│       ├── DeleteNoteUseCase.swift
│       ├── RestoreNoteUseCase.swift
│       ├── ClearAllDataUseCase.swift
│       ├── StartVoiceSessionUseCase.swift
│       ├── CreateReminderUseCase.swift
│       └── ExecuteAssistantCommandUseCase.swift
├── Features/
│   ├── Home/
│   ├── Voice/
│   ├── Browse/
│   ├── Notes/
│   ├── PhoneControl/
│   ├── Reminders/
│   ├── Memory/
│   ├── Lock/
│   └── Settings/
├── Integrations/
│   ├── AppIntents/
│   ├── Notifications/
│   └── Shortcuts/
├── Resources/
│   ├── Assets.xcassets
│   └── Localizable.xcstrings
└── Tests/
    ├── EchoUnitTests/
    └── EchoUITests/
```

## 8. Application state

Track current route, authentication state, lock state, active tab, voice session state, microphone state, privacy mode, pending destructive action, permission statuses, background state, and whether a note is unlocked.

```swift
enum AppLockState {
    case locked
    case unlocking
    case unlocked
}

enum VoiceSessionState {
    case idle
    case requestingPermission
    case listening
    case processing
    case speaking
    case muted
    case ended
    case failed(String)
}
```

Clear temporary decrypted note content when the app locks, enters the background, the user triggers emergency lock, or the security session ends.

## 9. Security architecture

1. Generate a random vault encryption key.
2. Store the key in the Keychain.
3. Protect Keychain access with device security where appropriate.
4. Encrypt note content with CryptoKit, preferably AES-GCM.
5. Store only encrypted content in SwiftData.
6. Authenticate before decrypting content.
7. Clear decrypted content when the app locks.
8. Do not place private note content in logs, analytics, notifications, screenshots, or app-switcher previews.

Support Face ID, Touch ID where available, device passcode fallback, manual lock, auto-lock, background lock, triple-tap emergency lock, and deletion of all local data.

Never hardcode real private information in sample data.

## 10. Secure notes

A secure note contains an ID, encrypted title, encrypted body, category, creation date, updated date, favorite status, pinned status, archived status, and deleted-at date when recently deleted.

Support create, view after authentication, edit, search after authentication, favorite, pin, archive, restore, delete, permanently delete, undo deletion, and empty Recently Deleted.

Categories:

- Personal
- Wi-Fi
- PIN
- Travel
- Work
- Medical
- Emergency
- Other

## 11. Delete and cleanup behavior

Every destructive action requires a confirmation sheet or alert, item name, Cancel action, clearly labeled destructive action, and Undo where practical.

Required flows:

- Delete one note
- Delete one transcript
- Delete one reminder
- Delete one memory item
- Delete one history item
- Clear all history
- Clear all transcripts
- Clear all memories
- Delete all notes
- Delete all Echo data

The Delete all Echo data flow must be separate from individual note deletion and must explain that it removes secure notes, Recently Deleted notes, reminders, memories, transcripts, history, browser session data, and preferences. Require authentication before deleting protected data.

## 12. Voice architecture

The first voice MVP includes microphone permission, audio session setup, speech recognition, listening and processing states, text response display, text-to-speech, mute, end call, private transcript mode, and permission/error states.

A mock assistant service may be used before connecting to an AI provider. Do not claim voice is fully functional until microphone input, speech recognition, response generation, audio output, permissions, and error handling work.

If a remote AI service is used, disclose that voice or transcript data may leave the device, do not send secure note content by default, require confirmation before sending private note content, use secure transport, and provide transcript retention controls.

## 13. Quick Browse

Use `WKWebView` for a lightweight browser. Support URL entry, search, back, forward, reload, close tab, clear session, open in Safari, and a voice button.

Do not implement password autofill, login saving, credential capture, hidden browsing surveillance, or secret browsing history.

Display:

> Browsing tracked off

This statement must accurately describe the implementation.

## 14. Phone controls

Use App Intents and Siri Shortcuts for supported actions.

Echo may open supported apps, run user-configured Shortcuts, open Messages, Maps, Camera, and Calendar, start supported Focus actions, trigger supported flashlight actions, and start user-created workflows.

Echo cannot promise unrestricted control over Wi-Fi, Bluetooth, other apps, private app data, or every system setting. If an action is unavailable, explain the limitation and offer a Shortcut, open the relevant Settings screen, or provide instructions.

Actions that send messages, make calls, share information, or change important settings require confirmation.

## 15. App Intents

Create App Intents for:

- Open Echo
- Start voice session
- Open Quick Browse
- Open Secure Notes
- Lock Echo
- Create reminder
- Run a configured assistant action

Each intent needs a clear title, user-friendly description, availability, error handling, and privacy-aware behavior.

## 16. Back Tap behavior

Echo cannot directly intercept iPhone Back Tap. The supported flow is:

```text
Back Tap → iOS Shortcut → Echo App Intent or Open Echo → Selected Echo experience
```

Recommended mapping:

- Single tap: Quick Browse
- Double tap: Voice session
- Triple tap: Lock Echo

Provide an in-app setup guide. Do not claim Echo independently listens for Back Tap events.

## 17. Settings

Include Back Tap setup, Face ID/passcode settings, auto-lock timing, background locking, lock now, voice output, speaking speed, transcript retention, private call mode, browsing tracking status, assistant memory toggle, action history toggle, local storage status, network-use disclosure, clear history, clear transcripts, clear memories, Recently Deleted, and Delete all Echo data.

## 18. MVP screens

Build these first:

1. Launch and lock screen
2. Home dashboard
3. Voice call screen
4. Quick Browse
5. Notes list
6. Note detail
7. Note editor
8. Delete confirmation
9. Recently Deleted
10. Phone Controls
11. Settings
12. Privacy and Data Management

Do not build advanced memory, cloud sync, family access, or complex routines until the core app is stable.

## 19. MVP priorities

### Priority 1 — Secure foundation

- App lock
- Secure local notes
- Create, edit, and delete notes
- Recently Deleted
- Delete all data
- Home screen
- Settings
- Empty states
- Error states

### Priority 2 — Voice interface

- Voice screen
- Microphone permission
- Speech recognition
- Text-to-speech
- Transcript display
- Transcript retention settings

### Priority 3 — Quick Browse

- Lightweight browser
- Browser cleanup
- Clear session
- Open in Safari
- Voice button

### Priority 4 — Apple integrations

- App Intents
- Siri Shortcuts
- Supported phone actions
- Reminders
- Notification scheduling

### Later

- Local assistant memory
- Optional encrypted sync
- Smart routines
- Advanced AI reasoning
- Sharing features
- Additional app integrations

## 20. Empty and error states

Implement empty states for notes, Recently Deleted, call history, reminders, memories, recent actions, and browser tabs.

Handle Face ID unavailable, authentication failure, microphone permission denied, speech recognition unavailable, network unavailable, AI service unavailable, Shortcut unavailable, browser loading failure, encryption failure, storage failure, delete failure, and notification permission denial. Never silently fail.

## 21. Accessibility

Support Dynamic Type, VoiceOver labels, sufficient contrast, Reduce Motion, large touch targets, clear destructive-action labels, non-color-only indicators, haptic feedback where appropriate, and accessible loading/error states.

## 22. Privacy requirements

Clearly tell users whether voice data leaves the device, whether transcripts are saved, whether notes remain local, what history is collected, how to delete data, which permissions are required, which actions are simulated, and which actions are unavailable because of iOS restrictions.

Do not say “Nothing leaves this device” unless the entire implementation guarantees that. Prefer accurate language such as:

> Secure notes are stored locally on this device.

## 23. Testing requirements

Create unit and UI tests for note creation, editing, deletion, undo deletion, permanent deletion, Delete all data, authentication success and failure, background locking, voice permission denial, transcript retention, browser session clearing, reminder creation and deletion, App Intent execution, empty states, accessibility labels, and data wiping.

## 24. Implementation rules

- Start with a compilable MVP scaffold.
- Use mock services where real services are not ready.
- Clearly label mock behavior.
- Do not hardcode private data.
- Do not include password-manager features.
- Do not include an autofill extension.
- Do not include login-saving functionality.
- Do not include family vault sharing in the MVP.
- Do not claim unsupported iOS functionality.
- Keep destructive actions explicit and reversible where practical.
- Keep secure data separate from ordinary app state.
- Use dependency injection for testability.
- Use local-only storage by default unless the user explicitly enables sync.

## 25. Expected deliverables

Produce:

1. Final architecture explanation
2. Clean project file structure
3. Data models
4. Security design
5. Navigation design
6. MVP implementation order
7. SwiftUI screen scaffolding
8. Mock voice service
9. Mock phone-control service
10. App Intent definitions
11. Unit test plan
12. UI test plan
13. Accessibility checklist
14. Privacy checklist
15. Security checklist
16. List of unsupported iOS features
17. Instructions for replacing mock services with real services

Start with the secure notes foundation, app lock, navigation, deletion flows, and visual design. Add voice and Apple integrations after the core foundation is stable.
