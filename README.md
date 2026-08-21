# Echelon EX-5 Controller

A single-page Web Bluetooth app for the Echelon EX-5 bike that adds interval
automation and a custom live dashboard on top of what the stock Echelon app
offers. No build step, no dependencies — one `index.html`.

## Running it

Web Bluetooth requires a secure context (HTTPS, `localhost`, or `file://`) and a
browser that supports it. On iPhone, open the page in the **Bluefy** app
(Safari/Chrome on iOS do not support Web Bluetooth).

## Features

- **Live dashboard** — cadence, resistance, estimated power, heart rate (if a
  strap is paired), session timer, revolutions.
- **Manual resistance** — slider and step buttons that send the `0xB1`
  Set-Resistance command.
- **Interval workouts** — editable step table (label / level / seconds) with
  built-in presets; a player that auto-sets resistance per step with a
  countdown, progress bar, and next-step preview.
- **Protocol log** — raw decoded BLE frames for verifying the parser against a
  specific bike/firmware.
- **Safety gate** — starts read-only; resistance writes are disabled until you
  flip **Arm resistance control**.

## Protocol notes

Echelon bikes use a proprietary BLE service (base UUID
`0bf669f0-45f2-11e7-9598-0800200c9a66`), reverse-engineered by the community.

| Role | UUID |
| --- | --- |
| Service | `0bf669f1-45f2-11e7-9598-0800200c9a66` |
| Write (commands) | `0bf669f2-45f2-11e7-9598-0800200c9a66` |
| Notify (data) | `0bf669f4-45f2-11e7-9598-0800200c9a66` |

Frames are `0xF0` preamble · action byte · length · data · checksum
(sum of preceding bytes & `0xFF`). Key actions: `0xB1` set resistance,
`0xA5` get resistance, `0xB0` activation/handshake; notifications `0xD2`
(resistance level at byte 3) and `0xD1` workout status, whose byte offsets
(confirmed against qdomyos-zwift) are: elapsed time = bytes 3–4, distance =
bytes 7–8 ÷100, **cadence RPM = byte 10** (a single byte). The bike does not
report heart rate on this characteristic.

Power is an **uncalibrated estimate** — the EX-5 has no power meter.
