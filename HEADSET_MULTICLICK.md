# Headset Multi-Click Detection

This document describes the headset multi-click detection feature implemented in audio_service.

## Overview

Audio service now automatically supports multi-click detection for headset buttons, similar to Android's native audio session behavior. This feature works seamlessly without any configuration required.

## Behavior

When a user presses the headset button (play/pause button):

- **Single click**: Toggle play/pause
- **Double click**: Skip to next track
- **Triple click**: Skip to previous track

## Technical Details

### Detection Window
- The detection window is set to **300ms** between clicks
- If clicks occur more than 300ms apart, they are treated as separate single clicks
- Clicks within 300ms are accumulated and processed after the timeout

### Platform Implementation

#### Android
Implementation is in `AudioService.java` in the `MediaSessionCallback` class:
- Uses `Handler` with `postDelayed` to manage click timeout
- Only applies multi-click detection to `KEYCODE_HEADSETHOOK` events
- Direct next/previous button presses are handled normally

#### iOS/macOS
Implementation is in `AudioServicePlugin.m`:
- Uses `NSTimer` with scheduled block execution
- Applies to `togglePlayPauseCommand` events
- Direct next/previous track commands are handled normally

### Code Flow

1. User presses headset button
2. Platform detects key event (Android: `KEYCODE_HEADSETHOOK`, iOS: `togglePlayPauseCommand`)
3. Click counter increments
4. Any pending timer is cancelled
5. New timer is scheduled for 300ms
6. If another click comes within 300ms, counter increments and timer resets
7. After 300ms with no new clicks, action is executed based on click count:
   - 1 click → `onClick(MediaButton.media)` or `click` method
   - 2 clicks → `skipToNext()`
   - 3+ clicks → `skipToPrevious()`

## Compatibility

- ✅ Android: All versions supported by audio_service
- ✅ iOS: All versions supported by audio_service
- ❌ macOS: Not implemented (would require `togglePlayPauseCommand` handler)
- ❌ Web: Not applicable (no headset button support)

## User Experience

The feature is designed to be intuitive:
- Users familiar with Android's audio session behavior will find it natural
- The 300ms timeout provides a good balance between responsiveness and multi-click detection
- Single clicks have a 300ms delay to allow for multi-click detection

## Example Usage

No code changes are required! The feature works automatically with any existing audio_service implementation:

```dart
final _audioHandler = await AudioService.init(
  builder: () => MyAudioHandler(),
  config: const AudioServiceConfig(
    androidNotificationChannelId: 'com.mycompany.myapp.channel.audio',
    androidNotificationChannelName: 'Music playback',
  ),
);
```

Users can now:
1. Single-click headset button → play/pause toggles
2. Double-click headset button → skip to next track
3. Triple-click headset button → skip to previous track

## Future Enhancements

Potential improvements for future versions:
- [ ] Configurable timeout via `AudioServiceConfig`
- [ ] Configurable action mapping (e.g., swap double/triple click actions)
- [ ] macOS support via similar implementation
- [ ] Disable/enable multi-click detection via configuration
