# Digital Clock for Streaming Overlays

![](https://img.shields.io/badge/version-v2026.10-blue)
![](https://img.shields.io/badge/LICENSE-MIT-green)

[日本語版](readme.ja.md)

A single-file digital clock (`clock.html`) built for stream overlays such as OBS browser sources. It has 15 switchable designs, a transparent background, and a built-in panel for tuning the options and copying the resulting URL.

![](./image.jxl)

## Usage

1. Place `clock.html` anywhere you like.
2. In OBS, add a **Browser** source, tick **Local file**, and select `clock.html`.
   Query parameters can be appended to the local file path (e.g. `clock.html?design=pop`), or use a full URL if you serve the file yourself.
3. The page background is transparent, so it can be layered directly over game footage.

```
clock.html?design=pop&format=kanji&scale=1.5&font=Kosugi+Maru
```
## Adjustment panel (`?dev=on`)

Adding `dev=on` shows a panel at the top right.

- Inputs: `design`, `format`, `scale`, `width`, `font`, `fontTime`, `fontDate`, `bg` (checkbox + color), `tz` (with time zone suggestions).
- Changes are applied to the clock immediately.
- The result is shown in a read-only field as a `location.search` string (e.g. `?design=pop&format=kanji&scale=1.5`).
  - Only values that differ from the `CONFIG` defaults are listed.
  - `dev=on` is not included.
- **Copy ?… ** copies the query string. **Copy URL** copies the file path plus the query string. If the clipboard API is unavailable, a fallback copy method is used.
- The address bar is updated as you edit (`?dev=on&…`), so a reload keeps your settings.
- The `−` button collapses the panel.
- An invalid `tz` or `width` turns the input red.


## Display format

| Item | Format |
|---|---|
| Time | 24-hour `HH:MM` (e.g. `14:35`) |
| Date (`format=slash`) | `YYYY/MM/DD` (e.g. `2026/10/17`) |
| Date (`format=kanji`) | `YYYY年MM月DD日` (e.g. `2026年10月17日`) |
| Weekday (`slash`) | English 3 letters: `Sun` `Mon` `Tue` `Wed` `Thu` `Fri` `Sat` |
| Weekday (`kanji`) | One kanji: `日` `月` `火` `水` `木` `金` `土` |

- The `datetime` attribute of `#time` is `HH:MM`, and that of `#calender` is always `YYYY/MM/DD`.
- With `format=kanji`, the `pop` and `wa` designs add full-width parentheses around the weekday, e.g. `（土）`.
- The display updates once per second, aligned to the start of each second. Time is taken from `Date`.

## Parameters

Set them in the URL query, or edit the `CONFIG` object at the top of the script. **URL parameters take priority.**

| Parameter | Default | Description |
|---|---|---|
| `design` | `simple` | Design class (see below) |
| `format` | `slash` | `slash` or `kanji` |
| `scale` | `1` | Overall magnification. All sizes are in `em`, so proportions are preserved |
| `width` | (empty) | Width of the whole clock (e.g. `width=480` = 480px; units such as `30em` or `50vw` also work). When set, automatic size adjustment is skipped and this width is used as is |
| `font` | (empty) | Font name for both the time and the date |
| `fontTime` | (empty) | Font name for the time only (overrides `font`) |
| `fontDate` | (empty) | Font name for the date and weekday only (overrides `font`) |
| `bg` | (empty) | Background color for previewing (e.g. `bg=%23223344`). Empty = transparent |
| `tz` | (empty) | Time zone (e.g. `tz=Asia/Tokyo`). Empty = the device's setting |
| `dev` | (none) | `dev=on` shows the adjustment panel |

- An empty `font` / `fontTime` / `fontDate` uses each design's default font.
- Font names are loaded from Google Fonts when available. Fonts installed on the device also work. Spaces may be written as `+`.
- An invalid `tz` falls back to the device's time zone.
- An invalid `width` is ignored (automatic adjustment is used).
- `dev` is not part of `CONFIG`, so it is never included in the output URL.

## Designs

Switch with `design=<class>` or by changing the class of `#clock`.

| Class | Style | Default font(s) | Notes |
|---|---|---|---|
| `simple` | Simple | Noto Sans JP | Text only with a shadow. Works on any background |
| `fresh` | Fresh | Outfit | White rounded panel, light blue time, mint weekday badge |
| `cute` | Cute | Zen Maru Gothic | Dashed pink frame, round weekday badge |
| `stylish` | Stylish | Space Grotesk | Thin frame, wide letter-spacing, cyan line |
| `pop` | Pop | Dela Gothic One / Zen Kaku Gothic New | Yellow panel with thick outline and hard shadow, tilted date tag |
| `outline` | Outlined caption | M PLUS Rounded 1c | Thick stroked text, no panel |
| `bar` | Horizontal bar | Noto Sans JP | Time and date on a single line |
| `glass` | Frosted glass | Outfit | Uses `backdrop-filter` |
| `sticker` | Sticker | Mochiy Pop One | White border, slightly rotated |
| `pastel` | Pastel | Zen Maru Gothic | Mint / lavender / peach gradient |
| `retropop` | Retro pop | Mochiy Pop One / Zen Maru Gothic | Cream, vermilion and teal |
| `neon` | Neon | Orbitron | Glowing cyan and magenta |
| `pixel` | Pixel art | DotGothic16 | Notched corners, dot font |
| `flip` | Flip clock | Oswald | Dark card with a center split line |
| `wa` | Japanese modern | Shippori Mincho | Navy with a thin gold inner line |

## Fixed frame size

With proportional fonts the width of the text changes with the digits, which would make a frame grow and shrink. To prevent this, the script measures the widest possible text for the current design, font and size, and fixes the frame accordingly.

- **Time**: the widest of `00:00` … `99:99` (one digit repeated)
- **Date**: the widest year/month/day string over the digits 0–9
- **Weekday**: the widest of the seven weekday names
- Widths are stored as CSS variables in `em` (`--time-w`, `--cal-w`, `--week-w`).
- Measurements are redone after fonts finish loading and whenever the design, format or font changes.
- `font-variant-numeric: tabular-nums` is also applied, so fonts that support tabular figures keep their digits aligned.

The frame size never changes, but with fonts whose digits differ in width, the text may shift by a few pixels inside the frame (it is centered).

### Specifying the width manually (`width`)

With `width` set, the measurement above is skipped entirely and the clock is given exactly that width (`box-sizing: border-box`, so padding and borders are included). The contents are centered inside it.

- A plain number is in px and is **not** affected by `scale`. Use a unit such as `em` if you want it to follow `scale`.
- Text is never wrapped. If the width is too small for the content, the content overflows.


## Requirements and notes

- A browser with ES2015+ support. No external scripts are used; only Google Fonts stylesheets are loaded.
- Fonts need a network connection to load from Google Fonts. Offline, fallback fonts are used.
- `glass` relies on `backdrop-filter`. If your OBS browser source does not support it, only the translucent background is shown.
- Font weights cannot be specified by parameter. To get a bolder look, choose a font family that is itself bold.
- Only `simple` and `outline` have no panel or frame. The other designs draw their own panel or frame, so check how they look on both bright and dark scenes.
- Time zone conversion uses `Intl.DateTimeFormat`.

## File list

| File | Description |
|---|---|
| `clock.html` | The clock (HTML, CSS and JavaScript in one file) |
| `readme.md` | This file (English) |
| `readme.ja.md` | Japanese version |
