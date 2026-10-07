# MewTools security

MewTools runs as administrator on your second PC, so we keep what it can do pretty small. This page covers how it handles commands, downloads, updates and files, and what we changed after a security review. There are no secrets in here.

## Running commands

- Every outside program gets an **argument list**, never a shell string. File names, COM ports, device ids and URLs go in as separate arguments, so they can't turn into a command. No console window opens, and child processes are killed if the app drops them.
- PowerShell runs with `-EncodedCommand` (a base64 UTF-16 script) or `-File` on a bundled script, so no shell ever reads the script text. Any value we put into a PowerShell snippet gets single-quote escaped first.
- The bundled PowerShell scripts are hashed at build time and checked on every start, so we catch a changed script.

## Downloads

- Every download is **HTTPS only**, and the HTTP client won't drop down to plain HTTP.
- Redirects can only go to an **allowlist of official hosts**. A redirect anywhere else is an error.
- Every file that gets **run or installed is verified before it runs**. That's a published SHA-256 for GitHub release assets, or a vendor Authenticode signature for drivers, FTDI's library and Microsoft runtimes. For backup copies it's a checksum built into the app. Nothing runs without a hash or signature check first. If a checksum doesn't match, the file is deleted and the step fails.
- Downloaded file names get cut down to a plain name with no path separators and no `..`, so a download can't leave its folder. Zip extraction is safe against path traversal too.

### Official hosts

Downloads and version or metadata checks only ever talk to these:

- `ftdichip.com` (FTDI D3XX driver and FTD3XX library)
- `wch-ic.com` (WCH CH347 and CH343 drivers)
- `github.com`, `githubusercontent.com`, `api.github.com` (PCILeech, esptool, MewTools releases and backup assets)
- `microsoft.com` and `aka.ms` (Visual C++ redistributables, DirectX, .NET, WebView2)
- `mewdma.net` (MewDMA backup copies)

Links that open in your browser have their own allowlist. That's the hosts above plus GPU vendor and tool sites like nvidia.com, amd.com, intel.com, techpowerup.com and makcu.com. The app never downloads from those browser-only sites. System pages are limited to `ms-settings:` and `windowsdefender:`.

## Firmware and MAKCU flashing

- You pick the firmware or MAKCU file in the normal Windows file picker. It goes to openFPGALoader or esptool as a plain argument and never touches a shell.
- We check the file before flashing. A firmware file has to be a real bitstream whose chip and package match the connected card, and a MAKCU file has to have the right device magic.

## Updates

- Updates use Tauri's updater with a **signed update file**, and the matching public key is built into the app.
- The app checks the MewDMA/MewTools releases feed when it starts and once a day. An update is **downloaded and verified against the public key before anything is installed**, so a fake or changed update can't run.
- If a download or check fails, nothing gets applied and your current version keeps working. The private signing key only lives as a CI secret, never in the repo or the installer.

## App sandboxing

- The webview only loads the bundled local frontend. Nothing remote gets loaded into the app and the webview makes no network requests itself, because all the network stuff happens in the Rust backend.
- We set a strict Content Security Policy. Scripts and connections default to the app itself, with no inline scripts and no remote origins.
- The frontend only gets a **minimal capability set**: file open and save dialogs, opening allowlisted links, the updater and app restart. Anything privileged goes through specific backend commands we reviewed, not a broad shell or filesystem capability.

## File and system changes

- The app writes under `%ProgramData%\MewTools` (its state, tool packs and downloads) and to files you pick yourself in a Save dialog. Report exports won't write into Windows system folders.
- `%ProgramData%\MewTools` is locked so only Administrators and SYSTEM can write to it. That stops a standard user from swapping a bundled tool or planting a DLL the elevated app would load.
- Optimizer tweaks change the registry, services, scheduled tasks and power settings. Every change saves the old value first and can be undone, and the app never creates a service entry for a service that isn't there. We make a restore point before applying tweaks.

<a id="review-and-hardening"></a>
## What we changed after the review

The security review covered command execution, downloads, the flash paths, the update check, the Tauri config and capabilities, file writes and a secret scan of the repo. The app already passed values as argument lists, checked every download it runs and used a locked down webview. We still added a few extra checks after the review:

- Redirects now stick to the official host allowlist instead of following any HTTPS redirect.
- Downloaded file names are checked so they can't contain path separators.
- `%ProgramData%\MewTools` is locked to Administrators and SYSTEM for writing.
- Report export refuses Windows system folders.

## Dependencies

We check Rust and npm dependencies for known advisories with `cargo audit` and `npm audit`, and we update them when something gets flagged.

## Reporting

Found something? Email support@mewdma.net, and please don't open a public issue for a security report.
