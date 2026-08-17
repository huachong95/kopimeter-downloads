# Cutting a release into this repo

This repo is the customer-facing half of KopiMeter. The source lives in the
private `huachong95/KopiMeter` repo; nothing but shippable artefacts and the
guide belongs here.

Two things get published, and they are independent:

| What | How it gets here | How often |
|---|---|---|
| `index.html` (the set-up guide) | `tools/publish-downloads.sh` in the source repo, then commit | whenever the guide changes |
| `KopiMeterSetup-<version>.exe` (Windows) | attached to a GitHub **Release** — never committed to the tree | per version |
| `KopiMeter-<version>.dmg` (macOS, Apple Silicon) | same release, same rule | per version |
| `SHA256SUMS.txt` | same release; see "Two platforms, one checksum file" below | per version |

Binaries are release assets, not tracked files. A committed `.exe` bloats
every future clone forever and cannot be replaced without rewriting history.

---

## 1a. Build the Windows installer (on Windows)

From a checkout of the **source** repo, on a real Windows machine — PyInstaller
only produces executables for the OS it runs on, so WSL will not do:

```powershell
powershell -ExecutionPolicy Bypass -File packaging\build-windows.ps1
```

That gates itself: it refuses to package if the test suite fails, and refuses
if the built exe fails `--selftest`. It writes, in order:

- `dist\KopiMeter.exe`
- `dist\KopiMeterSetup-<version>.exe`
- `dist\SHA256SUMS.txt` — generated **last**, covering only that run's
  artefacts

Then walk the customer path yourself on a machine **without Python installed**
(a fresh VM or Windows Sandbox); a dev box hides missing-dependency bugs
because the dependencies are already there. See `packaging/README.md`.

## 1b. Build the macOS app (on an Apple Silicon Mac)

```bash
bash packaging/build-macos.sh
```

Same shape as the Windows script and the same two gates: it refuses to package
if the test suite fails, and refuses if `KopiMeter.app --selftest` fails. It
writes `dist/KopiMeter.app`, `dist/KopiMeter-<version>.dmg` and
`dist/SHA256SUMS.txt`.

**Needs Python 3.13 exactly** — the lock is hash-locked to that minor and
`--require-hashes` refuses wheels it has no hash for. Set `KOPIMETER_PYTHON`
if 3.13 is not first on PATH.

No Mac? **CI builds it.** The `macos-package` job in `.github/workflows/ci.yml`
runs on GitHub's Apple Silicon runners and uploads `macos-installer` as a
workflow artifact on every push — download that and attach it to the release.
It builds and self-tests the bundle on a real arm64 Mac, but the runner has no
Bluetooth radio, so pairing is still only provable on your own machine.

### Two platforms, one checksum file

Each build script writes a `SHA256SUMS.txt` covering **only its own run's
artefacts** — so building on Windows then on macOS leaves you with two files
that each know about half the release. Concatenate them before attaching:

```bash
cat windows/SHA256SUMS.txt macos/SHA256SUMS.txt > SHA256SUMS.txt
```

Never retype a hash by hand. A mistyped digest sends a customer who followed
the instructions a false alarm, which teaches everyone to skip the step.

## 2. Publish the release

```
Tag:   v<version>              e.g. v0.1.0
Title: KopiMeter <version>
```

Attach all three:

- `KopiMeterSetup-<version>.exe` — Windows
- `KopiMeter-<version>.dmg` — macOS, Apple Silicon
- `SHA256SUMS.txt` — covering both (see above)

**Also paste the contents of `SHA256SUMS.txt` into the release notes.** This is
not redundancy for its own sake: a checksum served from the same place as the
binary is verified by whatever served the binary, so anyone who can replace one
can replace the other. Release notes live in GitHub's database rather than in
object storage, so the two would have to be breached separately. With the
installer unsigned, this is the only integrity control the product has.

### Release-notes template

```markdown
KopiMeter <version> — Windows and macOS (Apple Silicon).

**[Set-up guide](https://huachong95.github.io/kopimeter-downloads/)** — about
ten minutes, keeps your place as you go.

**Windows** will show "Windows protected your PC" with an unknown publisher.
Click **More info** → **Run anyway**.

**macOS** will say the developer cannot be verified, with no override in the
dialog — **right-click the app and choose Open** instead. You only do this
once. macOS also asks for Bluetooth permission the first time.

Both are expected: this is a small independent project, so the installer is
not code-signed and the app is not notarised. Verify the download against the
checksum below if you would rather check first.

KopiMeter needs an **Apple Silicon** Mac (M1 or later) on macOS 11+.

### SHA256
```
<paste SHA256SUMS.txt verbatim>
```

KopiMeter is an independent product. It is not affiliated with, endorsed by,
or sponsored by Anthropic.
```

## 3. Update the guide, if it changed

From the source repo:

```bash
tools/publish-downloads.sh /path/to/kopimeter-downloads
cd /path/to/kopimeter-downloads
git add index.html && git commit -m "Update guide for <version>" && git push
```

The script refuses to publish if the guide gained an external asset reference
(it must stay one self-contained file) or if its QR code stops encoding the
Pages URL.

---

## One-time setup

**GitHub Pages must be enabled** or the QR printed on the box card leads
nowhere: *Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)`*.
The guide is then served at
`https://huachong95.github.io/kopimeter-downloads/`.

`.nojekyll` is committed so Pages serves the file as-is instead of running it
through Jekyll.

## Filenames and links

The installer filename carries its version (`KopiMeterSetup-0.1.0.exe`), so
there is **no** stable `releases/latest/download/KopiMeterSetup.exe` URL —
that permalink form only works for a fixed filename. Everything that points a
customer at the download uses `releases/latest`, the page, which always
resolves. If you ever want a fixed-name permalink, drop the version from
`OutputBaseFilename` in `packaging/installer.iss` first; don't add the link and
hope.
