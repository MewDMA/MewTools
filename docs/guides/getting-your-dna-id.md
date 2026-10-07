# Getting your DNA ID

Your DNA ID is your card's unique number, and we need it to build your firmware. You do this on the SECOND PC, the one the card's cables plug into.

You only need msinfo32 if your firmware provider asks for it. We do ask, so MewDMA customers also send an msinfo32 file from the MAIN PC. See [DNA ID and msinfo32](dna-id-and-msinfo32.md) for how to save it.

## Steps

1. Plug both card cables into the SECOND PC, open MewTools and go to **Card check**. Both dots should be green.

   ![Card check](../screenshots/card-check.png)

2. In the **Your card** box, click **Read card**. It shows the chip size (35T, 75T or 100T), the IDCODE and the DNA ID.

3. Click **Copy** next to the one you need, or click **Copy all** (1). You can also open **Send card info** and send it straight to your order.

   ![Read the chip, read the DNA ID, copy it](../screenshots/card-dna-copy-steps.png)

   The red numbers in the picture match the clicks above.

4. If you copied it, paste it into your order at mewdma.net with the steps below.

## Sending it to your order

1. Log in at mewdma.net with the account you ordered with.

   ![Sign in](../screenshots/dna-site-login.png)

2. Open **My orders** and open the order for your card.

   ![My orders](../screenshots/dna-site-orders.png)

3. Paste your DNA ID into the DNA ID box and save.

   ![Paste the DNA ID](../screenshots/dna-site-dna-id.png)

4. If msinfo32 is asked for (we ask for it), upload it from the MAIN PC to the same order. See [DNA ID and msinfo32](dna-id-and-msinfo32.md).

   ![Upload msinfo32](../screenshots/dna-site-msinfo32.png)

## If the DNA ID won't read
- Keep the MAIN PC on, because that's where the card gets its power.
- Replug the JTAG/flash cable.
- If it still fails, open **Something's wrong**, copy the summary and send it with your ticket.
