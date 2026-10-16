# Audio Watchdog

Project: https://github.com/Space-Egg111/Audio-Watchdog

## Objective

Keep Spotify playing continuously while automatically lowering its volume whenever Firefox or another selected application produces audio.

## Configuration

- Main source: spotify.exe
- Firefox: Duck Trigger
- FluentFlyout: Ignore
- Default role for unclassified apps: Ignore (recommended)
- Global ducking volume: 20%
- Pause grace period: 3 seconds
- Volume fade speed: Medium

## Troubleshooting

### Spotify pauses instead of ducking

Cause: Firefox was configured as Pause Trigger.

Fix: Set firefox.exe to Duck Trigger in Live Streams.

### Spotify volume changes unexpectedly

Cause: FluentFlyout appeared as Playing even without intentional playback.

Fix: Set FluentFlyout to Ignore.

## Verification

1. Play music in Spotify.
2. Start a YouTube video in Firefox.
3. Verify Spotify continues playing at reduced volume.
4. Pause YouTube and wait 3 seconds.
5. Verify Spotify volume returns to normal.

## Notes

Confirm final settings and startup behaviour before treating the configuration as fully validated.
