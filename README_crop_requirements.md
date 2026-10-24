# Dataset 7: Crop Requirements (FAO EcoCrop)

Project: crop recommendation and planting-window system for Mustang and Manang, Nepal
Prepared: 17 October 2026 | Status: ready to use, with four rows still to verify (see section 5)

## 1. Files in this package

| File | What it is |
|---|---|
| `Dataset7_crop_requirements_EcoCrop.xlsx` | Full hand-off workbook: README, data table, data dictionary, sources, verification log |
| `crop_requirements_ecocrop.csv` | The same data table as plain CSV (loads directly into pandas) |
| `README_crop_requirements.md` | This file |

## 2. What the data is

One row per crop (11 rows), metric units. Each row gives the climate and soil conditions in which the crop grows, according to FAO's EcoCrop database.

**Crops:** potato, barley, buckwheat (Tartary), wheat, maize, finger millet, proso millet, lentil, kidney bean, chickpea, faba bean.
Apple is not included (perennial; treated separately in the project plan).

**Use in the project:** scoring planting windows, checking whether a crop suits a location and season, and the rules layer. Join to other tables on the `crop` column (match the crop names used in the MOALD yield table, Dataset 6).

## 3. Columns

| Column | Meaning | Unit |
|---|---|---|
| crop_id, crop, scientific_name | Row ID, crop name, species used | - |
| temp_opt_min_c, temp_opt_max_c | Optimal mean temperature range | deg C |
| temp_abs_min_c, temp_abs_max_c | Absolute (survival) temperature limits | deg C |
| rain_opt_min_mm, rain_opt_max_mm | Optimal annual rainfall | mm |
| rain_abs_min_mm, rain_abs_max_mm | Absolute annual rainfall limits | mm |
| ph_opt_min, ph_opt_max | Optimal soil pH | pH |
| ph_abs_min, ph_abs_max | Absolute soil pH limits | pH |
| cycle_min_days, cycle_max_days | Crop cycle, sowing to harvest | days |
| killing_temp_rest_c | Temperature that kills the crop (for mirror rows, the "Winter Minimum") | deg C |
| killing_temp_early_growth_c | Killing temperature at seedling stage | deg C |
| max_altitude_m | Highest altitude listed in EcoCrop | m |
| frost_class_derived | **Our own label, not FAO.** With a killing temp: hardy (<= -5), semi-hardy (-5 to -2), tender (> -2). Without: tender if temp_abs_min_c >= 8, otherwise unknown | - |
| source_type, source_ref, data_status | Where the row came from and how far to trust it | - |
| notes | Row-specific caveats | - |

## 4. Sources

- **Official FAO EcoCrop datasheets (7 rows, `data_status = official`):** maize, finger millet, proso millet, lentil, kidney bean, chickpea, faba bean. Link for each is in `source_ref`. Database: https://ecocrop.apps.fao.org
- **University of Illinois, Alternative Crops tables (4 rows, `data_status = mirror_check_recommended`):** potato, barley (spring), Tartary buckwheat, wheat (hard red spring). These republish EcoCrop-based values. I converted degrees F to C and inches to mm.

**Citations for the report**
- FAO. *EcoCrop database*. https://ecocrop.apps.fao.org (accessed 17 October 2026).
- Illinois State Water Survey. *Alternative Crops, crop requirements*. https://isws.illinois.edu/data/altcrops/ (accessed 17 October 2026).

## 5. Quality checks and known limits

**Checked:** every value was compared again with its source page (187 of 187 matched), and no minimum exceeds its maximum. The full log is in the `verification` sheet of the workbook. This confirms the values were copied correctly; it does not make the mirror rows official.

**Limits to know about**
1. **Four mirror rows.** Potato, barley, wheat and buckwheat did not come from official FAO pages. The mirror gives only one rain range and one pH range, which I stored as the optimal range (an assumption). Their absolute rain and pH columns are empty. Replace these rows with the official EcoCrop values before final submission.
2. **Buckwheat.** The row is Tartary buckwheat (*Fagopyrum tataricum*, the colder-climate species). Common buckwheat (*F. esculentum*) is not collected yet.
3. **Spring types.** Wheat and barley rows are spring varieties. Confirm with your supervisor that this matches practice in Mustang and Manang. Winter types have much longer cycles and colder killing temperatures.
4. **Thin frost data.** Only chickpea has official killing temperatures. EcoCrop lists "no input" for the other official crops.
5. **Expert opinion.** EcoCrop values are expert estimates, not measured optima. State this in the report.

## 6. Handling missing values

An empty cell means the source gives no value. It does **not** mean zero. Do not fill with 0 or with an average across crops.

Suggested frost rule (agree with your supervisor, then state it as an assumption in the report):
1. Use `killing_temp_early_growth_c` if present.
2. Otherwise use `killing_temp_rest_c`.
3. If both are empty, use 0 deg C (the standard definition of frost).

Keep a flag for which crops used the 0 deg C default.

```python
import pandas as pd
df = pd.read_csv("crop_requirements_ecocrop.csv")   # empty cells load as NaN
df["frost_threshold_c"] = (df["killing_temp_early_growth_c"]
                           .fillna(df["killing_temp_rest_c"])
                           .fillna(0.0))
df["frost_threshold_is_default"] = (df["killing_temp_early_growth_c"].isna()
                                    & df["killing_temp_rest_c"].isna())
```

For rain and pH when absolute limits are empty: score using the optimal range only (full score inside, penalty outside).

## 7. Remaining to-do

- [ ] Replace the four mirror rows with official EcoCrop values (potato, barley, wheat, common buckwheat)
- [ ] Confirm spring vs winter wheat and barley with the supervisor
- [ ] Agree the frost fallback rule
