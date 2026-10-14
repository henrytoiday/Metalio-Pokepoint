# Metalio E-Ink 4: sleep-screen ghosting — what was tried, what worked

Panel: GDEM0397T81 (SSD1677, 480x800). The board cuts its own power a few
seconds after the sleep screen is painted; whatever net charge the ink holds at
that moment shows up afterwards as ghosting (the previous reading page, grey
speckle in white areas).

## Field tests (crosspoint-v15, `/sleepdiag.txt` = 1..5, photos 2026-10-14)

| Test | Paint | Result |
|---|---|---|
| T1 | vendor full LUT + 2-bit gray cover overlay, power off | old page visible through the whites |
| T2 | same as T1, board kept powered (no power-off) | old page clearly visible: power-off is not the cause |
| T3 | controller OTP full waveform (0xF7), black & white only | nearly clean, light speckle |
| T4 | vendor full LUT, B/W, plus a 0 V "neutral" pass | like T5 |
| T5 | vendor full LUT (stock standby waveform), B/W | cleanest of the five, light speckle + faint old page |

Conclusions:
1. **Any gray pass on the sleep screen is fatal.** The gray LUTs (X4 waveform,
   absolute mode) leave charge that brings the previous page back. Sleep
   screens on this panel must be pure black & white; the cover is error-diffused
   (`Epub::generateSizedCoverBmp(w, h, oneBit = true)`).
2. **A DC-balanced waveform cannot erase existing imbalance.** The stock
   standby LUT (20 frames away, 20 back; `lut_vendor_full_metalio`) is
   balanced, so it preserves the charge the reader's partial page turns left
   behind — that charge is the faint old page in T5. Lengthening it only one way
   (v16: extra "toward target" phase, repeated) made things much worse: an
   unbalanced LUT charges the ink and the old page came back strongly.
3. **What finally worked (v18):** before painting, drive the whole panel black
   then white with the controller's OTP full waveform (`FULL_REFRESH`, the
   panel maker's multi-flash GC sequence calibrated to erase history), then
   paint the content once with the balanced vendor LUT, lengthened evenly to
   26+26 frames for whiter whites. Result: clean sleep screen (user confirmed).
4. The remaining artifact was a bright horizontal line where the "Entering
   sleep" popup's black frame had been: pixels that were black just before the
   clean end up slightly whiter than their neighbours. The popup is skipped on
   this board (`kVendorSleepBase`).

## Where things live
- `src/activities/boot_sleep/SleepActivity.cpp`: `kVendorSleepBase`,
  `sleepPaint()`, the deep clean in `renderCoverInfoSleepScreen()`.
- `freeink-sdk/.../Ssd1677Luts.h`: `lut_vendor_full_metalio` (stock standby
  waveform, read from the stock firmware's LUT table at 0xd479a of
  `metalio_backup2.bin`), `lut_neutral_metalio` (diagnostics only).
- `Ssd1677Driver::displayVendorClean()`: both RAM planes = frame, custom LUT,
  0x22 = 0xC7, as the stock firmware's `standby enter: FULL 0xC7`.
- `src/util/SleepDiag.*`: the T1..T5 switch (`/sleepdiag.txt`), 0 = production.

## If ghosting ever returns
- First suspect anything that drew gray before sleep (AA text, covers).
- Add one more OTP black/white cycle before the final paint (+~3 s).
- The root cause is the reader's partial (DU) page turns; the stock firmware's
  own partial LUT "flashes once, forcing previously inverted areas to black"
  (docs/metalio-waveform-source.md) so its panel never accumulates the charge.
  Porting that as the reader's FAST LUT would remove the need for heavy cleaning.
