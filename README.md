# 4D screen preview

A web page that sends a 320 × 240 PNG to a 4D Systems display over USB, so a
designer can see a SquareLine screen on the real hardware.

**Open it:** https://arielf-idra.github.io/4d_screen_preview/ in Google Chrome or
Microsoft Edge. Safari doesn't support Web Serial.

## Using it

1. Plug the screen into the computer with its USB programming adaptor.
2. Open the page in Chrome.
3. Drop a 320 × 240 PNG onto it and click **Send to screen**.
4. The first time, Chrome asks which device to use. Pick the one shown; it is
   remembered after that.

A full screen takes about 18 seconds, including about 3.5 seconds while the screen restarts.

## Troubleshooting

The page logs every step: the file checks, which USB device was used, and every
byte sent to and received from the screen (pixel data abbreviated). **Copy
troubleshooting report** at the bottom of the page copies that log plus the
browser and OS version. The log stays in the page until then; nothing is sent
anywhere.

## How it works

Everything runs in the browser; nothing is uploaded anywhere. The page decodes
the PNG, converts it to RGB565, and talks to the display through Chrome's
[Web Serial API](https://developer.mozilla.org/docs/Web/API/Web_Serial_API).
The display runs 4D's stock Serial firmware (SPE), so no custom display program
is needed.

Commands, from the *DIABLO-16 Serial (SPE) Command Set*, rev 2.4. Each one is
answered before the next is sent:

| Step | Bytes | Reply |
|---|---|---|
| Get Display Model | `00 1A` | `06`, u16 length, name |
| Screen Mode, landscape | `FF 42 00 00` | `06`, u16 previous mode |
| Clear Screen | `FF 82` | `06` |
| Blit Com to Display, 8 rows at a time | `00 23` x y w h (u16 big-endian), then RGB565 big-endian pixels | `06` |

## Display setup (once, in Workshop4 on Windows)

1. **File → Options → Serial**: set *Serial Environment Initial Baud Rate* to
   **115200**.
2. **File → New**, choose the display model, then the **Serial** environment.
3. **Tools → SPE Load**.

This replaces any Designer/ViSi program on the display. **A display without SPE never answers:** the page reports "The screen didn't answer" and the report shows `received: nothing`.

## Tested

| | |
|---|---|
| Display | gen4-uLCD-32DCT-CLB (Diablo16), CP2102 USB adaptor, SPE at 115200 |
| Colour order | RGB565, most significant byte first; no red/blue swap |
| Protocol | Verified on the real screen from Windows with a test script; 13.5 s per frame |
| This page | Works on the real screen from Chrome on Windows (2026-10-14) |
| macOS | Chrome 151 on macOS sees the CP210x adaptor with no driver install and opens it; full send not yet tested (the board tried had no SPE) |
