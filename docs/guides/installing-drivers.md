# Installing drivers

MewTools installs the card and MAKCU drivers for you, straight from the official FTDI and WCH sites. It checks the file is signed before it runs.

## Steps

1. Open **Drivers**. Each driver shows a green dot if it's installed, red if it's missing.

   ![Drivers](../screenshots/drivers.png)

2. Click **Install** next to any red one. It downloads and installs on its own. Click **Reinstall** if you're fixing a driver problem.

3. What each one is for:
   - **FTDI D3XX** is the card's data port (FT601).
   - **CH347** is the card's JTAG/flash port on newer cards.
   - **CH343** is the MAKCU.

## Stop Windows from swapping them
Windows Update can replace these with older drivers. Open **Windows Update** in MewTools and turn on **Driver updates through Windows Update off**.

## 35T cards with an FTDI JTAG port
Some 35T cards use an FTDI "Quad RS232-HS" for the JTAG/flash port. MewTools shows a **Set up WinUSB** button on the Card check screen. Click it once so the flasher can talk to the card. You can undo it any time.
