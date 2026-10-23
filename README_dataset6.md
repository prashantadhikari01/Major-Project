# Dataset 6 – District crop statistics (Nepal) – hand-off notes

Built for the Mustang/Manang planting-window and crop project. District-level **area, production and yield** of
potato, barley, buckwheat, wheat, maize and millet, ready to be joined with weather and soil data.

## Which file do I use?

| File | Use it for |
|---|---|
| `tableA_skeleton.csv` | **The model-ready table.** One row per district × year × crop (1,332 rows). Join weather/soil onto this. |
| `crop_stats_clean.csv` | Everything that was kept (2,712 rows, incl. years outside the weather window), with source names. For reference. |
| `raw_extracted/yearbook_*_district_crops.csv` | The numbers pulled from each yearbook PDF (one file per year). `*_audit.csv` adds the PDF page of every value. |
| `crop_stats_dataset6.ipynb` | The Colab notebook that combines all sources and rebuilds the two tables above. |
| `extract_yearbook_tables.py` | Script that extracts the district tables from a yearbook PDF. |

## Where the numbers come from

* **ICIMOD NepalAgriStats** (cereal + cash-crop files), fiscal years 2003/04 – 2013/14. Buckwheat only 2010/11 – 2013/14.
* **MOALD yearbooks "Statistical Information on Nepalese Agriculture"**: 2075/76 (2018/19), 2076/77 (2019/20),
  2077/78 (2020/21), 2078/79 (2021/22), 2080/81 (2023/24). One fiscal year per yearbook.
* Years covered in `tableA_skeleton.csv`: **2003–2013, 2018, 2019, 2020, 2021, 2023**.
  **Missing: 2014–2017 and 2022** (those yearbooks could not be found online).
* Districts: Group A = Mustang, Manang. Group B = Dolpa, Humla, Mugu, Jumla, Bajhang, Darchula, Rasuwa, Dolakha,
  Solukhumbu, Sankhuwasabha, Taplejung, Gorkha, Myagdi (**starting list, still to be confirmed against MOALD
  ecological zones** – edit `GROUP_B` in the notebook and re-run if it changes).
* Pulses are **not** included (the yearbook splits them by type with no total).

## Column dictionary (`tableA_skeleton.csv`)

| Column | Meaning |
|---|---|
| `district`, `group` | District name; `A` = Mustang/Manang (the app districts), `B` = other districts used only for learning |
| `crop` | potato, barley, buckwheat, wheat, maize, millet |
| `year` | **Start year of the Nepali fiscal year.** `2020` means FY 2020/21. Bikram Sambat years were converted. |
| `area_ha`, `production_mt` | Area (hectares) and production (metric tonnes) |
| `yield_kg_ha` | Yield in kg/ha (yearbooks give t/ha; converted ×1000). Missing yields are filled with production×1000/area. |
| `area_share` | This crop's area ÷ total area of the six crops in that district-year |
| `grown` | 1 if area > 0 |
| `commonly_grown` | 1 if `area_share` ≥ 5% (threshold is a design choice – confirm with supervisor). **≈83% of rows are 1**, so this label is very imbalanced. |
| `yield_mismatch` | 1 if the printed yield differs >10% from production ÷ area (source inconsistency) |
| `yield_outlier` | 1 if yield is far from the crop's typical value (check against the source before trusting) |
| `repeat_prev_year` | 1 if production is identical to the previous year (likely a copied/estimated figure) |
| `zero_area_pos_prod` | 1 if area is 0 but production > 0 (impossible; source error) |
| `split_time` | `train` / `test` – test = the last 3 years with data (2020, 2021, 2023) |
| `split_place` | `train` = Group B districts, `test` = Group A (Mustang, Manang) |
| `usable_for_yield` | 1 if yield > 0 **and** `yield_mismatch` = 0. Use this to select rows for the yield regressor (1,291 rows). |

## How to use it correctly

1. **Yield regressor:** use rows with `usable_for_yield == 1`. Consider also dropping `yield_outlier == 1` after checking them.
2. **Classifier ("is this crop grown here"):** use `grown` or `commonly_grown` (see the imbalance warning).
3. **Never split randomly.** Use `split_time` (honest forecast test) and `split_place` (train on Group B, test on Mustang/Manang).
4. **Joining weather/soil:** join on `district` + `year`. Weather is per location, so first aggregate the 9 app
   locations to their district (Mustang: Jomsom, Marpha, Kagbeni, Charang, Lo Manthang; Manang: Chame, Pisang,
   Braga, Manang Village) and give every Group B district a representative point. Weather must cover **2003–2013,
   2018–2021 and 2023**.
5. **Season timing (please confirm):** `year` is the fiscal-year start. Spring/summer crops (potato, maize, millet,
   buckwheat) are sown and harvested within that calendar year. Winter crops (wheat, barley) are probably recorded in the
   fiscal year of their spring harvest, so their growing season may start in the previous autumn. Decide the season
   window per crop and state it in the report.

## Known limitations (state these in the report)

* **Mustang and Manang values are noisy.** E.g. Manang potato yield: 16.0 (2018), 18.6 (2019), 11.1 (2020),
  20.1 (2021), 12.5 (2023) t/ha. About 67 of 359 Group A rows repeat the previous year's production exactly.
* **Possible break between ICIMOD (≤2013) and the yearbooks (≥2018).** Mustang potato yield 9.1 t/ha in 2013 vs 15.7 in 2018.
* **Yearbooks contain printing errors** (e.g. Dolakha buckwheat/barley yields look swapped in 2077/78). Extracted numbers match
  the PDF text; flagged rows are marked by `yield_mismatch`.
* District yields have no planting date, so this table cannot teach planting-date effects.
* A dash (`-`) in a yearbook table was recorded as 0 (crop not grown); ICIMOD already contains explicit zeros.
* Districts such as Nawalparasi/Rukum were split in 2015; they are not among the project districts.

## Rebuilding

Open `crop_stats_dataset6.ipynb` in Colab, upload the ICIMOD CSVs and the `raw_extracted/yearbook_*` CSVs, run all cells.
To add another yearbook: `python extract_yearbook_tables.py FILE.pdf 2022/23 yearbook_2022_23_district_crops.csv`
(needs `poppler-utils`; in Colab `!apt-get -q install poppler-utils`), then re-run the notebook.
