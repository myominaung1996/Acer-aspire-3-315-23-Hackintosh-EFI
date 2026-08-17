# Velocity VPN — Session Notes (Q)

Project path (local): `D:\App Creation\VelocityVPNF`  
Package: `com.mma.velocityvpn`  
Display name: **Velocity VPN**  
Backend: sing-box / libbox (Karing-style, not OpenVPN)

---

## 1. Windows — Connect & TUN

### Leftover TUN adapter
- **Symptom:** Error / Retry — `Leftover TUN adapter...` or `start inbound/tun[tun-in]: configur...`
- **Cause:** Previous session left WinTUN / sing-box orphan (crash, second window, tray quit without cleanup).
- **Quick fix:** Close all Velocity windows → Task Manager end `VelocityVPN` / `sing-box` → wait 2–3s → **Run as administrator** → Retry.
- **App fix (Agent):** Cleanup only on **detected** leftover (not every connect). No 2s sleep on happy path. Single-instance lock. Graceful tray Quit stops VPN **before** exit.

### Admin / elevation
- **Karing-style (preferred):** GUI = normal user; TUN = elevated helper/service on Connect only.
- **Temporary:** Right-click → Run as administrator.
- **`flutter run` elevation error:** Manifest must be `asInvoker`, not `requireAdministrator`. TUN privilege on Connect only.

### System-wide VPN
- TUN mode = whole Windows (browser, IDM, games). Needs elevation for WinTUN.
- IDM error *"Could not find proxy configuration in System Settings"* → IDM Options → **No proxy** (TUN does not set system proxy).

### Tray icon missing
- App opens but no tray / hidden icons → tray init broken or dependency removed.
- **Agent:** Restore tray on startup; Show / Quit menu; Quit = graceful VPN stop then exit.

---

## 2. Windows — UI / Layout

### Default window size
- Change from **400×700** → **400×900** in `windows/runner/main.cpp` (or equivalent).
- Update min height if defined.

### Home Tab — single screen (no scroll)
**Order (top → bottom):**
1. Status row (Protected / Device ID)
2. Server card (compact)
3. Metric tiles (fixed size)
4. Traffic graph (fixed height ~80–100px)
5. Auto-select + Kill switch
6. **Connect button last** (below all cards)

- No layout jump on connect/disconnect.
- Connect animations: **breathe** (idle) → **spin** (connecting) → **shield fade** (connected).

### Theme: Aurora Soft (chosen)
- Tokens: mist `#F3F7FB`, ink `#1A1A1A`, coral `#FF6B4A`, amber `#FFB020`, aqua `#2EC4B6`
- Light gradient background; no dark teal default; no purple AI look.
- Brand **Velocity** hero on Home.

---

## 3. Device ID Lock / Share

**Not Phase 5/6** — implemented in Phases 1–3:

| Phase | Scope |
|-------|--------|
| 1 | Models: Device ID, allow-list, `SECURE_APP_ONLY`, `velocity-locked://` |
| 2 | UI: Settings Device ID + copy; Share QR; Import reject |
| 3 | Connect gate: wrong Device ID → `start()` blocked |

User reported feature **not fully in app yet** — use full end-to-end Agent prompt in chat history.

---

## 4. Android — Build Fix

### Error
```
:file_picker:checkDebugAarMetadata
flutter_plugin_android_lifecycle requires compileSdk 36+
:file_picker compiled against android-34
```

### App module (`android/app/build.gradle.kts`)
```kotlin
compileSdk = 36
```

### Plugin modules — do NOT use `afterEvaluate` (breaks Gradle)
**Wrong (causes line 29 error):**
```kotlin
subprojects {
    afterEvaluate { ... }  // "Cannot run afterEvaluate when already evaluated"
}
```

**Correct (`android/build.gradle.kts` root):**
```kotlin
import com.android.build.gradle.LibraryExtension

subprojects {
    plugins.withId("com.android.library") {
        extensions.configure<LibraryExtension> {
            compileSdk = 36
        }
    }
}
```

### Commands
```powershell
taskkill /F /IM java.exe 2>$null
flutter clean
flutter pub get
flutter run
```

If `flutter clean` fails to remove `build/`, close Velocity / Gradle / Studio and delete `build` manually.

---

## 5. Connect performance (Windows)

- Target: ~1–2s (Karing-like), not ~5s.
- Avoid: cold rebuild every connect, 2s leftover wait on every start, blocking UI probes.
- Add timing logs: `leftover_check_ms`, `cleanup_ms`, `tun_create_ms`, `total_ms`.

---

## 6. Config Tab redesign

- User rejected early card-heavy mockups as “cheap.”
- Directions explored: Editorial, Obsidian Glass, Studio Black; then **Solar Pop** vs **Aurora Soft**.
- **Aurora Soft** selected for full-app theme.

---

## 7. Agent prompts (copy to VelocityVPNF Cursor)

### Leftover TUN (fast path only)
```
On start, if leftover TUN detected: auto-cleanup once, wait 300–500ms max, retry once.
Do NOT run cleanup + 2s wait on every Connect.
Single-instance lock. Graceful tray Quit stops VPN before exit.
```

### Tray graceful quit
```
Tray Quit: await disconnect/stop → WinTUN destroy → kill orphan sing-box → then exit.
Shared shutdownVpnAndExit() for tray Quit (not parallel race).
```

### Android compileSdk
```
Remove afterEvaluate hack. Use plugins.withId("com.android.library") { compileSdk = 36 }.
Keep app compileSdk = 36.
```

### Aurora theme
```
ui: Aurora Soft premium theme across app (see tokens above).
```

### Home single-screen layout
```
ui(home): single-screen 400x900, connect below all cards, fixed metrics/graph heights.
```

---

## 8. Phase reference (Velocity VPN)

| Phase | Scope |
|-------|--------|
| 1–3 | Data, Import/UI, Connect + Lock/Share |
| 5 | Kill switch, split, DNS, tile, battery (no Lock/Share) |
| 6 | iOS/macOS tunnel (no Lock/Share) |
| 7 | Windows |
| 8–9 | Polish, l10n, legal |

---

## 9. Distribution notes

- **Play In-App Update:** Play Store installs only; not for GitHub APK.
- **Play VPN:** Organization account + D-U-N-S required.
- **Apple VPN:** Individual $99 possible; Network Extension needs proper entitlements.

---

## 10. libbox Android

- Build: `scripts/build_libbox.ps1` → `packages/vpn_engine/android/libs/libbox.aar` (gitignored).
- Requires Go + JDK 17 on dev machine.

---

*Generated from Cloud Agent session notes. Edit in VelocityVPNF repo or copy to local docs.*
