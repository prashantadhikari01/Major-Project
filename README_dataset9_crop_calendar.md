# Dataset 9: Crop Calendars for Mustang and Manang

Version 2. Prepared for the crop recommendation project (planting-window backtest).

## 1. What this dataset is

A reference table of **when each crop is sown and harvested** in the high-mountain zones of Mustang and Manang. The app's planting windows are compared against these reference windows in the backtest.

**Important:** this is a literature-based calendar. No official MOALD or Agriculture Knowledge Centre month table for Mustang or Manang was found online. Rows are labelled by how well they are supported (section 5). Treat unconfirmed rows as draft.

## 2. Files

| File | Use |
|---|---|
| `dataset9_crop_calendar.xlsx` | Full workbook with six sheets (below) |
| `crop_calendar.csv` | Same as sheet `crop_calendar`, for loading in code |
| `README_dataset9_crop_calendar.md` | This file |

Sheets in the workbook:

| Sheet | Content |
|---|---|
| `README` | Short version of this file |
| `crop_calendar` | Main table, 33 rows |
| `zones` | The 9 app locations mapped to elevation zones Z1, Z2, Z3 |
| `local_notes` | Site facts: irrigation season, snow, frost, communal dates |
| `sources` | Source list with links (S1 to S15) |
| `to_verify` | Where to get the missing primary data |

## 3. Zones

Locations are grouped by elevation because sowing time depends mainly on altitude.

| Zone | Elevation (m) | Locations |
|---|---|---|
| Z1 | 2,500 to 3,000 | Jomsom, Marpha, Kagbeni, Chame |
| Z2 | 3,000 to 3,600 | Pisang, Braga, Manang Village, Charang |
| Z3 | 3,600 to 4,000 | Lo Manthang |

Elevations on the `zones` sheet are approximate. **Replace them with the elevations from Dataset 1** before final use.

Reference-only zone codes (not app locations): `AN1` (Humla/Jumla analog), `NAT1`, `NAT2`, `NAT3` (national potato calendar by altitude).

## 4. Column dictionary (`crop_calendar`)

| Column | Meaning |
|---|---|
| `record_id` | Row ID (CC001 and so on) |
| `crop` | Potato, Barley, Wheat, Buckwheat, Maize, Millet, Pulses |
| `variety_note` | Type, for example winter-sown, naked barley, peas |
| `zone_id` | Z1, Z2, Z3, AN1, NAT1 to NAT3 |
| `elevation_band_m` | Elevation range of the zone |
| `example_locations` | App locations in that zone |
| `grown_status` | `core` main crop; `marginal` rare or risky; `not_typical` expected not grown (months blank); `analog` or `reference` other places, for comparison only |
| `season_type` | spring, summer or winter sowing |
| `sow_start_month`, `sow_end_month` | Sowing window, months 1 to 12 (Gregorian) |
| `harvest_start_month`, `harvest_end_month` | Harvest window, months 1 to 12 |
| `sow_period_bs_approx`, `harvest_period_bs_approx` | Same windows in Nepali months, approximate |
| `cycle_days_min`, `cycle_days_max` | Days from sowing to harvest, where known |
| `sow_start_doy`, `sow_end_doy`, `harvest_start_doy`, `harvest_end_doy` | Day of year (non-leap), computed from the months by formula |
| `evidence_level` | A, A/B, B or P (section 5) |
| `evidence_area` | Where the evidence comes from (Mustang/Manang, national altitude table, Humla analog, none found) |
| `source_id` | Source codes, see `sources` sheet |
| `notes` | What is sourced and what is inferred |
| `verified_by` | Blank. Fill in who confirmed the row and from which source |

Months are whole-month windows, so day-of-year values are coarse. The sowing window of a winter crop can fall in the year before its harvest (for example sown in December 12, harvested in June 6).

## 5. Evidence levels

| Level | Meaning | Rows |
|---|---|---|
| A | Months or crop presence stated in a source found online | 12 |
| A/B | Part of the row is sourced, part is inferred | 10 |
| B | Draft from general high-altitude farming knowledge, no source found | 11 |
| P | Reserved for rows confirmed from a primary local source | 0 |

Most A rows are for the national altitude table, Humla or crop presence, so check `evidence_area` before relying on one.

## 6. Key sourced facts

- **Kali Gandaki valley (Z1):** winter barley is sown in December and the fields are green by March and April. Buckwheat, maize and potato are the autumn crops (S6).
- **Jharkot, 3,550 m (Z2 band):** barley and wheat are irrigated March to early June. Buckwheat can follow barley as a second crop. The harvest comes after the August Yartung festival (S7).
- **Manang (Z2):** summer cultivation only. Snow covers the valley November to February. Wheat covers about 38% of cultivated land (S8).
- **Lo Manthang (Z3):** one crop per year. Main crops are wheat, buckwheat, naked barley, peas, mustard and potato (S9).
- **Potato by altitude (S10, secondary source):** at 2,200 to 3,000 m plant March to April and harvest July to September. At 3,000 to 4,000 m plant late April to early May and harvest September to October.
- **Buckwheat (S11):** April to May is the best sowing time above 2,000 m.

## 7. Notes for the backtest

- In Manang all farmers plant and harvest on dates fixed by the village headman with a lama (S14). In Lo Manthang the community leader sets the dates and the harvest lasts about two weeks (S9, S15). Reference dates are therefore shared village dates, not individual choices.
- Manang's valley bottom is more frost-prone than the slopes at germination and harvest (S8). Use this when interpreting frost-date results.
- Apple is perennial and is left out, as in the project plan.

## 8. How to load

```python
import pandas as pd

df = pd.read_csv("crop_calendar.csv")

# Only rows for the app zones, with real months, and sourced evidence
app = df[df.zone_id.isin(["Z1", "Z2", "Z3"])]
app = app[app.grown_status.isin(["core", "marginal"])]
sourced = app[app.evidence_level.isin(["A", "A/B"])]

# Sowing window in day of year
print(sourced[["crop", "zone_id", "sow_start_doy", "sow_end_doy",
               "harvest_start_doy", "harvest_end_doy"]])
```

## 9. What is still missing

1. The local crop calendar from the Agriculture Knowledge Centre (Krishi Gyan Kendra) in Mustang and Manang. This is the best source and is usually one page.
2. MOALD Krishi Diary or an official calendar by ecological zone.
3. Full text of Aase and Vetaas (2007), *Human Ecology* 35:453 to 460 (S14), which likely gives actual Manang planting and harvest dates.

When a row is confirmed, overwrite its months, set `evidence_level` to `P`, and fill `verified_by`.

## 10. Sources

| ID | Source |
|---|---|
| S1 | Humla cropping pattern study, JIAAS (nepjol.info/index.php/JIAAS/article/view/56971) |
| S2 | Buckwheat in Nepal review, JIAAS (nepjol.info/index.php/JIAAS/article/view/58593) |
| S3 | PotatoPro Nepal page (potatopro.com/potato-markets/nepal) |
| S4 | Holy Mountain Treks, The people of Mustang (weak source) |
| S5 | The Himalayan Times, Marpha harvest photo, 30 Oct 2016 |
| S6 | Meier, Pohle, Grießinger (2022), *Land* 11(9):1567 (doi.org/10.3390/land11091567) |
| S7 | Holmelin and Aase (2013), *Sustainability* 5(4):1387 (doi.org/10.3390/su5041387) |
| S8 | Aase, Chaudhary, Vetaas (2010), *Area* 42(2):228 |
| S9 | Khattri and Pandey, Lo Manthang agricultural adaptation (ijae.springeropen.com/articles/10.1186/s41257-020-00039-w) |
| S10 | Nepal potato altitude calendar table (secondary compilation, page could not be reopened) |
| S11 | Buckwheat in Nepal review, full text (ResearchGate) |
| S12 | Barley Cultivation and Cultural Representations, *Revue d'Etudes Tibetaines* no. 33 |
| S13 | Destination Manang, Experience Tradition (weak source) |
| S14 | Aase and Vetaas (2007), *Human Ecology* 35 (paywalled, abstract only) |
| S15 | Trans-Himalayan Livelihood in Upper Mustang (ResearchGate) |

Full links are on the `sources` sheet.
