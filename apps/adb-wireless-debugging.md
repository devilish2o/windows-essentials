# Android ADB over Wi-Fi (Windows 11)

Use Android Debug Bridge (ADB) over a trusted, dedicated Wi-Fi network to control an Android phone from Windows without a USB cable or a third-party remote-control service.

This guide documents the setup used on **RANGER (ThinkPad X1 Carbon, Windows 11)** with a **Samsung Galaxy S23 Ultra**. The example use case is sending swipe gestures to TikTok using an existing **AutoHotkey v2** script.

## What this provides

- Free Google Android Platform Tools (`adb.exe`).
- Wireless pairing between the phone and RANGER.
- No third-party Android app, camera access, cloud account, or subscription.
- Windows keyboard shortcuts can send Android touch gestures.
- Wireless debugging can be switched off whenever it is not needed.

## 1. Install Android Platform Tools on Windows

1. Download the official [Android SDK Platform Tools for Windows](https://developer.android.com/tools/releases/platform-tools).
2. Extract the ZIP to:

   ```text
   C:\platform-tools
   ```

3. Open PowerShell and verify the installation:

   ```powershell
   cd C:\platform-tools
   .\adb.exe version
   ```

Keep this folder in place; the AutoHotkey example below expects `C:\platform-tools\adb.exe`.

## 2. Enable Wireless debugging on the S23 Ultra

Developer options are required for Android's built-in Wireless debugging feature.

1. On the phone, open **Settings → About phone → Software information**.
2. Tap **Build number** seven times.
3. Confirm with the phone PIN if prompted.
4. Return to Settings and open **Developer options**.
5. Enable **Wireless debugging**.
6. Accept the confirmation prompt.

Keep the phone and RANGER connected to the same trusted Wi-Fi network. A dedicated Wi-Fi network is preferable. Do not expose ADB ports through router port forwarding or to the public internet.

## 3. Pair RANGER with the phone

1. On the S23, open **Developer options → Wireless debugging**.
2. Select **Pair device with pairing code**.
3. The phone displays an IP address, a temporary pairing port, and a six-digit pairing code.
4. In PowerShell on RANGER, run the command using the IP and pairing port shown on the phone:

   ```powershell
   cd C:\platform-tools
   .\adb.exe pair PHONE_IP:PAIRING_PORT
   ```

   Example only (use the values currently displayed on your phone):

   ```powershell
   .\adb.exe pair 192.168.1.50:37123
   ```

5. Enter the six-digit pairing code when prompted.

**Important:** The pairing port is not the same as the normal Wireless debugging connection port. Do not assume they match.

## 4. Connect to the phone

1. Return to the main **Wireless debugging** screen (not the pairing-code screen).
2. Find the IP address and port shown under the device's wireless debugging details. This is the connection port.
3. On RANGER, run:

   ```powershell
   .\adb.exe connect PHONE_IP:CONNECTION_PORT
   ```

   Example only:

   ```powershell
   .\adb.exe connect 192.168.1.50:39871
   ```

4. Confirm the connection:

   ```powershell
   .\adb.exe devices -l
   ```

The phone should appear with a state of `device`. If it says `unauthorized`, check the phone for an authorization prompt. If it says `offline`, reconnect or toggle Wireless debugging off and on, then connect again.

The connection port can change when Wireless debugging is restarted. Always use the current port displayed by Android.

## 5. Test a swipe from PowerShell

Open TikTok on the phone and make sure a video is visible.

Check the current display dimensions:

```powershell
.\adb.exe shell wm size
```

For a display reporting `Physical size: 1080x2340`, test a swipe up:

```powershell
.\adb.exe shell input swipe 540 2000 540 600 300
```

Test a swipe down:

```powershell
.\adb.exe shell input swipe 540 600 540 2000 300
```

The command format is:

```text
adb shell input swipe START_X START_Y END_X END_Y DURATION_MS
```

Adjust the coordinates to the phone's reported logical display size and the app's usable screen area. These examples assume a 1080 × 2340 coordinate space; they are not universal.

## 6. Add TikTok shortcuts to an existing AutoHotkey v2 script

Do not create a second resident AHK script if RANGER already has one. Add the following hotkeys to the existing `.ahk` file.

Replace `192.168.1.50:39871` with the current ADB connection endpoint shown by `adb devices`. If the connection port changes, update this value or use the reconnect procedure below.

```autohotkey
; TikTok control on the paired Android phone.
; Ctrl + Alt + Down: next video (swipe up)
^!Down::
{
    RunWait '"C:\platform-tools\adb.exe" -s 192.168.1.50:39871 shell input swipe 540 2000 540 600 300', , "Hide"
}

; Ctrl + Alt + Up: previous video (swipe down)
^!Up::
{
    RunWait '"C:\platform-tools\adb.exe" -s 192.168.1.50:39871 shell input swipe 540 600 540 2000 300', , "Hide"
}
```

Save the existing script and choose **Reload Script** from its AutoHotkey tray icon. Test with TikTok in the foreground on the phone.

The coordinates above are examples. Adjust them to the S23's logical display dimensions as reported by `adb shell wm size`.

## 7. Reconnect after a Wi-Fi or debugging restart

Wireless debugging may assign a different connection port after being disabled, restarted, or after a network change.

On the phone, open **Developer options → Wireless debugging** and note the current IP address and connection port. Then run:

```powershell
cd C:\platform-tools
.\adb.exe connect PHONE_IP:CONNECTION_PORT
.\adb.exe devices -l
```

If the phone's IP changes frequently, create a DHCP reservation for the S23 in the router. A DHCP reservation stabilizes the IP address but does not guarantee that Android's ADB connection port will remain the same.

To disconnect the current endpoint:

```powershell
.\adb.exe disconnect PHONE_IP:CONNECTION_PORT
```

## 8. Security and privacy

- Pair only with RANGER and only on a trusted network.
- Android's pairing code is temporary; do not share it.
- ADB pairing authorizes a trusted computer. Keep RANGER secured and remove its debugging authorization from the phone if the computer is no longer trusted.
- Do not configure router port forwarding for ADB.
- Do not expose ADB ports to the internet or an untrusted network.
- Turn **Wireless debugging** off in Developer options when finished. This stops the wireless ADB listener.
- A dedicated Wi-Fi network reduces exposure, but should still use a strong Wi-Fi password and should not be treated as a substitute for disabling Wireless debugging when idle.

To revoke previously trusted debugging computers, use the **Revoke USB debugging authorizations** option in Developer options (Android wording may vary). This may require pairing RANGER again next time.

## Troubleshooting checklist

| Symptom | What to check |
|---|---|
| `adb` is not recognized | Run `.\adb.exe` from `C:\platform-tools`, or add that folder to PATH. |
| `failed to connect` | Confirm same Wi-Fi, Wireless debugging enabled, and current connection IP/port. |
| `unauthorized` | Check the phone for an authorization prompt. |
| `offline` | Toggle Wireless debugging off/on and reconnect using the newly displayed port. |
| Pairing fails | Use the temporary pairing port and current six-digit code from **Pair device with pairing code**. |
| Pairing worked but connect fails | Use the separate connection port on the main Wireless debugging screen. |
| A swipe does nothing | Confirm TikTok is foreground, then verify display dimensions and swipe coordinates. |

## Official reference

- [Android Debug Bridge (ADB)](https://developer.android.com/tools/adb)
- [Android SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools)
- [Wireless debugging in Android Studio](https://developer.android.com/tools/adb#connect-to-a-device-over-wi-fi-android-11+)
