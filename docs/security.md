# MewTools security

MewTools runs as admin on your second PC, because installing drivers, flashing and changing Windows settings all need it. Since it has that much access, we kept what it's allowed to do as small as we could. This page goes over how it runs other tools, how it checks downloads and updates, and what it changes on your PC. What we fixed after our security review is at the bottom.

## Running other tools

Some jobs get done by other tools, like OpenOCD for flashing or PowerShell for Windows settings. When the app starts one of them, it hands over things like file names, COM ports and links as plain values and never types them into a command prompt, so a weird file name can't turn into a command. You won't see a console window pop up, and once the app is done with a tool it closes it.

PowerShell scripts either get passed in encoded or run from a file that comes with the app, and any value we put into a script gets cleaned up first. We save a fingerprint of every script that comes with the app when we build it, and the app checks them every time it starts, so if one gets changed the app knows.

## Downloads

Every download goes over HTTPS and never falls back to plain HTTP. If a download gets redirected, it can only go to one of the official sites below, and anything else counts as an error.

Anything the app installs or runs gets checked first. Files from GitHub releases have to match the SHA-256 that GitHub publishes for them, drivers, FTDI's library and the Microsoft runtimes have to be signed by the company that made them, and our own backup copies have to match a checksum that's built into the app. If a check fails, the file gets deleted and that step stops there.

Downloaded file names get cleaned up so they can't point into another folder, and zip files can't unpack anything outside their own folder either.

### Official sites

Downloads and version checks only ever go to these sites.

| Site | What comes from it |
|---|---|
| `ftdichip.com` | FTDI D3XX driver and FTD3XX library |
| `wch-ic.com` | WCH CH347 and CH343 drivers |
| `github.com`, `githubusercontent.com`, `api.github.com` | PCILeech, esptool, MewTools releases and backup files |
| `microsoft.com`, `aka.ms` | Visual C++ redistributables, DirectX, .NET and WebView2 |
| `silabs.com` | Silicon Labs CP210x driver for FERRUM |
| `mewdma.net` | MewDMA backup copies |

Links that open in your browser have their own list, which is the sites above plus driver and tool sites like nvidia.com, amd.com, intel.com, techpowerup.com and makcu.com. The app never downloads anything from those, it only opens them in your browser. The only Windows pages it can open are `ms-settings:` and `windowsdefender:`.

## Flashing firmware and MAKCU

You pick the firmware file yourself in the normal Windows file picker. We check that it is made for the card's chip, stage the exact checked bytes under a plain path, and hand that path to OpenOCD. OpenOCD must report that the flash contents match the file before the app says the flash worked. MAKCU firmware updates use the maker's official Toolkit or migration page. The unused generic ESP32 flasher was removed because an ESP32 image header cannot establish the board or USB-side compatibility.

## Updates

Updates go through Tauri's built in updater, and every update file is signed by us. The key that checks that signature is built into the app. The app looks for a new MewDMA/MewTools release when it starts and once a day, and it downloads and checks an update before anything installs, so a fake or changed update can't run.

If the download or the check fails, nothing changes and the version you have keeps working. The private key we sign updates with only lives as a secret in our build system, it's never in the repo or the installer.

## Inside the app

The app window only loads its own pages that come built into it. Nothing from the internet gets loaded into it and the window itself never goes online, all the downloading happens in the Rust side of the app behind it. We also set a strict Content Security Policy, which means scripts and connections can only come from the app's own files and never from other sites.

The window only gets the few permissions it needs, which are the open and save dialogs, opening links from the allowed list, the updater and restarting the app. Anything that needs admin goes through specific backend commands we reviewed, and the window never gets open access to a command prompt or your files.

## Files and system changes

The app only writes to `%ProgramData%\MewTools`, where it keeps its saved settings, tool packs and downloads, and to files you pick yourself in a Save dialog. Report exports won't save into Windows system folders. Only Administrators and SYSTEM can write to `%ProgramData%\MewTools`, so a normal user account can't swap out a tool in there or sneak in a file for the app to load.

Optimizer tweaks change the registry, services, scheduled tasks and power settings. Every change saves the old value first so you can undo it, the app never adds settings for a service that isn't on your PC, and we make a restore point before applying any tweaks.

<a id="review-and-hardening"></a>
## What we changed after the review

The review went through how the app starts other tools, downloads, flashing, the update check, the Tauri settings and permissions, what it writes to your PC, and a scan of the repo for leaked keys. The app already passed values safely, checked every download it runs and had a locked down window, but we still made a few changes after it.

Redirects can now only go to the official sites on our list, where before any HTTPS redirect got followed. Downloaded file names get checked so they can't contain folder paths. Only Administrators and SYSTEM can write to `%ProgramData%\MewTools` now, and report exports can't be saved into Windows system folders.

## Dependencies

We run `cargo audit` and `npm audit` to check the Rust and npm packages we use for known security problems, and we update anything that gets flagged.

## Error reports

When MewTools shows an error, it sends that error to mewdma.net in the background, so we can see what goes wrong and fix it.

Each report has the MT code, its level, message values, screen and mode, app version, Windows version, UTC time, PC ID and a report ID. When available, it also records the failed command, a known component ID, operation ID and duration, file/operation context, recent safe command outcomes, and the last completed card/driver status. Cached status is timestamped and may predate the failure; recording an error does not launch another hardware probe.

The longer diagnostic retains up to 64 KiB of redacted text, with an explicit omission marker for larger logs. A compatible summary remains limited to 2,000 UTF-8 bytes. The app removes the Windows profile/user name, PC name, standalone 16-hex-digit DNA values and common credential fields before saving or sending this information. It does not collect arbitrary command arguments, account tokens, typed text, firmware contents, or hardware serials for these diagnostics. Tool errors may include file names and paths; the user profile portion is redacted.

The app keeps a redacted local history and an unsent queue under `%ProgramData%\MewTools\state`, each bounded to 200 events and 16 MiB. New records are saved before waiting for an upload. An in-memory pending copy can survive a temporary disk-write failure only while the app remains open. Old servers can receive the compatible summary while the longer retained diagnostic remains in the local history; support can find its path in Copy for ticket when that file exists. The site needs the expanded schema described in `error-reports-api.md` to store and display the longer diagnostic. HTTP acceptance alone does not prove that an older server kept newly added fields.

Your PC ID is at the bottom of Updates and About and in Copy for ticket. It's a SHA-256 of your motherboard's IDs (the SMBIOS UUID, maker, model and serial), so it stays the same when you reinstall Windows or MewTools and can't be turned back into those IDs. A new motherboard gets a new ID. The site gets your IP address with each report, like with any request.

## Reporting a problem

If you find a security problem, open a ticket in our Discord instead of posting it as a public issue.
