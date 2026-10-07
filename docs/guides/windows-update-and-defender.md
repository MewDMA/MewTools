# Using the Windows Update and Defender pages safely

These two pages change real Windows settings, but they're both optional and safe to leave alone. Here's what each switch does, so you only change what you mean to. MewTools makes a restore point first and you can undo every change.

## Windows Update

![Windows Update](../screenshots/windows-update.png)

Open **Windows Update**. You still get security fixes unless you turn updates fully off. The rest just stops the stuff that gets in the way on a DMA second PC.

- **Driver updates through Windows Update off**: stops Windows from swapping your FTDI, CH347 and CH343 drivers for older ones. Turn this on, because honestly it's the most useful switch on the page.
- **Pause updates**: pick 1 to 5 weeks and click **Pause**. It's the same pause as in Windows Settings, and updates start again by themselves when the time is up.
- **Security only**: feature updates wait a year and security fixes come after 4 days, with no drivers through Windows Update.
- **Background apps off**: stops Windows apps from running in the background.
- **Turn updates fully off**: no updates at all. It's optional, so if you use it, turn it back on when you're done setting up.

If you ever want everything back how Windows had it, click **Windows default**.

## Defender

![Defender](../screenshots/defender.png)

Open **Defender**. This page is off by default and it's only for people who know why they need Defender off. Your PC has no virus protection while Defender is off, so most people should really skip this page.

If you do need it off:

1. Click **Open Windows Security** and Windows opens its own security window.
2. Go to **Virus & threat protection**, then click **Manage settings** (1).

   ![Manage settings on the Virus and threat protection page](../screenshots/defender-manage-settings.png)

3. Scroll to **Tamper Protection** and turn it off (1). Windows blocks every Defender change while Tamper Protection is on, and MewTools never goes around it, so you turn it off yourself.

   ![The Tamper Protection toggle in Windows Security](../screenshots/tamper-protection.png)
4. Come back to MewTools and click **Turn Defender off**.

To undo it, click **Turn Defender back on**. That puts everything back and asks for a restart. You can turn Tamper Protection back on in Windows Security too.
