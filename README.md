# CAN_DBC_MAZDA_MX-5 2015-1.5L/MT/JP

English | [日本語](README.ja.md)

A CAN DBC based on data collected from a Japanese-market 2015 Mazda Roadster / MX-5 (ND, 1.5L, 6MT).

The starting point was **[CAN_DBC_6thGenMazda](https://github.com/berumiya/CAN_DBC_6thGenMazda) by berumiya 990S labo**. After getting the original DBC working, I compared CAN logs from my car with actual vehicle operations, using Codex to assist with analysis of model-year differences and additions or corrections to the definitions. My thanks to berumiya for publishing the original DBC and research.

The upstream project was primarily tested on a 2022 Roadster 990S. This repository documents findings from my own 2015 car.

## Target vehicle and file

| Item | Details |
|---|---|
| Vehicle | Mazda Roadster / MX-5, 2015 S trim, ND5RC |
| Engine and transmission | 1.5L, 6-speed manual |
| Market | Japan (JP) |
| CAN | Classic CAN, 500 kbit/s |
| File | [MAZDA_MX-5_2015_1.5L_MT_JP.dbc](MAZDA_MX-5_2015_1.5L_MT_JP.dbc) |
| Contents | 75 messages, 176 signals |
| Source revision | ND1-2015-MT-20261019-R2, v1.4.1 |

## Usage

1. Open the DBC file linked above and select **Download raw file**. To download the entire repository, select **Code → Download ZIP**.
2. Import it into a CAN analysis tool that supports DBC files and set the CAN interface to 500 kbit/s. **Listen Only** mode is recommended when collecting data from a real vehicle.
3. Check signal units, comments, and value descriptions before use. Some signals interpret the same bits in different ways, so tools that check for overlapping signals may require an import option to allow overlaps.

For example, in a Python environment with cantools available:

```python
import cantools

db = cantools.database.load_file(
    "MAZDA_MX-5_2015_1.5L_MT_JP.dbc",
    strict=False,
)
```

## Main changes from the upstream DBC

- Updated interpretations of gear, fuel, lighting, and other signals for the 2015 car.
- Added candidate signals for reverse selection, reverse lamps, brake lamps, and clutch states.
- Defined a 20-bit brake-pressure candidate.
- Defined ambient temperature as `0x420 byte7 × 0.25 °C`.
- Preserved the payloads of undecoded messages as raw values.

Three VIN-related signals that use a nonstandard format are not included in this standard DBC.

## Signal interpretation notes

Byte numbers are zero-based. Motorola start-bit numbering in DBC files differs from bit numbering within a byte.

| Signal | Interpretation and notes |
|---|---|
| `Fuel_Tank` (0x09E) | Fuel-gauge reading: byte5 × 0.2 L. Inclination and acceleration can change the reading; use stable readings while stopped as a fuel-level reference. |
| `Fuel_Related` (0x43F) / `Fuel_Related_2` (0x09E) | Raw values with unknown meanings. No conversion to liters or percent, or interpretation as a compensated fuel level, has been added. |
| `C000_TOTAL_TIME` (0x40A) | Seconds counter: `raw × 0.1 + 1 sec`. Its origin and purpose are unknown. |
| `BrakePressure` (0x078) | Pressure candidate: Motorola `27\|20@0+`, `raw × 0.001 − 40 bar`. The bar conversion and interpretation of the low byte are provisional. |
| `SteeringAngle_related` (0x086) | Raw value that increases when steering left. The zero point and scale in the reference formula `(raw − 16000) × 0.1 deg` are provisional; the DBC itself keeps the raw value. |
| `Steering_Wheel_Absolute_Angle` (0x082) | The upstream definition is retained, but it does not match steering observations on this vehicle. |
| `WheelSpeed_1` through `WheelSpeed_4` (0x215) | Wheel speeds in km/h. Signals 1/3 correspond to the left side and 2/4 to the right side; front/rear assignment is unknown. |
| `Travel_distance_related` (0x420) | Distance counter with raw values 0..254, modulo 255. Observations are consistent with approximately 0.2 m/count; the DBC factor and unit remain raw. |
| `AmbientTemp` (0x420) | byte7 × 0.25 °C. Subzero encoding and invalid values are unknown. |
| `Clutch_State_Candidate` (0x166) | Raw states 0/62/187 are interpreted as released-side/intermediate/deeply-pressed-side candidates. They are not percentages of pedal force or travel. |
| `MT_Gear_Actual` (0x0FD) | Forward gears 1..6. Updates may lag a shift. Reverse is a separate candidate signal. |

Acceleration and yaw-rate data include raw and corrected candidates; their physical scaling and relationships are not fully understood. Some DBC min/max values are placeholders inherited from upstream and should not be treated as actual sensor limits.

### Reading 0x40A

The first two bytes of 0x40A form a selector (`Central_Config_Index`) that determines how to interpret the remaining payload. The current interpretations are:

| Selector | Signal | Interpretation |
|---|---|---|
| `0xC000` | `C000_TOTAL_TIME` | Seconds counter: `raw × 0.1 + 1 sec`. Origin and purpose unknown. |
| `0xC001` | `C001_ODO` | Raw value read as an odometer reading in kilometers. |
| `0xBF00` | `BF00_IG_ON_Timer` | Raw value read as seconds since engine start. Reset condition unknown. |

Payloads for the other selectors are retained as raw values.

## Brake reference scale and source

The percentage-conversion analysis referenced [the ND documentation in RaceChronoDiyBleDevice](https://github.com/timurrrr/RaceChronoDiyBleDevice/blob/841c7ab958e92b83953398475e4fc7cdd1c64771/can_db/mazda_mx5_nd.md) by Timur Iskhodzhanov.

That document provides a formula that subtracts 156 from a 12-bit value in 0x078, divides by 2.56, and clamps the result to 0..100%. The relative scale examined with my car uses the upper 12 bits of the 20-bit field, `q = raw >> 8`, and computes `max(q − 156, 0) × 100 / 256`. This can produce values above 100%.

Here, 100% is a reference-scale point, not a calibrated maximum for pedal force, pedal travel, or hydraulic pressure. **The DBC defines `BrakePressure` as a pressure candidate in bar and does not include this percentage conversion.** The 20-bit pressure hypothesis and comparisons with my car are this project's work; they do not represent pressure-calibration results supplied by the reference document.

## Usage precautions and disclaimer

This DBC is not official manufacturer documentation. It is based on independent analysis by an individual. Its accuracy and completeness are not guaranteed.

The author accepts no liability for any damage arising from the use of this data.

The author has not tested sending CAN messages to a vehicle or using this data for vehicle-control purposes.

## Sources and licenses

- Original DBC: **berumiya 990S labo**, [CAN_DBC_6thGenMazda](https://github.com/berumiya/CAN_DBC_6thGenMazda), commit `c802353b4e52546e196b8ee5dc483f73c14dbc2b`.
- Additional analysis using my car, modifications, and publication preparation: **takakka**, with Codex as an analysis assistant.
- Reference documentation: **Timur Iskhodzhanov**, [RaceChronoDiyBleDevice](https://github.com/timurrrr/RaceChronoDiyBleDevice), commit `841c7ab958e92b83953398475e4fc7cdd1c64771`, `can_db/mazda_mx5_nd.md`.

The DBC and additions or modifications to this documentation are provided under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). See [LICENSE](LICENSE) and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md). Retain the original attribution and indicate the source, license, and modifications when redistributing.

The reference material from RaceChronoDiyBleDevice retains that project's [MIT license notice](licenses/RaceChronoDiyBleDevice-MIT.txt). No endorsement or supervision by the original authors or the vehicle manufacturer is implied.
