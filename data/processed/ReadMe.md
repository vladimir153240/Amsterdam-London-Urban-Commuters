# Processed Data

## Overview

This folder contains **harmonized, cleaned, and filtered datasets** from both cities, ready for exploratory data analysis (EDA) and statistical modeling. 

These are the final datasets used for all analysis phases — all filters, data quality checks, and standardizations have been applied. The files can be accessed via the link: https://drive.google.com/drive/folders/1k3Kp7FmfG1fdkhVB9PYi1cJ5DTJZPHs2.

---

## Files in This Folder

### **Amsterdam Processed Data: `amst_processed.csv`**

**Size**: ~500 KB | **Rows**: 681 | **Columns**: 31

**Source**: ODiN 2023 raw data → filtered & harmonized

**Description**: Amsterdam commuting trips (weekdays only, Mon-Fri) for the 3-mode choice model (car, PT, bicycle).

#### Filtering Applied

**Starting Dataset**: 1,500,000+ trips from ODiN 2023

**Filters Applied** (sequential):
1. **Regular trips only** (new_trip_id = 1): Remove trip series → ~1,200,000
2. **Amsterdam residents** (home_municipality = 363): → ~45,000
3. **Commuting purpose** (trip_purpose = 1): → ~25,000
4. **Weekdays only** (Mon-Fri): → ~22,000
5. **Exclude holidays** (is_holiday = 0): → ~21,000
6. **Choice set modes** (car, PT, bicycle): → ~5,000
7. **Car driver role** (exclude car passengers): → ~4,800
8. **Known income** (not decile 11): → ~3,500
9. **Known car ownership** (not 10): → ~2,200
10. **Known driving license** (not 2): → **681**

**Final Sample**: 681 commuting trips (44.3% bicycle, 26.4% car, 13.8% PT)

#### Data Notes

✅ No missing values in key columns (travel_time, distance, mode)  
✅ All rows are valid commuting trips  
✅ Weights are population expansion factors — use in statsmodels `freq_weights`  
✅ Distance converted to km (original was hectometers)  
✅ Peak hour coded as 7-9 AM or 5-7 PM  

---

### **London Processed Data: `lnd_processed.csv`**

**Size**: ~1.2 MB | **Rows**: 25,188 | **Columns**: 28

**Source**: NTS 2023 raw data (trip, individual, household merged) → filtered & harmonized

**Description**: London commuting trips (weekdays only, Mon-Fri) for the 3-mode choice model (car, PT, bicycle).

#### Filtering Applied

**Starting Dataset**: 3.2M+ trips in NTS 2002-2024

**Filters Applied** (sequential):
1. **Year = 2023**: Filter to 2023 only → ~350,000 trips
2. **London region** (PSUStatsReg_B01ID = 8): → ~45,000
3. **Commuting purpose** (TripPurpose_B04ID = 1): → ~30,000
4. **Weekdays only** (Mon-Fri): → ~25,500
5. **Normal days** (TravelDayType = 1, exclude bank holidays): → ~25,000
6. **Choice set modes** (car, PT, bicycle): → ~25,000
7. **Data quality filters** (travel time 2-120 min, distance 0.2-50 miles): → **25,188**

**Final Sample**: 25,188 commuting trips (7.1% bicycle, 52.7% car, 40.2% PT)

#### Data Notes

✅ Weekday data only (Mon-Fri, normal days)  
✅ Multiple tables merged (trip + individual + household)  
✅ Weights included — W5 (trip weight), W3 (person weight)  
✅ Distance converted to km (original was miles)  
✅ Transfers calculated (for PT trips)  
✅ Missing income handled — Only quintiles 1-5 (no missing codes)  

---

## Column Harmonization

Both datasets use the same **22 core columns** for comparative analysis:

```python
shared_columns = [
    "person_id", "trip_id", "trip_purpose", "mode_detailed",
    "travel_time_min", "distance_km", "departure_hour",
    "weekday", "is_holiday", "age_band", "gender",
    "income_quintile", "has_driving_license", "n_cars_household",
    "weight_trip", "weight_person", "city", "is_peak", "chosen_mode",
    "n_transfers", "has_transfer", "n_legs"
]
```

This allows **stacking both datasets for pooled analysis**:
```python
# Add city identifier
df_amst['city'] = 'Amsterdam'
df_lnd['city'] = 'London'

# Stack for pooled analysis
df_combined = pd.concat([df_amst[shared_columns], 
                          df_lnd[shared_columns]], 
                         ignore_index=True)

print(df_combined.shape)  # (25,869, 22)
```

---

## How Data Was Created

### Amsterdam Pipeline
```
Raw ODiN .sav file (1.5M trips)
    ↓
Load with pyreadstat
    ↓
Apply sequential filters
    ↓
Recode variables (distance hm→km, income decile→quintile)
    ↓
Harmonize column names
    ↓
Standardize values (mode codes, gender codes, etc.)
    ↓
amst_processed.csv (681 trips)
```

### London Pipeline
```
Raw NTS .tab files (trip, individual, household)
    ↓
Load and filter to 2023 PSU IDs
    ↓
Merge tables on [PSUID, IndividualID, HouseholdID]
    ↓
Apply same sequential filters 
    ↓
Recode variables (distance miles→km, mode codes)
    ↓
Standardize column names & values
    ↓
lnd_processed.csv (25,188 trips)
```

---

## Using Weights in Analysis

Both datasets include **population expansion weights**. Use when analyzing population-level statistics.

### Amsterdam

- **`weight_trip`**: Trip-level weight (for trip volume estimates)
- **`weight_person`**: Person-level weight (for person characteristics)

### London

- **`W5`**: Trip weight (replaces `weight_trip`)
- **`W3`**: Person weight (household weight)

---

## Notes

✅ All data is cleaned and ready for analysis — No further preprocessing needed  
✅ Filters are fully documented — See "Filtering Applied" above  
✅ Both datasets use same schema — Allows comparative analysis  
✅ Weights are included — Use for population-level inference  
✅ No sensitive data — Only aggregated trip-level records  

---

## File Size Note

- **amst_processed.csv**: ~500 KB (681 rows × 31 columns)
- **lnd_processed.csv**: ~1.2 MB (25,188 rows × 28 columns)
