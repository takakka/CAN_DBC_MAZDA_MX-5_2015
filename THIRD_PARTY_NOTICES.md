# Third-party notices

## berumiya / CAN_DBC_6thGenMazda

- Creator: **berumiya 990S labo**.
- Source: <https://github.com/berumiya/CAN_DBC_6thGenMazda>.
- Commit: `c802353b4e52546e196b8ee5dc483f73c14dbc2b`.
- Original file: `MX5ND_6thGenMazda_HSCAN.dbc`.
- License: **Creative Commons Attribution 4.0 International (CC BY 4.0)**, <https://creativecommons.org/licenses/by/4.0/>. Full text: [LICENSE](LICENSE).

The published DBC is adapted from this work for a Japanese-market 2015 Mazda Roadster / MX-5, 1.5L, 6MT. Changes include model-year-specific interpretations, additional candidate signals, the 20-bit brake-pressure hypothesis, ambient-temperature extraction, and shorter technical comments. The numerical definitions and value descriptions match the project's R2 v1.4.1 export; the publication preparation changes comments and notices only.

The original work is personal, hobbyist-level reverse engineering, not official manufacturer documentation. Accuracy and completeness are not guaranteed; its author disclaims responsibility for damage or loss arising from its use. No endorsement by berumiya or the vehicle manufacturer is implied.

Additional analysis and editorial changes: Copyright (c) 2026 takakka. These contributions and the accompanying documentation are offered under CC BY 4.0. Original third-party rights and notices are retained.

## timurrrr / RaceChronoDiyBleDevice

- Creator: **Timur Iskhodzhanov** (`timurrrr`).
- Source: <https://github.com/timurrrr/RaceChronoDiyBleDevice>.
- Commit: `841c7ab958e92b83953398475e4fc7cdd1c64771`.
- Referenced document: [`can_db/mazda_mx5_nd.md`](https://github.com/timurrrr/RaceChronoDiyBleDevice/blob/841c7ab958e92b83953398475e4fc7cdd1c64771/can_db/mazda_mx5_nd.md).
- License: **MIT**. Full, unmodified notice: [licenses/RaceChronoDiyBleDevice-MIT.txt](licenses/RaceChronoDiyBleDevice-MIT.txt).
- Copyright (c) 2020 Timur Iskhodzhanov.

The document was used to cross-check CAN extraction and scaling, particularly the brake-related 12-bit field and the reference percentage scale (subtract 156, divide by 2.56, and clamp to 0..100). The README describes an adapted relative percentage without the upper clamp. The DBC itself retains the project's separate, uncalibrated 20-bit pressure hypothesis in bar. That hypothesis is not a calibrated pressure definition supplied by the referenced document.

MIT-origin reference material retains its MIT notice; CC BY 4.0 does not replace that notice. The complete MIT text is also embedded in a global DBC comment so that a standalone DBC download retains it. No source code or firmware from RaceChronoDiyBleDevice or its libraries is included. No endorsement by its author or the RaceChrono application is implied.
