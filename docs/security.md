# MewTools security

MewTools runs as administrator on the second PC, so it is built to keep that trust narrow. This page describes how the app handles commands, downloads, updates and files, what was checked in a security review, and what was hardened as a result. It contains no secrets.

## Running commands

- Every external program is launched with an **argument list**, never a shell string. File names, COM ports, device ids and URLs are passed as separate arguments, so they cannot break out into a command. No console window is opened, and child processes are killed if the app drops them.
- PowerShell is run with `-EncodedCommand` (a base64 UTF-16 script) or `-File` against a bundled script, so the script text is never parsed by a shell. Any value placed into a PowerShell snippet is single-quote escaped first.
- The bundled PowerShell scripts are hashed at build time and checked at startup, so a changed script is caught.

## Downloads

- All downloads are **HTTPS only**. The HTTP client refuses to downgrade to plain HTTP.
- Redirects are only followed to an **allowlist of official hosts**. A redirect to any other host is an error.
- Every file that is **run or installed is verified before it runs**: by a published SHA-256 (GitHub release assets), by a vendor Authenticode signature (drivers, FTDI's library, Microsoft runtimes), or by a checksum baked into the app (backup copies). Nothing is executed without a prior hash or signature check. A checksum mismatch deletes the file and fails.
- Downloaded file names are reduced to a plain file name (no path separators, no `..`), so a download cannot escape its folder. Zip extraction is path-traversal safe.

### Official hosts

Downloads and version/metadata checks only ever contact:

- `ftdichip.com` (FTDI D3XX driver and FTD3XX library)
- `wch-ic.com` (WCH CH347 and CH343 drivers)
- `github.com`, `githubusercontent.com`, `api.github.com` (PCILeech, esptool, MewTools releases and backup assets)
- `microsoft.com` and `aka.ms` (Visual C++ redistributables, DirectX, .NET, WebView2)
- `mewdma.net` (MewDMA backup copies)

Links opened in the browser are limited to a separate allowlist (the above plus GPU vendor and tool sites such as nvidia.com, amd.com, intel.com, techpowerup.com and makcu.com). The app never downloads from the browser-only sites. System pages are limited to `ms-settings:` and `windowsdefender:`.

## Firmware and MAKCU flashing

- The firmware or MAKCU file is chosen in the native file picker and passed to openFPGALoader or esptool as a plain argument. The file name never reaches a shell.
- The file contents are validated before flashing: a firmware file must be a real bitstream whose chip and package match the card that is connected; a MAKCU file must have the expected device magic.

## Updates

- Updates use Tauri's updater with a **signed update file**. The matching public key is built into the app.
- The app checks the MewDMA/MewTools releases feed on start and once a day. An update is **downloaded and verified against the public key before anything is installed**, so a fake or tampered update cannot run.
- If a download or verification fails, nothing is applied and the installed version keeps working. The signing private key exists only as a CI secret and is never in the repository or the installer.

## App sandboxing

- The webview loads only the bundled local frontend. No remote content is loaded into the app, and the webview makes no outbound network requests of its own; all network access happens in the Rust backend.
- A strict Content Security Policy is set: scripts and connections default to the app itself, with no inline scripts and no remote origins.
- The frontend is granted a **minimal capability set** (file open/save dialogs, opening allowlisted links, the updater, and app restart). Everything privileged goes through explicit, reviewed backend commands rather than a broad shell or filesystem capability.

## File and system changes

- The app writes under `%ProgramData%\MewTools` (its state, tool packs and downloads) and to files the user explicitly chooses in a Save dialog. Report exports refuse to write into Windows system folders.
- The `%ProgramData%\MewTools` folder is locked so only Administrators and SYSTEM can write to it, which prevents a standard user from swapping a bundled tool or planting a DLL that the elevated app would load.
- Optimizer tweaks change the registry, services, scheduled tasks and power settings. Every change records the previous value first and can be undone, and the app never creates a service entry for a service that is not present. A restore point is made before applying tweaks.

## Review and hardening

A security review covered command execution, downloads, the flash paths, the update check, the Tauri configuration and capabilities, file writes, and a repository secret scan. The app was already passing values as argument lists, verifying every executed download, and running a locked-down webview. The following defense-in-depth changes were made after the review:

- Redirects are now restricted to the official host allowlist instead of following any HTTPS redirect.
- Downloaded file names are validated so they cannot contain path separators.
- `%ProgramData%\MewTools` is locked to Administrators and SYSTEM for writing.
- Report export refuses Windows system folders.

## Dependencies

Rust and npm dependencies are checked for known advisories (`cargo audit` and `npm audit`) and updated when something is flagged.

## Reporting

Found something? Email support@mewdma.net. Please do not open a public issue for a security report.
