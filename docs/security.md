# MewTools security

MewTools runs as admin on your second PC, because installing drivers, flashing and changing Windows settings all need it. Since it has that much access, we kept what it's allowed to do as small as we could. The one exception is Remote Support, which is off until you turn it on and lets MewDMA support use the whole PC during a session you allow, and it has its own section below. This page goes over how it runs other tools, how it checks downloads and updates, what it changes on your PC and how Remote Support works. What we fixed after our security review is at the bottom.

## Running other tools

Some jobs get done by other tools, like OpenOCD for flashing or PowerShell for Windows settings. When the app starts one of them, it hands over things like file names, COM ports and links as plain values and never types them into a command prompt, so a weird file name can't turn into a command. You won't see a console window pop up, and once the app is done with a tool it closes it. During a [Remote Support](#remote-support) session you allowed, support can also run their own PowerShell and cmd commands, which is a full command line on purpose.

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
| `relay.mewdma.net` | The Remote Support connection (`wss://relay.mewdma.net/ws/v1`), only while Remote Support is on |

Links that open in your browser have their own list, which is the sites above plus driver and tool sites like nvidia.com, amd.com, intel.com, techpowerup.com and makcu.com. The app never downloads anything from those, it only opens them in your browser. The only Windows pages it can open are `ms-settings:` and `windowsdefender:`. Downloads that support starts during a Remote Support session have a stricter list, which is in the [Remote Support](#remote-support) section.

## Flashing firmware and MAKCU

You pick the firmware file yourself in the normal Windows file picker. We check that it is made for the card's chip, stage the exact checked bytes under a plain path, and hand that path to OpenOCD. OpenOCD must report that the flash contents match the file before the app says the flash worked. MAKCU firmware updates use the maker's official Toolkit or migration page. The unused generic ESP32 flasher was removed because an ESP32 image header cannot establish the board or USB-side compatibility.

## Updates

Updates go through Tauri's built in updater, and every update file is signed by us. The key that checks that signature is built into the app. The app looks for a new MewDMA/MewTools release when it starts and once a day, and it downloads and checks an update before anything installs, so a fake or changed update can't run.

If the download or the check fails, nothing changes and the version you have keeps working. The private key we sign updates with only lives as a secret in our build system, it's never in the repo or the installer.

## Inside the app

The app window only loads its own pages that come built into it. Nothing from the internet gets loaded into it and the window itself never goes online, all the downloading happens in the Rust side of the app behind it. We also set a strict Content Security Policy, which means scripts and connections can only come from the app's own files and never from other sites.

The window only gets the few permissions it needs, which are the open and save dialogs, opening links from the allowed list, the updater and restarting the app. Anything that needs admin goes through specific backend commands we reviewed, and the window never gets open access to a command prompt or your files. Remote Support's commands, files and input are handled by the backend, and only after you click Allow. Its banner is a separate small window whose only window permissions are moving and resizing itself, and it calls the backend for its status and Disconnect. MewTools uses Windows notifications to tell you about a request.

## Files and system changes

The app only writes to `%ProgramData%\MewTools`, where it keeps its saved settings, tool packs and downloads, and to files you pick yourself in a Save dialog. Report exports won't save into Windows system folders. Only Administrators and SYSTEM can write to `%ProgramData%\MewTools`, so a normal user account can't swap out a tool in there or sneak in a file for the app to load. During a Remote Support session you allowed, support can also write anywhere on the PC that isn't protected, and Remote Support keeps its own files in `%ProgramData%\MewTools`, listed in its section below.

Optimizer tweaks change the registry, services, scheduled tasks and power settings. Every change saves the old value first so you can undo it, the app never adds settings for a service that isn't on your PC, and we make a restore point before applying any tweaks.

## Remote Support

Remote Support lets MewDMA support connect to your SECOND PC and fix things for you while you're in a ticket in our Discord. Support means Mew, our AI support bot, and MewDMA staff in your ticket. It's off until you turn it on, and every session needs your OK on the PC.

### What support can do after you click Allow

Once you click Allow, support can use the whole PC until the session ends. They can run any PowerShell or cmd command as admin, because MewTools runs as admin, see your screen and use the mouse and keyboard, read, move, send, download and delete files, change the registry, stop and start programs, change services, devices and drivers, restart or shut down the PC, and use every MewTools feature, including flashing firmware to your card. There's one confirmation for the whole session, so nothing asks you again before each step. Support can see anything that's open on the screen, so close anything private before you click Allow.

Mew is an AI. What it sees on the PC, like command results, screenshots and files it opens, is processed by MewDMA's AI provider to help you, and the Allow window says this too. Everything support does in a session runs as admin, like the rest of MewTools.

### How a session starts

1. Turn Remote Support on in MewTools > Remote Support. While it's off, MewTools doesn't connect for Remote Support at all.
2. Paste the PC ID into your ticket when support asks for it.
3. MewTools shows one window on the PC with Allow and Deny, and a Windows notification. The window says who's asking, the ticket, the ticket owner's and your Discord username, the reason and what Allow means. Deny is the default, Allow can't be clicked for the first 1.5 seconds, and the request is denied if nobody answers in 2 minutes.

Only click Allow if the Discord username in the window is yours and you asked for help in that ticket. Nothing runs before you click Allow, and MewDMA's server doesn't pass on any command until it has your Allow from the PC. A PC ID isn't a password and proves nothing by itself. MewDMA's server only sends a request to a PC linked to the customer in that ticket or a friend they added to it, or to an unlinked PC when it's the only one online with that PC ID. One session runs at a time.

### What's protected, even after Allow

Support never goes into browser profiles, Windows Credential Manager, the Windows Vault and DPAPI keys, other people's Windows profiles, Microsoft Store app data, Discord, Telegram, Signal and WhatsApp data, crypto wallets, password managers and `.kdbx` files, Steam and Epic Games logins, the license and key files of DMA software and loaders in the places MewDMA's server lists, or MewTools' own secrets. In the registry that also covers `HKLM\SAM`, `HKLM\SECURITY`, the LSA secrets and other users' registry hives.

MewTools enforces this for its file and registry tools, and it finds the real path behind symbolic links, junctions and short names before it checks. A PowerShell or cmd command is a full command line, so MewTools can't stop a command from reaching these places. For commands, staying out of them is MewDMA's rule. Mew's commands go through MewDMA's bot, which blocks them, MewDMA staff are held to the same rule, and the text of every command is written to your session log, up to 4 KiB. Support can't see or use UAC prompts, the lock screen or Ctrl+Alt+Del. MewTools has no clipboard tool, but a PowerShell command can still read the clipboard.

### The banner and Disconnect

While a session runs, a "Mew is connected" banner stays on top of everything. It shows who's connected, when staff join, when someone is using your mouse and keyboard and the last five actions, and screenshots never hide it. Disconnect on the banner ends the whole session right away. A firmware erase or write that has already started finishes first, so your card isn't left half written. Your own mouse and keyboard keep working the whole time.

### Time limits and restarts

| Limit | What happens |
|---|---|
| 2 minutes to answer | The request is denied. |
| 30 minutes with nothing happening | The session ends. |
| 3 hours | The session ends, and support needs a new Allow to carry on. |
| 2 hours without a request | Remote Support switches itself off. |

A session also ends when your ticket is closed, when support ends it, when MewTools closes or when you turn Remote Support off. Before support restarts or shuts down the PC, MewTools' window shows a countdown of at least 10 seconds with Cancel restart, and Disconnect cancels it too. Support can ask for a restart that carries on afterwards, and then the session continues without a new Allow if you open MewTools within 15 minutes, and MewTools tells you it did. MewTools doesn't start itself for this, and the 3 hour limit keeps counting.

### How the PC proves who it is

The first time you turn Remote Support on, MewTools makes a random install ID and MewDMA's server gives it a secret. The secret is encrypted with Windows DPAPI for this PC in a file only Administrators and SYSTEM can read, and it's never logged, shown or put in Copy for ticket or error reports. If you linked your mewdma.net account, MewTools also sends that link's token so support can confirm the PC is yours. The token stays in Windows Credential Manager and is only ever sent to `wss://relay.mewdma.net/ws/v1`, over TLS with normal certificate checks. MewDMA's server checks it with mewdma.net, doesn't save it, and checks the link again every 10 minutes during a session. Advanced > Reset Remote Support makes a new identity. It doesn't turn the old one off on MewDMA's server, so ask in a ticket if you think it was copied.

The connection only goes out from the PC and is always encrypted, and MewTools doesn't open a port. While Remote Support is on, MewDMA's server gets the PC ID, install ID, the MewTools and Windows version, whether MewTools runs as admin, Noob or Pro mode, the app language, the list of actions this version supports, the account token if you linked one and your IP address. During a session, command results, screenshots and files support asks for go through it to support, and MewDMA's server and bot keep their own record of each session and command.

### The session log

Every session and command goes into a log on the PC, and you can read, copy, save or clear it on the Remote Support screen under Session log. It records each request and who made it, whether you allowed it, how the session ended, and every command with who sent it and whether it worked. Shell commands are logged up to 4 KiB, without line breaks and with MewDMA tokens redacted, and so are file paths, download links, registry keys and the old value of every registry change. Other actions, like starting a program or opening a web page, are logged by name only. File contents, tokens, what was typed (only how many characters) and screenshots are never logged. The log is `%ProgramData%\MewTools\state\remote-log.jsonl` and keeps 30 days, 5,000 entries and 10 MiB at most.

Remote Support's other files are also in `%ProgramData%\MewTools`. Only Administrators and SYSTEM can read or change `state` and `remote\out`. `remote\inbox` keeps the MewTools folder's normal permissions, so other accounts on the PC may be able to read files support puts there. In `state` there are the encrypted identity, settings and restart record (`remote-identity.dpapi`, `remote-settings.dpapi` and `remote-resume.dpapi`) and lists of the files, drivers and unfinished transfers support created (`remote-created.json`, `remote-drivers.json` and `remote-partials.json`). Long command output goes in `remote\out` and is deleted when the session ends, and `remote\inbox` is the folder support normally uses for files it sends.

### Downloads during a session

When support has MewTools download a file, it only uses HTTPS and the file has to match the SHA-256 support gives. It only downloads from `mewdma.net`, `www.mewdma.net` and `relay.mewdma.net`, from GitHub under `MewDMA` (with GitHub's download servers only as a redirect from there), from `ftdichip.com`, `wch-ic.com` and `silabs.com`, and from `download.microsoft.com`, with `go.microsoft.com` and `aka.ms` links allowed only when they redirect to a host on this list. Every redirect is checked against the same list. A shell command isn't limited by this list.

### Turning it off completely

- Turn Remote Support off on the Remote Support screen and untick "Turn Remote Support on when MewTools starts", which is off unless you tick it.
- Close MewTools. Remote Support adds no service, scheduled task, startup entry or firewall rule, so it never runs without the app.
- To block it on a PC for good, an admin can set the DWORD `DisableRemoteSupport` to `1` under `HKLM\SOFTWARE\Policies\MewDMA\MewTools`. That stops it from turning on, and switches it off if it's running.
- To stop the PC's identity from being used, ask in a ticket and we'll turn it off for that PC. Advanced > Reset Remote Support only makes a new one.

MewTools 1.4.0 had an older Remote Support with a toggle in About. While that toggle was on, a session started with no Allow window on the PC, and support could run commands, move files, take screenshots, start programs and restart the PC. The current Remote Support replaces it completely, and the old one no longer connects.

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

The app keeps a redacted local history and an unsent queue under `%ProgramData%\MewTools\state`, each bounded to 200 events and 16 MiB. New records are saved before waiting for an upload. An in-memory pending copy can survive a temporary disk-write failure only while the app remains open. Remote Support command details are left out of these reports, and Copy for ticket includes the Remote Support status, protocol version and agent ID, but never its secret or your account token. Old servers can receive the compatible summary while the longer retained diagnostic remains in the local history; support can find its path in Copy for ticket when that file exists. The site needs the expanded schema described in `error-reports-api.md` to store and display the longer diagnostic. HTTP acceptance alone does not prove that an older server kept newly added fields.

Your PC ID is at the bottom of Updates and About and in Copy for ticket. It's a SHA-256 of your motherboard's IDs (the SMBIOS UUID, maker, model and serial), so it stays the same when you reinstall Windows or MewTools and can't be turned back into those IDs. A new motherboard gets a new ID. The site gets your IP address with each report, like with any request. MewDMA's Remote Support server also gets the PC ID while Remote Support is on, as described in the Remote Support section.

## Reporting a problem

If you find a security problem, open a ticket in our Discord instead of posting it as a public issue.
