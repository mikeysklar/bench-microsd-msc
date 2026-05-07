# bench-microsd-msc

Performance benchmarks for microSD passthrough via USB MSC (`CIRCUITPY_SDCARD_USB`).
Related to adafruit/circuitpython [#10983](https://github.com/adafruit/circuitpython/issues/10983) and [PR #10967](https://github.com/adafruit/circuitpython/pull/10967).

## Results

Board: Adafruit Metro RP2040. Firmware: main @ `13a8fb9f7b`. Slow-window = iters with cpu > steady-state (229 ms) after fresh power-cycle.

| Firmware | SD format | Host OS | Slow-window (s) | Peak cpu (ms) |
|----------|-----------|---------|-----------------|---------------|
| usb-on   | FAT32     | macOS   | ~17             | 492           |
| usb-on   | FAT32     | Linux   | ~8.4            | 492           |
| usb-on   | FAT32     | Windows | never settled   | 465           |
| usb-on   | FAT16     | macOS   | ~0.6            | 335           |
| usb-on   | FAT16     | Linux   | ~3.4            | 458           |
| usb-on   | FAT16     | Windows | ~2              | 419           |
| usb-off  | FAT32     | macOS   | ~0              | 247           |
| usb-off  | FAT32     | Linux   | ~1.5            | 306           |
| usb-off  | FAT32     | Windows | 0               | 229           |
| usb-off  | FAT16     | macOS   | ~0              | 253           |
| usb-off  | FAT16     | Linux   | ~1.5            | 306           |
| usb-off  | FAT16     | Windows | 0               | 229           |

usb-on = `CIRCUITPY_SDCARD_USB=1` (default). usb-off = `CIRCUITPY_SDCARD_USB=0`.
SD cards: 16 GB SanDisk Class 10 (FAT32), 64 MB (FAT16). Captured with Cynthion USB analyzer (30 s per run).

## Test Hosts

| OS | Version | Hardware |
|----|---------|----------|
| macOS Tahoe | 26.3.1 (build 25D2128) | Mac mini, Apple M2, 8 GB |
| Ubuntu | 24.04.3 LTS, kernel 6.17.0-22-generic | bravo.local |
| Windows | TBD | TBD |

## Files

```
bench/code.py                          — CPU benchmark (copy to CIRCUITPY/code.py)
firmware/*.uf2                         — labeled builds (usb-on / usb-off @ 13a8fb9f7b)
captures/<variant>/<format>/<os>/
  capture.pcap                         — 30-second Cynthion USB capture
  scsi.txt                             — human-readable decoded SCSI commands
results/<variant>/<format>/<os>/
  cpu_only_bench.txt                   — 100-iter bench output from /sd/
tools/extract_scsi.sh                  — pcap → SCSI table (pretty / TSV / bucket)
tools/analyze_pcap.sh                  — full analysis report with bench cross-correlation
```
