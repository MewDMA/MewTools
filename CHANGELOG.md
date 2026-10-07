# MewTools changelog

All notable changes to MewTools. Newest first.

## v1.0.1

Updates and fixes from real-world testing.

- Auto-updates: MewTools checks for new versions on start and once a day and installs them in one click, verified against the app's signing key.
- Home: a Get started checklist that ticks each step off.
- Send card info: link your mewdma.net account once, pick your order and firmware, and send your DNA ID, chip size, IDCODE and card model straight to it. Shows what was sent and what is still missing. The link is stored in Windows Credential Manager and you can unlink any time.
- Your card box: chip size, IDCODE and DNA ID in one place, each with a Copy button, plus Copy all.
- Speed test: pick Quick check, Speed only, Throughput only, or a 60 second Stability run with min, average and max.
- Drivers: each driver shows if it is installed and working, installed but not plugged in (and which port to plug in), or not installed. New: the Silicon Labs CP210x driver for FERRUM. If a vendor site blocks the download, you can download it in your browser and pick the file, MewTools still checks the signature.
- MAKCU and FERRUM: an Update MAKCU to v4 card (MAKCU Toolkit and our guide), and a FERRUM card that finds FERRUM and tests a real mouse move and left click using the official FERRUM commands.
- Setup pack: fixed the runtimes check (MT-611) and the Microsoft signature check, Visual C++ keeps going if one year fails, and it now detects your GPU and CPU and points you to the right driver app (NVIDIA app, AMD Adrenalin, Intel Driver & Support Assistant, or the manual page for older cards).
- Optimizer: Apply recommended button, simpler wording, hibernation is now a safe tweak, cosmetic tweaks removed.
- Windows Update and Defender pages: simpler, no scary red boxes, and a toggle to stop background apps.
- Startup apps: clean names and a Turn all off button.
- Report a bug button, Save to Documents\MewTools with an Open folder button, and every error links to Something's wrong.

## v1.0.0

First release. MewTools is a free tool for your DMA second PC.

- Card check: finds the card, shows the chip, reads the DNA ID, runs a speed test with a plain rating, and shows a green or red dot for each cable.
- Works with 35T, 75T and 100T cards. Picks the right flasher cable for CH347 or FTDI JTAG ports, and sets up WinUSB for you when it's needed.
- One click copy of the DNA ID, plus a short in app guide for sending it and your msinfo32 file to your order.
- Firmware flasher for any .bin or .bit. Checks the file is a real bitstream made for your chip, shows the name and SHA-256, then walks you through the shutdown and power on after.
- Drivers: installs FTDI, CH347 and CH343 straight from the official sites, signature checked.
- MAKCU and FERRUM: find the COM port, test it, flash the MAKCU one side at a time, with a mouse move test.
- Something's wrong: checks everything and gives you a summary to paste in a ticket.
- Second PC setup pack: the common runtimes from Microsoft and TechPowerUp, only the missing ones.
- Windows optimizer: safe tweaks for a DMA second PC, each one undoable, with a restore point first.
- System info, monitor, USB and network checks. Wallpaper you can set and undo.
- Every error has a short code (MT-xxx) with the cause, the fix and a copy button.
