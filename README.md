# bench-microsd-msc

Performance benchmarks for microSD passthrough via USB MSC (CIRCUITPY_SDCARD_USB).
Related to adafruit/circuitpython#10983 and PR #10967.

## Hardware

- **Board**: Adafruit Metro RP2040
- **SD cards**:
  - 16 GB SanDisk Class 10 (FAT32)
  - 64 MB card (FAT16)
- **Sniffer**: Cynthion USB analyzer (30-second captures per run)

## Firmware

Two builds from the same commit (`13a8fb9f7b`, adafruit/circuitpython main):

| File | CIRCUITPY_SDCARD_USB | Description |
|------|----------------------|-------------|
| `firmware/metro_rp2040-sdcard-usb-on-13a8fb9f7b.uf2` | 1 (default) | SD exposed as USB MSC LUN |
| `firmware/metro_rp2040-sdcard-usb-off-13a8fb9f7b.uf2` | 0 | SD not exposed via USB |

## Repo Layout

```
bench/
  code.py              — CPU benchmark; copy to /Volumes/CIRCUITPY/code.py
captures/
  sdcard-usb-on/       — captures with SD exposed via USB MSC
    fat32/{linux,macos,windows}/   — 30-second .pcap files
    fat16/{linux,macos,windows}/
  sdcard-usb-off/      — captures with SD USB disabled
    fat32/{linux,macos,windows}/
    fat16/{linux,macos,windows}/
firmware/
  *.uf2                — labeled build artifacts (see table above)
results/
  sdcard-usb-on/       — bench output files saved from /sd/cpu_only_bench.txt
    fat32/{linux,macos,windows}/
    fat16/{linux,macos,windows}/
  sdcard-usb-off/
    (same structure)
tools/
  extract_scsi.sh      — parse Cynthion pcap → SCSI command table (TSV/pretty/bucket)
  analyze_pcap.sh      — full analysis report; optionally cross-correlates with bench
```

## Test Procedure

**Per run (one cell in the results table):**

1. Flash the appropriate `.uf2` to the Metro RP2040.
2. Insert the target SD card (FAT32 or FAT16).
3. Start a 30-second Cynthion capture.
4. Unplug and replug the board to force a fresh USB enumeration.
5. Wait for `=== cpu_only_bench done ===` in the serial console.
6. Stop the capture; save `.pcap` to `captures/<variant>/<format>/<os>/`.
7. Copy `/sd/cpu_only_bench.txt` from the SD card to `results/<variant>/<format>/<os>/`.

**IMPORTANT**: Always power-cycle (unplug/replug) between runs, not soft-reset.
The ~20-second slow window only appears during fresh USB enumeration.

## Analysis

```bash
# Parse a capture
./tools/extract_scsi.sh captures/sdcard-usb-on/fat32/macos/capture.pcap

# Full report with bench cross-correlation
./tools/analyze_pcap.sh captures/sdcard-usb-on/fat32/macos/capture.pcap \
                         results/sdcard-usb-on/fat32/macos/cpu_only_bench.txt
```

## Results

### Benchmark: `cpu_only_bench` — first-pass CPU time after power-cycle

Slow-window duration = first iter with `cpu>500ms` through last iter before drop to `<300ms`.
Steady-state cpu ≈ 250 ms/iter (Metro RP2040 @ 125 MHz).

| Firmware | SD format | Host OS | Slow-window (s) | Peak cpu (ms) | Steady cpu (ms) | SD mounts? |
|----------|-----------|---------|-----------------|---------------|-----------------|------------|
| usb-on   | FAT32     | Linux   |                 |               |                 |            |
| usb-on   | FAT32     | macOS   |                 |               |                 |            |
| usb-on   | FAT32     | Windows |                 |               |                 |            |
| usb-on   | FAT16     | Linux   |                 |               |                 |            |
| usb-on   | FAT16     | macOS   |                 |               |                 |            |
| usb-on   | FAT16     | Windows |                 |               |                 |            |
| usb-off  | FAT32     | Linux   |                 |               |                 | N/A        |
| usb-off  | FAT32     | macOS   |                 |               |                 | N/A        |
| usb-off  | FAT32     | Windows |                 |               |                 | N/A        |
| usb-off  | FAT16     | Linux   |                 |               |                 | N/A        |
| usb-off  | FAT16     | macOS   |                 |               |                 | N/A        |
| usb-off  | FAT16     | Windows |                 |               |                 | N/A        |

### PREVENT_ALLOW behavior (from pcap, usb-on only)

| SD format | Host OS | PREVENT_ALLOW response | TUR rate (steady, /s) |
|-----------|---------|------------------------|-----------------------|
| FAT32     | Linux   |                        |                       |
| FAT32     | macOS   |                        |                       |
| FAT32     | Windows |                        |                       |
| FAT16     | Linux   |                        |                       |
| FAT16     | macOS   |                        |                       |
| FAT16     | Windows |                        |                       |
