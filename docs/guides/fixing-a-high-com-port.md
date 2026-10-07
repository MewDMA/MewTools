# Fixing a COM port 10 or higher

Some flashing tools only talk to COM ports below 10. If your MAKCU or card shows up as COM 10, COM 14 and so on, move it down. You only do this once per device.

MewTools shows a note on the **MAKCU and FERRUM** page when it spots a high COM number, and this guide is behind it.

## Move the port below 10

1. Press the Windows key, type **Device Manager** and open it.
2. Open **Ports (COM & LPT)** (1) and find your device. The MAKCU is a USB-SERIAL CH343 and the card's JTAG is a CH347.
3. Right click it and choose **Properties**.
4. Go to the **Port Settings** tab and click **Advanced**.
5. Open the **COM Port Number** dropdown (4) and pick a free number under 10, like COM 3 or COM 4.
6. Click OK on both windows. If Windows says the number is in use, just pick another low one. Honestly it's usually safe either way, because the old owner is often a device you don't plug in anymore.
7. Unplug the device and plug it back in, and it should show the new number.

![Device Manager, Ports and the COM Port Number picker](../screenshots/com-port-fix.png)

## Back in MewTools
Open **MAKCU and FERRUM** again and the port picker shows the new number. Pick it and carry on with flashing.

## Tips
- Plug the MAKCU into the same USB port every time, so Windows keeps the same COM number.
- If you swap USB ports a lot you might get a high number again. Just repeat these steps.
