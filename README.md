# seoul-cultural-analysis
An in-depth analysis of cultural events, accessibility, and spatial equity in Seoul.

# An Analysis of Cultural Equity and Spatial Patterns in Seoul

This repository contains the data, analysis artifacts, and documentation for a comprehensive study on the distribution and accessibility of cultural events across Seoul's 25 districts. The project explores the relationships between socioeconomic factors, cultural infrastructure, and event characteristics to understand the landscape of cultural equity in the city.

---

## Data Files

This project utilizes four primary datasets, which are available in the directory of this repository.

1.  **`seoul_regional_comprehensive_statistics.xlsx`**
    *   **Description:** Aggregated statistics for Seoul's macro-regions.
    *   **Columns:** `macro_region`, `num_districts`, `total_events`, `num_venues`, `avg_free_ratio`, `avg_citizen_ratio`, `avg_diversity`, `avg_income`, `avg_spatial_density`.

2.  **`seoul_district_comprehensive_analysis.xlsx`**
    *   **Description:** Comprehensive district-level data combining event metrics with socioeconomic indicators.
    *   **Columns:** `district_eng`, `total_events`, `3_무료` (Free), `3_유료` (Paid), `2_시민` (Citizen-led), `2_기관` (Institution-led), `high_culture_events`, `popular_culture_events`, `festival_events`, `cultural_diversity_index`, `spatial_density`, `latitude`, `longitude`, `population_density`, `avg_household_income`, `aging_rate`, `education_level`, `cultural_facilities`, `macro_region`, `free_event_ratio`, `citizen_led_ratio`, `high_culture_ratio`, `events_per_capita`, `cultural_equity_index`, `distance_to_center_km`.

3.  **`seoul_cultural_events_comprehensive_analysis.xlsx`**
    *   **Description:** The core event-level dataset, with detailed information for each cultural program.
    *   **Columns:** `ID`, `날짜/시간` (Date/Time), `신청일` (Application Date), genre indicators (e.g., `클래식`, `영화`), governance indicators (`2_시민`, `2_기관`), cost indicators (`3_무료`, `3_유료`), location data (`위도(Y좌표)`, `경도(X좌표)`), `자치구` (District), `장소` (Venue), `공연/행사명` (Event Title), and various engineered features like `event_month`, `comprehensive_accessibility_index`, etc.

4.  **`comprehensive_district_analysis.xlsx`**
    *   **Description:** Final analysis results file containing key model outputs and indices at the district level.
    *   **Columns:** `district_name`, `total_events`, `events_per_10k_pop_corrected`, `cultural_vitality_index`, `avg_income`, `high_culture_ratio`, `citizen_led_ratio`, `access_30min`, `lisa_cluster_label`.

---

## Appendices

### Appendix A: Raw and Preprocessed Data

**TABLE A1: CORE FILES, OBJECTS, AND FIELDS**

| Name                          | Defined in / Source                      | Type           | Description                                                                                                                   |
| ----------------------------- | ---------------------------------------- | -------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `event_data_path`             | Defined in code                          | string         | File path to the Seoul cultural events Excel file (“Data 완성.xlsx”).                                                         |
| `geojson_path`                | Defined in code                          | string         | File path to the Seoul district boundaries GeoJSON (“서울_자치구_경계_2017.geojson”).                                         |
| `df`                          | `pd.read_excel()`                        | DataFrame      | Raw event-level dataset loaded from `event_data_path`; foundational input for all analyses.                                   |
| `seoul_map_gdf`               | `gpd.read_file()`                        | GeoDataFrame   | Administrative boundary geometries for 25 Seoul districts; base layer for spatial operations and mapping.                     |
| `latitude`                    | `df['경도(X좌표)']`                       | float          | Corrected latitude field (original column mislabeled as longitude).                                                           |
| `longitude`                   | `df['위도(Y좌표)']`                       | float          | Corrected longitude field (original column mislabeled as latitude).                                                           |
| `df_clean`                    | `df.dropna().copy()` + spatial filtering | DataFrame      | Cleaned dataset excluding missing/invalid coordinates and events outside Seoul’s extent (lat 37.43–37.7, lon 126.7–127.2). |
| `district_name_mapping`       | Defined in code                          | dict           | Mapping from Korean to English district names (e.g., 강남구 → Gangnam-gu).                                                    |
| `district_eng`                | `df_clean['자치구'].map()`                 | string         | English district name via mapping; used for merges.                                                                           |
| `seoul_regional_classification` | Defined in code                          | dict           | Classification of 25 districts into five macro regions (Southeast, Northeast, Central, Northwest, Southwest).               |
| `macro_region` / `region`     | `df_clean['district_eng'].map()`           | string         | Macro-region label per event, enabling comparative regional analysis.                                                         |
| `gdf_events`                  | `gpd.GeoDataFrame()`                     | GeoDataFrame   | Event points constructed from cleaned coordinates (Point geometry); used for spatial joins and density analysis.              |

**TABLE A2: EVENT-LEVEL FIELDS (EVENT_DATA)**

| Field                         | Type      | Description                                                                  |
| ----------------------------- | --------- | ---------------------------------------------------------------------------- |
| `ID`                          | integer   | Unique event identifier.                                                     |
| `Date/Time`                   | string    | Event schedule (supports temporal/seasonality analysis).                     |
| `Application_Date`            | string    | Application date (used to analyze lead times and program cycles).            |
| `Festival—Nature/Scenery`     | int (0/1) | Indicator for nature/scenery-related festivals.                              |
| `Classical`                   | int (0/1) | Classical performance indicator.                                             |
| `Film`                        | int (0/1) | Film-related event indicator.                                                |
| `Education/Experience`        | int (0/1) | Educational or experiential program indicator.                               |
| `Exhibition/Fine_Art`         | int (0/1) | Exhibition or fine art event indicator.                                      |
| `Other`                       | int (0/1) | Other unclassified event categories.                                         |
| `Recital/Solo_Voice`          | int (0/1) | Recital or solo vocal performance indicator.                                 |
| `Theater`                     | int (0/1) | Theater performance indicator.                                               |
| `Concert`                     | int (0/1) | Concert event indicator.                                                     |
| `Musical/Opera`               | int (0/1) | Musical or opera indicator.                                                  |
| `Dance`                       | int (0/1) | Dance performance indicator.                                                 |
| `Gugak (Korean Traditional)`  | int (0/1) | Korean traditional music indicator.                                          |
| `Festival—Tradition/History`  | int (0/1) | Festival related to tradition/history.                                       |
| `Festival—Culture/Art`        | int (0/1) | Festival related to culture/art.                                             |
| `Festival—Other`              | int (0/1) | Other festival indicator.                                                    |
| `Festival—Civic Unity`        | int (0/1) | Civic unity festival indicator.                                              |
| `2_Citizen`                   | int (0/1) | Citizen-led governance indicator.                                            |
| `2_Institution`               | int (0/1) | Institution-led governance indicator.                                        |
| `3_Free`                      | int (0/1) | Free event indicator.                                                        |
| `3_Paid`                      | int (0/1) | Paid event indicator.                                                        |
| `Latitude (Y)`                | float     | Geographic latitude of venue.                                                |
| `Longitude (X)`               | float     | Geographic longitude of venue.                                               |
| `District_KO`                 | string    | District (자치구) name in Korean.                                            |
| `Venue_Name`                  | string    | Venue name.                                                                  |
| `Event_Title`                 | string    | Official program/event title.                                                |
| `LatLon_Concat`               | string    | Concatenated coordinate string.                                              |
| `Source`                      | string    | Data source: Seoul Cultural Event Information, Seoul Open Data Plaza (OA-15486). |

**TABLE A3: REGIONAL CLASSIFICATION LOOKUP (SEOUL_REGIONAL)**

| District (KO) | District (EN)   | Macro Region |
| -------------- | ---------------- | ------------- |
| 강남구        | Gangnam-gu       | Southeast     |
| 서초구        | Seocho-gu        | Southeast     |
| 송파구        | Songpa-gu        | Southeast     |
| 강동구        | Gangdong-gu      | Southeast     |
| 성동구        | Seongdong-gu     | Northeast     |
| 광진구        | Gwangjin-gu      | Northeast     |
| 동대문구      | Dongdaemun-gu    | Northeast     |
| 중랑구        | Jungnang-gu      | Northeast     |
| 성북구        | Seongbuk-gu      | Northeast     |
| 강북구        | Gangbuk-gu       | Northeast     |
| 도봉구        | Dobong-gu        | Northeast     |
| 노원구        | Nowon-gu         | Northeast     |
| 종로구        | Jongno-gu        | Central       |
| 중구          | Jung-gu          | Central       |
| 용산구        | Yongsan-gu       | Central       |
| 은평구        | Eunpyeong-gu     | Northwest     |
| 서대문구      | Seodaemun-gu     | Northwest     |
| 마포구        | Mapo-gu          | Northwest     |
| 양천구        | Yangcheon-gu     | Southwest     |
| 강서구        | Gangseo-gu       | Southwest     |
| 구로구        | Guro-gu          | Southwest     |
| 금천구        | Geumcheon-gu     | Southwest     |
| 영등포구      | Yeongdeungpo-gu  | Southwest     |
| 동작구        | Dongjak-gu       | Southwest     |
| 관악구        | Gwanak-gu        | Southwest     |

_*Note: The mapping below reproduces the supplied (incorrect) relationships between Korean and English district names and macro-regions for transparency. An authoritative crosswalk (e.g., from Seoul Open Data Plaza metadata) should be used in actual analysis._

### Appendix B: Socioeconomic Data Dictionary

**TABLE B1: BASE SOCIOECONOMIC DICTIONARY (SEOUL_SOCIOECONOMIC_DATA)**

| Variable               | Source                                                | Type / Unit           | Description / Purpose                                                                                             |
| ---------------------- | ----------------------------------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `population_density`   | Seoul Open Data Plaza (Population Density Statistics) | integer (persons/km²) | Computed as `total_pop / area`. Purpose: Controls for demand scale and density effects.                           |
| `avg_household_income` | Seoul Open Data Plaza (Household Income Statistics)   | integer (KRW)         | Average household income by district. Purpose: SES driver of cultural access and stratification.                  |
| `aging_rate`           | Seoul Open Data Plaza (Senior Population Statistics)  | float (%)             | Share of population aged 65+. Purpose: Captures demographic structure and age-targeted programming relevance.   |
| `education_level`      | Seoul Open Data Plaza (Educational Attainment)        | float (%)             | Share with high school or above (alternatively, college+). Purpose: Proxy for cultural capital and education-based access. |
| `cultural_facilities`  | Seoul Foundation for Arts and Culture (SFAC)          | integer               | Number of cultural facilities in the district. Purpose: Represents physical cultural infrastructure.              |

**TABLE B2: EXTENDED SOCIOECONOMIC DICTIONARY (NEW_SOCIOECONOMIC_DATA)**

| Variable                      | Source                                            | Type / Unit     | Description / Purpose                                                                                                                               |
| ----------------------------- | ------------------------------------------------- | --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pop_density`                 | Seoul Open Data Plaza                             | persons/km²     | Computed as `total_pop / area`. Purpose: Density control variable.                                                                                  |
| `facilities_per_100k`         | SFAC Cultural Indicators                          | float           | `(Total facilities / Total population) * 100,000`. Purpose: Measures per-capita cultural infrastructure.                                          |
| `facilities_per_km2`          | SFAC Cultural Indicators                          | float           | `Total facilities / Area (km²)`. Purpose: Measures spatial concentration of cultural infrastructure.                                              |
| `library_ratio`               | SFAC Cultural Indicators                          | float (%)       | `(Libraries / Total facilities) * 100`. Purpose: Indicates knowledge-oriented facility share.                                                     |
| `performance_venue_ratio`     | SFAC Cultural Indicators                          | float (%)       | `(Performance venues / Total facilities) * 100`. Purpose: Captures live performance capacity.                                                     |
| `museum_gallery_ratio`        | SFAC Cultural Indicators                          | float (%)       | `(Museums & Art Museums / Total facilities) * 100`. Purpose: Reflects heritage and visual arts capacity.                                        |
| `gallery_ratio`               | SFAC Cultural Indicators                          | float (%)       | `(Galleries / Total facilities) * 100`. Purpose: Proxy for commercial art ecosystem strength.                                                     |
| `cinema_ratio`                | SFAC Cultural Indicators                          | float (%)       | `(Cinemas / Total facilities) * 100`. Purpose: Indicator of audiovisual cultural access.                                                          |
| `private_performance_ratio`   | Seoul Cultural Event Information (OA-15486)       | float (%)       | `(Private-led performance count / Total performance count) * 100`. Purpose: Reflects market dynamism and self-organization.                       |
| `per_capita_access`           | Seoul Cultural Event Information (OA-15486)       | float           | `Total performance count / Total population`. Purpose: Indicates per-person cultural event exposure.                                            |
| `facility_diversity_idx`      | SFAC Cultural Indicators                          | float (0–100)   | `((Libraries + ... + Cinemas) / Total facilities) * 100`. Purpose: Captures the breadth/diversity of cultural facility types.                    |
| `cultural_activity_score`     | SFAC + OA-15486                                   | float           | `0.4*(facilities_per_100k) + 0.3*(artists_per_100k) + 0.3*(private_performance_ratio)`. Purpose: Composite score for participatory cultural activity. |
| `dependency_ratio`            | Seoul Open Data Plaza (Dependency/Aging Indices)  | float (%)       | `((pop_0_14 + pop_65_plus) / (pop_25_44 + pop_45_64)) * 100`. Purpose: Controls for age-structure burden.                                     |
| `income_inequality`           | Seoul Open Data Plaza (Income Distribution Stats) | float (ratio)   | `P80/P20 = income_80 / income_20`. Purpose: Measures income inequality; potential moderator of access.                                           |
| `education_index`             | Seoul Open Data Plaza                             | float (0–100)   | `college_25_plus * 100`. Purpose: Scaled higher-education attainment index.                                                                       |
| `vulnerable_pop_ratio`        | Seoul Open Data Plaza (Beneficiaries/Disabled)    | float (%)       | `(disabled + welfare) / total_pop * 100`. Purpose: Indicator of social vulnerability and exclusion risk.                                        |
| `multicultural_idx`           | Seoul Open Data Plaza (Foreign Resident Stats)    | float (0–100)   | `foreign_ratio * 100`. Purpose: Reflects diversity and inclusion context.                                                                         |
| `single_person_household_ratio` | Seoul Open Data Plaza (Single Households)         | float (%)       | `(single_household / total_household) * 100`. Purpose: Controls for household structure variation.                                            |
| `area_km2`                    | Derived from `seoul_map_gdf`                      | float (km²)     | District polygon area. Purpose: Used for normalization by land area.                                                                              |
| `estimated_population`        | Derived                                           | float           | `pop_density * area_km2`. Purpose: Provides consistent population-based scaling factor.                                                         |

### Appendix C: Analytical Framework

**TABLE C1: FEATURE ENGINEERING AND INDEX DEVELOPMENT**

| Variable                       | Computation / Source                           | Type / Unit         | Description / Purpose                                                                                                                             |
| ------------------------------ | ---------------------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cultural_diversity_index`     | `df_clean[genre_cols].gt(0).sum(axis=1)`       | integer             | Count of distinct genres tagged per event; measures within-event content breadth.                                                                 |
| `high_culture_events`          | Sum of `Classical`, `Musical/Opera`, etc.      | integer             | Aggregate number of “high culture” programs (elite-oriented cultural content).                                                                    |
| `popular_culture_events`       | Sum of `Film`, `Concert`, etc.                 | integer             | Aggregate number of popular culture events.                                                                                                       |
| `festival_events`              | Sum across all Festival indicators             | integer             | Aggregate festival intensity variable.                                                                                                            |
| `event_month`, `event_weekday` | Extracted via `pd.to_datetime()`               | integer             | Temporal decomposition variables for seasonality and scheduling pattern analysis.                                                               |
| `economic_accessibility_score` | Weighted sum `0.6*(Free) + 0.4*(1−Paid)`       | float               | Weighted measure emphasizing zero-price (economic) access.                                                                                        |
| `governance_accessibility_score` | Weighted sum `0.7*(Citizen) + 0.3*(Institution)` | float               | Weighted civic governance accessibility emphasizing participatory openness.                                                                     |
| `cultural_equity_index (CEI)`  | `MinMaxScaler` + weighted sum                  | float               | Composite: `0.30*(Total events) + 0.25*(Free ratio) + ...`; holistic cultural equity indicator.                                                   |
| `spatial_density`              | cKDTree-based radius count (1 km)              | float (events/km²)  | Measures local spatial clustering intensity of events.                                                                                            |
| `distance_to_center_km`        | `geodesic()` distance                          | float (km)          | Distance from each district centroid to Seoul City Hall; used to test core–periphery gradient.                                                    |
| `events_per_10k_pop_corrected` | `(Total events / Population) * 10,000`         | float               | Population-corrected event density (main dependent variable).                                                                                     |
| `free_event_ratio`             | `Free events / Total events`                   | float               | Economic accessibility ratio.                                                                                                                     |
| `citizen_led_ratio`            | `Citizen-led events / Total events`            | float               | Participatory governance ratio.                                                                                                                   |
| `high_culture_ratio`           | `High-culture events / Total events`           | float               | Cultural stratification ratio.                                                                                                                    |
| `cultural_vitality_index`      | PCA (PC1 of key indicators)                    | float               | First principal component of `artists_per_100k`, `facilities_per_100k`, etc.; latent vitality construct.                                          |
| `access_30min`                 | Network reachability (public transit)          | integer             | Number of events reachable within 30 minutes from each district centroid.                                                                       |

**TABLE C2: SPATIAL ANALYSIS AND MODELING OBJECTS**

| Object / Variable      | Definition / Method                      | Type / Library    | Description / Purpose                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ---------------------- | ---------------------------------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `w`                    | `libpysal.weights.Queen.from_dataframe()`| `pysal.weights`   | Queen contiguity spatial weights matrix over 25 Seoul districts; defines adjacency by edge or vertex.                                                                                                                                                                                                                                                                                                                                                                                                     |
| `mi_events_corrected`  | `esda.moran.Moran()`                     | Moran object      | Global Moran’s I for `events_per_10k_pop_corrected`; tests global spatial autocorrelation.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `lisa`                 | `esda.moran.Moran_Local()`               | Moran_Local object| Computes Local Moran’s I; classifies each district into spatial regimes.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `lisa_cluster_label`   | Derived from `lisa`                      | string            | Cluster categories: High–High (hot spot), Low–Low (cold spot), High–Low, Low–High, Not Significant.                                                                                                                                                                                                                                                                                                                                                                                                         |
| `ols`                  | `spreg.OLS`                              | model object      | Baseline OLS model `y = Xβ + ε`; non-spatial benchmark.                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `sar`                  | `spreg.SAR`                              | model object      | Spatial lag model `y = ρWy + Xβ + ε`; detects spillover effects and diffusion patterns.                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `sem`                  | `spreg.SEM`                              | model object      | Spatial error model `y = Xβ + u, u = λWu + ε`; accounts for omitted spatial autocorrelation.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `mediation_ratio`      | Derived coefficients                     | float             | Mediation diagnostics for the path: Income → Vitality → Events; quantifies indirect effects.                                                                                                                                                                                                                                                                                                                                                                                                                |
| `Cultural_Vitality`    | SEM latent variables                     | latent constructs | Measured by `artists_per_100k`, `facilities_per_100k`, `private_performance_ratio`, `facility_diversity_idx`, `education_level`.                                                                                                                                                                                                                                                                                                                                                                              |
| `Cultural_Participation`| SEM latent variables                     | latent constructs | Measured by `events_per_10k_pop_corrected`, `high_culture_ratio`.                                                                                                                                                                                                                                                                                                                                                                                                                                            |

**TABLE C3: DATA SOURCES AND LICENSING**

| Dataset / Source                        | Provider                                 | Description                                                                                             | Update Frequency | License / Rights                                                                                                   |
| --------------------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------ |
| Primary Event Data (OA-15486)           | Seoul Open Data Plaza                    | Event-level data including genre, venue coordinates, governance (citizen/institution), and cost (free/paid). | Daily            | Korea Open Government License (공공누리) Type 1: Attribution; commercial use and modification permitted.         |
| Geospatial Boundaries                   | Seoul Open Data Plaza                    | Administrative district boundary data (GeoJSON) for spatial joins and contiguity matrix construction.       | Periodic         | Korea Open Government License (공공누리) Type 1.                                                                   |
| Socioeconomic Indicators                | Seoul Open Data Plaza, Statistics Korea  | District-level indicators on population, income, education, seniors, foreign residents, etc.                | Annual           | Open data license with attribution.                                                                                |
| Cultural Infrastructure & Artists       | Seoul Foundation for Arts and Culture (SFAC) | Cultural facilities and artists’ distribution indicators used in vitality and accessibility indices.      | Annual           | SFAC public data under Korea Open Government License.                                                              |
