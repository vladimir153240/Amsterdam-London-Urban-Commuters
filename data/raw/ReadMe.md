# Raw Data Access & Setup

For the purposes of this project the raw_data is currently stored at: https://drive.google.com/drive/folders/12vy_0574lDJ9U7t6vDy4DTOHAFfWpesc?usp=sharing.

To access the data used in the analysis_notebooks access the folder raw_data. This folder contais the original sav file for Amsterdam and 5 tab files for London. The files are heavy and must be downloaded locally to reproduce the analysis_notebooks.    

To access the data used in the implementation notebooks go to the folder "implementation_data". This folder contains Parquet-compressed version for Amsterdam Data and fildered csv dataset for UK 2023. These two files are uploaded to save memory and reproduce the project via the notebooks in the implementation_notebooks.   


---

For full access of the raw data proceed as per the below:

## Amsterdam Raw Data

### File Information

| Item | Detail |
|------|--------|
| **Original File** | `ODiN2023_Databestand.sav` |
| **Size** | 122 MB (SPSS format) |
| **Compressed** | `odin_2023_raw.parquet` (~43 MB) |
| **Survey** | ODiN (Onderweg in Nederland) 2023 |
| **Coverage** | All Netherlands, all modes, all purposes |

### How to Access

#### Option A: Official Source (Free Registration)

1. Visit: https://www.cbs.nl/nl-nl/onze-diensten/methoden/onderzoeksomschrijvingen/korte-onderzoeksbeschrijvingen/odin
2. Download `ODiN2023_Databestand.sav`
3. Place in: `data/raw/odin/`

#### Option B: DANS Data Station (DOI)

```
Citation: Informatiepunt, ipwvl and Centraal Bureau voor de Statistiek (2024). 
ONDERZOEK ONDERWEG IN NEDERLAND - ODiN 2023. 
DANS Data Station Social Sciences and Humanities. 
https://doi.org/10.17026/SS/FNXJEU
```

Download: https://doi.org/10.17026/SS/FNXJEU

### Loading Code

```python
import pyreadstat
import pandas as pd

# Load original .sav file
odin_data, meta = pyreadstat.read_sav('data/raw/odin/ODiN2023_Databestand.sav')

# Get English column labels
columns = meta.column_labels

print(f"Loaded {len(odin_data):,} records, {len(odin_data.columns)} variables")
```

### Key Variables (Used in Analysis)

| Variable | Description | Codebook |
|----------|-------------|----------|
| `WoGem` | Residential municipality (CBS code) | 363 = Amsterdam |
| `MotiefV` | Trip purpose | 1 = Commuting ("Van en naar het werk") |
| `Weekdag` | Weekday (1-7) | [2-6] = Mon-Fri |
| `KHvm` | Main mode class | 1=Car, 4=PT, 5=Bicycle |
| `Reisduur` | Travel time (minutes) | Numeric |
| `AfstV` | Distance (hectometers) | Numeric; divide by 10 for km |
| `Leeftijd` | Age (years) | Numeric |
| `Geslacht` | Gender | 1=Male, 2=Female |
| `HHGestInkG` | Household income (deciles) | 1-10 |
| `OPRijbewijsAu` | Car license holder | 1=Yes, 0=No |
| `HHAuto` | Household car ownership | Count |

**Codebook**: `data/data_documentation/ODiN2023_Codeboek_v1.0.tab`

---

## London Raw Data

### File Information

| Item | Detail |
|------|--------|
| **Archive** | UKDA-5340-tab (UK Data Archive Study 5340) |
| **Format** | Tab-delimited (.tab files) |
| **Coverage** | 2002-2024 (23 years), all UK regions, all modes |
| **Size** | 5 files, total ~1.65 GB |
| **2023 Only** | ~400 MB (pre-filtered) |

### Files Description

| File | Records | Size | Purpose |
|------|---------|------|---------|
| `trip_eul_2002-2024.tab` | 3.2M | 1,015 MB | Individual trip records |
| `individual_eul_2002-2024.tab` | 360K | 258 MB | Personal characteristics |
| `household_eul_2002-2024.tab` | 110K | 112 MB | Household attributes |
| `psu_eul_2002-2024.tab` | 30K | 0.4 MB | Primary sampling units (geography) |
| `day_eul_2002-2024.tab` | 360K | 185 MB | Travel day information |

### How to Access

#### Option A: UK Data Service (Free Registration Required)

1. Visit: https://beta.ukdataservice.ac.uk/datacatalogue/series/series?id=2000037
2. Register (free, academic login available)
3. Download Study 5340 (2002-2024)
4. Extract `.tab` files to: `data/raw/london/raw/UKDA-5340-tab/tab/`

#### Option B: DOI Citation

```
Citation: Department for Transport (2025). 
National Travel Survey, 2002-2024 [data collection]. 
19th Edition. UK Data Service. 
SN: 5340. DOI: https://doi.org/10.5255/UKDA-SN-5340-15
```

### Loading Code

```python
import pandas as pd

# Load trip-level data
trip = pd.read_csv(
    'data/raw/london/raw/UKDA-5340-tab/tab/trip_eul_2002-2024.tab',
    sep='\t',
    low_memory=False
)

# Filter to 2023
trip_2023 = trip[trip['SurveyYear'] == 2023]

print(f"Loaded {len(trip_2023):,} trips for 2023")
```

### Key Variables (Used in Analysis)

| Variable | Description | Values |
|----------|-------------|--------|
| `SurveyYear` | Survey year | 2023 |
| `PSUID` | Primary sampling unit ID | Numeric |
| `IndividualID` | Person ID | Numeric |
| `HouseholdID` | Household ID | Numeric |
| `DayID` | Travel day ID | Numeric |
| `TripPurpose_B04ID` | Trip purpose (8 categories) | 1=Commuting |
| `MainMode_B04ID` | Main transport mode (13 categories) | 2=Bicycle, 3=Car driver, 7/8/10/13=PT modes |
| `TripTravTime` | Travel time (minutes) | Numeric |
| `TripDisIncSW` | Distance including short walk (miles) | Numeric |
| `TripStartHours` | Departure hour | 0-23 |
| `TravelWeekDay_B01ID` | Day of week | 1=Mon, ..., 7=Sun |
| `Age_B04ID` | Age band (9 categories) | 1-9 |
| `Sex_B01ID` | Gender | 1=Male, 2=Female |
| `HHIncQIS2023Eng_B01ID` | Income quintile (2023, England) | 1-5 |
| `DrivLic_B02ID` | Driving license | 1=Full, 2=Provisional, 3=None |
| `NumCarVan` | Household cars/vans | Count |
| `PSUStatsReg_B01ID` | Statistical region | 8=London |
| `W5` | Trip weighting factor | Numeric |

**Codebook**: `data/data_documentation/Main Table Variables London.csv`

---

## Data Merging (London Only)

London data requires **joining across tables** using ID keys:

```python
# Example: Merge trip + individual + household

trip_2023 = pd.read_csv('...trip_eul_2002-2024.tab', sep='\t')
trip_2023 = trip_2023[trip_2023['SurveyYear'] == 2023]

individual = pd.read_csv('...individual_eul_2002-2024.tab', sep='\t')
individual = individual[individual['SurveyYear'] == 2023]

household = pd.read_csv('...household_eul_2002-2024.tab', sep='\t')
household = household[household['SurveyYear'] == 2023]

# Merge
merged = trip_2023.merge(individual, on=['IndividualID', 'PSUID'], how='left')
merged = merged.merge(household, on=['HouseholdID', 'PSUID'], how='left')

print(f"Merged dataset: {merged.shape}")
```

**This is automated in Phase 2 notebook!**

---

## File Integrity Checks

After loading, verify expected sample sizes:

```python
import pandas as pd

# Amsterdam
amst = pd.read_parquet('data/raw/odin/odin_2023_raw.parquet')
assert len(amst) == 681, f"Expected 681 rows, got {len(amst)}"

# London
lnd = pd.read_parquet('data/raw/london/raw/UKDA-5340-tab/tab/nts_2023_trip.parquet')
assert len(lnd) == 25188, f"Expected 25,188 rows, got {len(lnd)}"

print("✅ All data integrity checks passed!")
```

---

## Data Documentation

📋 **For complete variable descriptions**, see:

- **Amsterdam**: `data/data_documentation/ODiN2023_Codeboek_v1.0.tab`
- **London**: `data/data_documentation/Main Table Variables London.csv`
- **Harmonized schema**: `data/data_documentation/english_column_labels.txt`

---

## Legal & Citation Requirements

✅ **Always cite the original sources** when using this data:

**Amsterdam:**
```bibtex
@data{SS/FNXJEU_2024,
  author = {Informatiepunt, ipwvl and Centraal Bureau voor de Statistiek},
  publisher = {DANS Data Station Social Sciences and Humanities},
  title = {{ONDERZOEK ONDERWEG IN NEDERLAND - ODiN 2023}},
  year = {2024},
  version = {V1},
  doi = {10.17026/SS/FNXJEU}
}
```

**London:**
```bibtex
@data{DfT_NTS_5340,
  author = {Department for Transport},
  publisher = {UK Data Service},
  title = {National Travel Survey, 2002-2024},
  year = {2025},
  edition = {19th},
  doi = {10.5255/UKDA-SN-5340-15}
}
```

---

## Next Steps

1. Download & place raw data files in correct directories
2. Run **Phase 1 & 2 notebooks** to load & filter data
3. Processed data automatically saved to `data/processed/`
4. Proceed to Phase 3 (EDA)

**Questions?** See `notebooks/README.md` for notebook-specific guidance.
