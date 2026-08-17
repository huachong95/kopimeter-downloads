# KopiMeter — downloads

> KopiMeter is an independent product. It is not affiliated with, endorsed by,
> or sponsored by Anthropic. Claude and Claude Code are trademarks of
> Anthropic, PBC.

A small desk meter for your Claude Code allowance — the kopitiam glass on
screen empties as you work and refills on the fresh pot, so you never meet the
limit mid-thought.

**[→ Set-up guide](https://huachong95.github.io/kopimeter-downloads/)** ·
**[→ Download for Windows or macOS](https://github.com/huachong95/kopimeter-downloads/releases/latest)**

This repository holds **only what customers download**: the installer, its
checksums, and the set-up guide. There is no source code here.

---

## Install — Windows

1. Download `KopiMeterSetup-<version>.exe` from the
   [latest release](https://github.com/huachong95/kopimeter-downloads/releases/latest).
2. Run it. **Windows will warn you** — see below.
3. Follow the [set-up guide](https://huachong95.github.io/kopimeter-downloads/);
   it takes about ten minutes and keeps your place as you go.

### "Windows protected your PC"

Expected. KopiMeter is a small independent project and the installer is not
code-signed, so Windows shows an **unknown publisher** warning. Click
**More info**, then **Run anyway**.

If you would rather check the download first, verify it against the checksum
published with the release — see below.

## Install — macOS

Requires an **Apple Silicon** Mac (M1 or later) on macOS 11 or newer.

1. Download `KopiMeter-<version>.dmg` from the
   [latest release](https://github.com/huachong95/kopimeter-downloads/releases/latest).
2. Open it and drag **KopiMeter** to Applications. **Do this before opening
   the app** — running it from the disk image works, but "Start at login"
   cannot point at a disk that will not be mounted next time, and KopiMeter
   will refuse rather than leave you a setting that quietly does nothing.
3. **Right-click the app and choose Open**, then confirm — see below.
4. **Allow Bluetooth** when macOS asks.
5. Click the cup in the menu bar and tick **Start at login**. macOS has no
   installer to do this for you, so without it KopiMeter is gone after a
   restart.
6. Follow the [set-up guide](https://huachong95.github.io/kopimeter-downloads/).

KopiMeter lives in the **menu bar**, not the Dock — look for the cup icon at
the top-right of your screen.

### "KopiMeter cannot be opened because the developer cannot be verified"

Expected, and macOS is stricter about this than Windows: a plain double-click
gives that message with **no way past it in the dialog**. Right-click (or
Control-click) the app and choose **Open** instead — the same dialog appears,
now with an Open button. You only do this once.

The app is not notarised for the same reason the Windows installer is not
signed: this is a small independent project and an Apple Developer account is
a recurring cost out of proportion to it.

### Bluetooth permission

macOS asks once, the first time KopiMeter looks for your device. If you miss
the prompt, enable it under **System Settings → Privacy & Security →
Bluetooth**. Without it the app runs normally and simply never finds the
device.

## Uninstalling — macOS

There is no uninstaller: a DMG is a drag, not an installer, so nothing is
tracked and there is nothing in System Settings to remove. Quit from the menu
bar, then paste this into Terminal:

```bash
launchctl bootout gui/$(id -u)/com.kopimeter.tray 2>/dev/null
rm -f  ~/Library/LaunchAgents/com.kopimeter.tray.plist
rm -rf /Applications/KopiMeter.app
```

That is the whole removal. To also drop your settings, logs and the remembered
pairing:

```bash
rm -rf ~/.config/kopimeter ~/Library/Logs/KopiMeter \
       "$HOME/Library/Application Support/KopiMeter"
```

Leaving those in place means a reinstall picks up where you left off. Finally,
forget the device under **System Settings → Bluetooth** if you are done with it.

On Windows, uninstall normally through **Settings → Apps**.

## Something not working?

`~/Library/Logs/KopiMeter/daemon.log` on macOS, or
`%LOCALAPPDATA%\KopiMeter\daemon.log` on Windows, is the log to send with a
support request — it is what turns "it stopped working" into a one-reply answer.

## Verifying your download (optional)

Windows ships `Get-FileHash` and macOS ships `shasum`, so this needs nothing
extra. Run the one for your platform against the file you downloaded.

Windows (PowerShell):

```powershell
Get-FileHash $HOME\Downloads\KopiMeterSetup-0.1.0.exe -Algorithm SHA256
```

macOS (Terminal):

```bash
shasum -a 256 ~/Downloads/KopiMeter-0.1.0.dmg
```

Compare what it prints with the line for that filename in the checksum file
for your platform — `SHA256SUMS-windows.txt` or `SHA256SUMS-macos.txt` — both
attached to the release **and** reproduced in the release notes. They should
match, ignoring upper/lower case. If they do not, delete the file and download
it again rather than running it.

Because the installer is unsigned, this checksum is the only integrity signal
the product has — which is why the hash is published in two places rather than
sitting next to the binary alone.

## What KopiMeter sends anywhere

Nothing leaves your computer except the usage poll your machine already makes.
The daemon reads the login `claude login` wrote, polls usage, and streams a
few numbers to the device over Bluetooth. The device has no Wi-Fi and stores
no credentials.

## Support

[Open an issue](https://github.com/huachong95/kopimeter-downloads/issues/new)
with what the tray icon says — that is usually enough to solve it in one
reply.

## Third-party components

KopiMeter bundles open-source components (LVGL, NimBLE-Arduino, ArduinoJson,
bleak, httpx, Pillow, pyobjc and others) under their own licences. The full
notice file travels with the download as `THIRD_PARTY_NOTICES.md` — beside the
app inside the DMG on macOS, next to the installed exe on Windows — and is
attached to each release.
