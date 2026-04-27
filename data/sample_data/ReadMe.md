# Sample Data

## Overview

This folder contains **small, representative samples** of processed data from both cities. These files are useful for:
- Testing and debugging code without loading full datasets
- Quick exploratory analysis
- Validation of data pipeline
- Reference examples for reproducibility

All sample files are already cleaned and filtered — ready for immediate analysis. All sample files can be access via the link: https://drive.google.com/drive/folders/1AtaR1FvMq6M_uG7MQ7EfOx-x1kQFoJi9.

---

## Files in This Folder

### Amsterdam Sample Data

#### `odin/odin_sample.csv`
**Size**: ~50 KB | **Rows**: 10 | **Columns**: 267 (full raw data)

**Content**: Raw ODiN 2023 records (before cleaning/harmonization)

**Purpose**: Reference for raw data structure

**Key Columns** (sample of important ones):
- `OP`, `OPID`: Respondent identifier
- `WoGem`: Residential municipality
- `Geslacht`: Gender
- `Leeftijd`: Age
- `KHvm`: Main transport mode class
- `Hvm`: Detailed transport mode
- `Reisduur`: Travel time (minutes)
- `AfstV`: Distance (hectometers)
- ...and 250+ more

**How to Load**:
```python
import pandas as pd

df_sample = pd.read_csv('odin/odin_sample.csv')
print(df_sample.shape)  # (10, 267)
print(df_sample.columns[:10])
```

**Columns Reference**: See `odin_metadata.csv` (in this folder) for column-by-column definitions

---

#### `odin/odin_metadata.csv`
**Size**: ~10 KB | **Rows**: 267 | **Columns**: 4

**Content**: Metadata for all 267 columns in `odin_sample.csv`

**Structure**:
| column_original | column_label_dut | column_labels_en | dtype |
|---|---|---|---|
| OP | Nieuwe persoon | New person | float64 |
| OPID | Uniek id voor iedere OP | Unique ID for each respondent | float64 |
| Geslacht | Geslacht OP | Gender of respondent | float64 |
| ... | ... | ... | ... |

**How to Use**:
```python
import pandas as pd

metadata = pd.read_csv('odin/odin_metadata.csv')

# Find English name of a Dutch column
dutch_col = 'Geslacht'
english_col = metadata[metadata['column_original'] == dutch_col]['column_labels_en'].values[0]
print(f"{dutch_col} → {english_col}")  # Geslacht → Gender of respondent

# Get all column names
all_cols_en = metadata['column_labels_en'].tolist()
```

**How to Map Columns**:
```python
# Create a mapping dictionary
col_mapping = dict(zip(
    metadata['column_original'], 
    metadata['column_labels_en']
))

# Rename columns
df_sample = df_sample.rename(columns=col_mapping)
```

---

### London Sample Data

#### `london_sample_trip_2023.csv`
**Size**: ~30 KB | **Rows**: 100 | **Columns**: 28 (Trip-level data)

**Content**: NTS 2023 trip-level records (one row = one trip)

**Key Columns**:
- `TripID`: Unique trip identifier
- `IndividualID`: Person ID
- `PSUID`: Primary sampling unit (geography)
- `MainMode_B04ID`: Transport mode code (1-13)
- `TripPurpose_B04ID`: Trip purpose code (1-8)
- `TripTravTime`: Travel time (minutes)
- `TripDisIncSW`: Distance including short walk (miles)
- `TripStartHours`: Departure hour (0-23)
- `Age_B04ID`: Age band (1-9)
- `Sex_B01ID`: Gender (1=Male, 2=Female)
- `DrivLic_B02ID`: Driving license status (1-3)
- `NumCarVan`: Number of household cars/vans
- `HHIncQIS2023Eng_B01ID`: Income quintile (1-5, 1=lowest)
- `W5`: Trip weight (for population expansion)

**How to Load**:
```python
import pandas as pd

df_trips = pd.read_csv('london_sample_trip_2023.csv')
print(df_trips.shape)  # (100, 28)

# Check mode distribution
print(df_trips['MainMode_B04ID'].value_counts())
```

**Mode Code Reference**:
```python
mode_dict = {
    1: "Walk",
    2: "Bicycle",
    3: "Car driver",
    4: "Car passenger",
    5: "Motorcycle",
    6: "Other private",
    7: "Bus (London)",
    8: "Other local bus",
    9: "Non-local bus/coach",
    10: "Underground/metro/LR",
    11: "Surface rail",
    12: "Taxi/minicab",
    13: "Other public"
}

df_trips['mode_name'] = df_trips['MainMode_B04ID'].map(mode_dict)
```

---

#### `london_sample_individual_2023.csv`
**Size**: ~20 KB | **Rows**: 50 | **Columns**: 50+ (Person-level data)

**Content**: NTS 2023 individual characteristics (one row = one person)

**Key Columns**:
- `IndividualID`: Person ID
- `HouseholdID`: Household ID
- `PSUID`: Geographic unit
- `PersNo`: Person number within household
- `Age_B04ID`: Age band (1-9)
- `Sex_B01ID`: Gender
- `DrivLic_B02ID`: Driving license status
- `EcoStat_B03ID`: Employment status (4 categories)
- `MaritalS_B01ID`: Marital status

**How to Load and Merge**:
```python
df_individuals = pd.read_csv('london_sample_individual_2023.csv')

# Merge with trip data on IndividualID
df_merged = df_trips.merge(
    df_individuals[['IndividualID', 'EcoStat_B03ID', 'MaritalS_B01ID']],
    on='IndividualID',
    how='left'
)
print(df_merged.shape)
```

---

#### `london_sample_household_2023.csv`
**Size**: ~15 KB | **Rows**: 40 | **Columns**: 40+ (Household-level data)

**Content**: NTS 2023 household characteristics (one row = one household)

**Key Columns**:
- `HouseholdID`: Household ID
- `PSUID`: Geographic unit
- `HHoldNumPeople`: Total household size
- `NumCarVan`: Number of cars/light vans
- `HHIncQIS2023Eng_B01ID`: Income quintile (1-5)
- `HHoldStruct_B02ID`: Household structure (1-6)

**How to Load and Merge**:
```python
df_households = pd.read_csv('london_sample_household_2023.csv')

# Merge household income with trip data
df_merged = df_trips.merge(
    df_households[['HouseholdID', 'HHoldNumPeople', 'NumCarVan']],
    left_on='HouseholdID',
    right_on='HouseholdID',
    how='left'
)
```

---

#### `london_sample_psu_2023.csv`
**Size**: ~5 KB | **Rows**: 8 | **Columns**: 10 (Geography data)

**Content**: NTS 2023 Primary Sampling Unit (PSU) geography

**Key Columns**:
- `PSUID`: Unique PSU identifier
- `PSUStatsReg_B01ID`: Statistical region (1-13, London=8)
- `SurveyYear`: Year of survey (2023)

**Purpose**: Maps PSU codes to regions; useful for geographic filtering

---

## How These Samples Relate to Full Data

| File | Sample Size | Full Size | Purpose |
|------|------------|-----------|---------|
| `odin_sample.csv` | 10 rows | 681 rows | Test raw structure |
| `london_sample_trip_2023.csv` | 100 rows | 25,188 rows | Test trip-level analysis |
| `london_sample_individual_2023.csv` | 50 rows | ~11,000 individuals | Test person-level merges |
| `london_sample_household_2023.csv` | 40 rows | ~8,000 households | Test household attributes |

**Scaling**: To estimate full dataset behavior, multiply sample results by expansion factors.

---

## Notes

- **Samples are representative** — Randomly selected from full dataset
- **All coding/filtering work is on FULL data** — These are for reference only
- **No missing values imputed** — Samples reflect real data quality
- **Weights preserved** — Trip and person weights are included
- **Not for inference** — These are purely for pipeline validation

---

## How to Access Full Data

**Full datasets are on Google Drive** (due to GitHub file size limits)

**Download Link**: [See main README.md for Google Drive link]

## Metadata Summary

**Column Meanings for Amsterdam**:
```
OP = New person (1=main respondent in household)
Geslacht = Gender (1=Male, 2=Female)
Leeftijd = Age (numeric years)
KHvm = Main mode class (1=Car, 4=PT, 5=Bicycle)
Reisduur = Travel time (minutes)
AfstV = Distance (hectometers; divide by 10 for km)
HHGestInkG = Income (deciles 1-10)
```

**Column Meanings for London**:
```
MainMode_B04ID = Transport mode (see mode_dict above)
TripPurpose_B04ID = Purpose (1=Commute, 2=Business, etc.)
TripTravTime = Travel time (minutes)
TripDisIncSW = Distance (miles)
Age_B04ID = Age band (1-9 categories)
Sex_B01ID = Gender (1=Male, 2=Female)
W5 = Trip weight (for expansion to population level)
```
