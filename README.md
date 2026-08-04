# PA-44 Performance Calculator

**Live:** https://gopilot.blog/pa44-performance-calculator/

A single-file, POH-accurate performance calculator for the **Piper PA-44-180 Seminole** (2× Lycoming O-360-A1H6, counter-rotating, G1000 NXi). Performance charts from Section 5 of the POH were digitized and anchored to the POH's own worked examples. Enter a METAR (or fetch one live), set weight and cruise altitude, and get climb / cruise / descent / runway numbers — plus the **single-engine (OEI) figures** that matter for multi-engine training and checkrides.

Built for multi-engine training and MEI prep at high-density-altitude airports like KPVU (field elev 4,497 ft).

---

## Features

- **Live METAR** — fetch by ICAO through GoPilot's own API (`api.gopilot.blog`, a Cloudflare Worker proxying the NOAA Aviation Weather Data API), with a 5-proxy public fallback and manual raw-METAR paste that auto-parses ICAO, wind, temperature, and altimeter
- **OEI · Single Engine panel** — the multi-engine centerpiece:
  - OEI climb rate at the **field** (current PA/OAT/weight)
  - OEI climb rate at **cruise altitude**
  - **Single-engine service ceiling** — the pressure altitude where OEI ROC decays to 50 fpm at the current ISA deviation and weight
  - Chart conditions per POH 5-21: 5° bank toward the operating engine, prop feathered, gear up
- **All-engine climb** — POH 5-19 chart at the climb **midpoint** PA/OAT, weight-corrected; climb at 88 KIAS (V<sub>YSE</sub>-referenced) converted IAS→CAS→TAS at midpoint DA
- **Cruise** — MP from POH 5-25 at cruise PA with ISA correction (±1 % per 8 °C), TAS from POH 5-27 at cruise PA/OAT (3,800 lb chart weight), total both-engine fuel flow from POH 5-25/5-27
- **Descent** — glidepath-based: `ROD = GS × (6076.12 / 60) × tan(γ)`, default 3.0°, adjustable in 0.1° steps
- **Wind** — course input gives head/tail and left/right crosswind components; ground speed, time, and distance for every phase
- **Take-off distance** — ground roll and 50 ft obstacle from POH Fig 5-13 / 5-15, weight and wind corrected
- **Accelerate-stop distance** — POH Fig 5-11: abort at V<sub>R</sub>, max braking, flaps 0° — the number every multi-engine takeoff briefing needs
- **Landing distance** — ground roll and 50 ft obstacle from POH Fig 5-35 / 5-37, weight and wind corrected
- **Climb fuel flow** — anchored to the POH §5.5 worked example (Fig 5-23, full throttle, full rich, both engines)

## Data Source

All performance data comes from one document:

> **Piper PA-44-180 Seminole Pilot's Operating Handbook / AFM, Report VB-2636 (Nov 3, 2016, rev. Dec 15, 2017), G1000 NXi**

| POH reference | Content |
|---|---|
| 5-3 | Airspeed calibration (IAS→CAS) |
| Fig 5-11 | Accelerate-stop distance |
| Fig 5-13 / 5-15 | Take-off ground roll / 50 ft obstacle |
| 5-19 | Climb performance (both engines) |
| 5-21 | Climb performance — one engine inoperative |
| Fig 5-23 (§5.5 example) | Climb fuel flow |
| 5-25 | Cruise power setting / manifold pressure |
| 5-27 | Cruise TAS and fuel flow |
| Fig 5-35 / 5-37 | Landing ground roll / 50 ft obstacle |

Values are looked up with interpolation between grid points; chart data is anchored to the POH's own worked examples.

## Why the Seminole Is Different

The PA-44 has **counter-rotating propellers** (left O-360-A1H6, right LO-360), so there is **no critical engine** — the asymmetric-thrust penalty is the same whichever engine fails. The OEI panel in this calculator exists because single-engine performance is the whole point of multi-engine training: on a hot day at a high-elevation field, the single-engine service ceiling can sit uncomfortably close to (or below) terrain, and this tool makes that visible before engine start.

## Core Formulas

```
PA  = elev + (29.92 − altim) × 1000
ISA = 15 − 1.98 × PA/1000                      [°C]
DA  = PA + 120 × (OAT − ISA)
σ   = (1 − 6.875586e-6 × DA)^4.2561
TAS = CAS / √σ
HW  = WS × cos(WindDir − Course)               [+ head / − tail]
XW  = WS × sin(WindDir − Course)               [+ right / − left]
ROD = GS × (6076.12 / 60) × tan(γ)             [1 NM = 6076.12 ft]
```

## Usage

- **Online:** open the live link, paste or fetch a METAR, adjust weight/altitude/power — everything recomputes instantly.
- **Offline:** download `index.html` — fully self-contained, no build, no dependencies beyond Google Fonts. METAR paste works with no network.
- On mobile, "Add to Home Screen" makes it behave like an app.

## Architecture

```
gopilot.blog/pa44-performance-calculator/   ← this repo (GitHub Pages)
api.gopilot.blog                            ← Cloudflare Worker METAR proxy
    └→ aviationweather.gov Data API         (60 s edge cache per ICAO)
```

The METAR proxy exists because aviationweather.gov does not send CORS headers, so browsers cannot call it directly. The Worker adds `Access-Control-Allow-Origin` and caches each ICAO for 60 seconds to respect NOAA's rate-limit guidance.

## Limitations

- Interpolation between chart grid points — anchor-verified at the POH worked examples, expect roughly ±5 % elsewhere
- Runway and accelerate-stop distances assume a paved, dry, level runway; apply POH safety factors for anything else
- Cruise TAS chart is referenced at 3,800 lb — light weights will true out slightly faster than shown
- OEI figures assume the POH chart configuration (feathered, gear up, 5° bank); anything less costs climb rate

## Disclaimer

**Planning reference only.** This tool is not a substitute for the POH/AFM. Always cross-check performance figures with the charts in the aircraft before flight. POH data remains the property of Piper Aircraft, Inc.; this project digitizes it solely for personal training and flight-planning use.

## Version History

- **v2.1** — own METAR API (`api.gopilot.blog`) as primary source; public proxies demoted to fallback
- **v2.0** — initial public release: full chart digitization, OEI panel, accelerate-stop, runway distances, glidepath descent

---

Built by **Philip Baek** · UVU Aviation Science · [gopilot.blog](https://gopilot.blog)
