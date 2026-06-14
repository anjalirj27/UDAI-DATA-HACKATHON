# UDAI Data Hackathon - Aadhaar Data Analysis Project

## 📊 Project Overview

This project analyzes **Aadhaar enrollment, biometric, and demographic data** from the UDAI (Unique Identification Authority of India) across India's states and districts. The goal is to identify operational inefficiencies, equipment failures, and enrollment anomalies to improve Aadhaar enrollment performance nationwide.

## 🎯 Objectives

1. **Identify Operational Issues**: Detect districts with equipment failures and infrastructure gaps
2. **Analyze Enrollment Patterns**: Understand demographic trends and age-group distribution in enrollments
3. **Detect Anomalies**: Flag districts with unusual activity patterns (biometric blackholes, saturated zones, etc.)
4. **Provide Actionable Insights**: Generate reports for policymakers and field officers

## 📁 Repository Structure

```
UDAI-DATA-HACKATHON/
├── README.md                          # This file
├── Mergedataset.ipynb                 # Main analysis notebook combining all datasets
├── demographic.ipynb                  # Demographic data exploration and analysis
├── Final_Enrollment_file.ipynb         # Enrollment data processing and insights
├── Final_biometeric_file.ipynb         # Biometric data analysis
│
├── enrol_final_csv.csv                # Processed enrollment dataset
├── demographic_final_csv.csv          # Processed demographic dataset
├── biometric_cleaned_csv.csv          # Processed biometric dataset
│
└── api_data_aadhar_*/                 # Raw API data directories
    ├── api_data_aadhar_biometric/
    ├── api_data_aadhar_demographic/
    └── api_data_aadhar_enrolment/
```

## 📋 Datasets

### 1. **Enrollment Data** (`Final_Enrollment_file.ipynb`)
- **Source**: Aadhaar enrollment API data
- **Key Columns**: 
  - `date`: Date of enrollment activity
  - `state`: State information
  - `district`: District information
  - `pincode`: Postal code
  - `age_0_5`: Count of children (0-5 years) enrolled
  - `age_5_17`: Count of children (5-17 years) enrolled
  - `adult_count_enrol`: Count of adults enrolled
- **Size**: 277,868 records
- **Processing**: Aggregation by state/district, age-group analysis, total enrollment calculations

### 2. **Biometric Data** (`Final_biometeric_file.ipynb`)
- **Source**: Biometric update API data
- **Key Columns**:
  - `date`: Date of biometric update
  - `state`: State information
  - `district`: District information
  - `pincode`: Postal code
  - `bio_age_5_17`: Count of children with biometric updates
  - `adult_count_bio`: Count of adults with biometric updates
- **Size**: 1,765,631 records
- **Processing**: Aggregation by location, biometric coverage analysis

### 3. **Demographic Data** (`demographic.ipynb`)
- **Source**: Demographic update API data
- **Key Columns**:
  - `date`: Date of demographic update
  - `state`: State information
  - `district`: District information
  - `pincode`: Postal code
  - `demo_age_5_17`: Count of children with demographic updates
  - `adult_count_demo`: Count of adults with demographic updates
- **Size**: 958,662 records
- **Processing**: Demographic change tracking, data quality assessment

## 🔍 Key Analysis & Findings

### Analysis Workflow

#### Step 1: Data Standardization & Aggregation
```python
# Calculate total activity per category
total_bio = bio_age_5_17 + adult_count_bio
total_demo = demo_age_5_17 + adult_count_demo
total_enrol = age_0_5 + age_5_17 + adult_count_enrol
```

#### Step 2: Pincode-Level Master Performance Table
- Merged all three datasets on (state, district, pincode)
- Created 60,783 unique pincode records
- Calculated total activity across all three metrics

#### Step 3: Anomaly Detection & Categorization

**Four Key Anomaly Categories Identified:**

1. **Possible Equipment Failure (No Bio)** - 22,900 pincodes
   - High demographic activity but ZERO biometric updates
   - Suggests broken biometric devices or lack of infrastructure
   - `Condition`: `total_demo > 50 AND total_bio == 0`

2. **Saturated Population (Updates Only)** - 23,391 pincodes
   - High update activity but ZERO new enrollments
   - Indicates fully enrolled population; only maintenance occurring
   - `Condition`: `total_enrol == 0 AND (total_bio > 100 OR total_demo > 100)`

3. **Underperforming/Dead Stations** - 5,404 pincodes
   - Extremely low activity across all metrics
   - May indicate closed or inactive enrollment centers
   - `Condition`: `0 < total_activity < 5`

4. **Normal / Active Stations** - 9,088 pincodes
   - Healthy enrollment and update activity
   - Operating within expected parameters

### State-Level Performance Summary

**Top Performing States** (by total activity):
- Andhra Pradesh: 4,485,472 total activities
- Assam: 1,456,707 total activities
- Uttar Pradesh: 1,311,456 total activities
- Maharashtra: 1,289,445 total activities
- Karnataka: 997,223 total activities

**States with Most Equipment Issues:**
- Uttar Pradesh: 82 biometric blackholes
- Madhya Pradesh: 60 biometric blackholes
- Maharashtra: 51 biometric blackholes
- Karnataka: 49 biometric blackholes
- Bihar: 44 biometric blackholes

### District-Level Insights

**Critical Finding**: 906 districts identified with **Biometric Blackholes** across 38 states
**Critical Finding**: 860 districts with **Saturated Population** patterns
**Critical Finding**: 26 districts showing both equipment failure AND saturation simultaneously

**High-Risk Regions Requiring Immediate Attention**:
- All districts in Jammu & Kashmir (23 simultaneous anomalies)
- West Bengal South 24 Parganas: 214,540 demographic updates but 0 biometric activity
- Large states (UP, MP, MH) show distributed anomalies

## 📊 Visualizations Generated

### 1. **State-Level Comparison Chart**
- Bar chart showing Top 15 states by total activity
- Three metrics: Biometric Updates, Demographic Updates, New Enrollments
- Reveals urban-centric spikes and rural enrollment gaps

### 2. **District-Level Heatmap**
- Geospatial visualization of demographic-to-biometric ratio
- Identifies geographic hotspots of equipment failure
- Dark red areas indicate high concern regions

### 3. **Anomaly Distribution Charts**
- Pie charts showing proportion of each anomaly type
- State-wise breakdown of issues
- Trend visualization across regions

## 🚨 Data Quality Issues Identified

1. **Date Format Inconsistency**
   - Enrollment: DD-MM-YYYY format
   - Biometric & Demographic: YYYY-MM-DD format
   - ⚠️ Risk: Silent errors in time-series analysis

2. **State Naming Variants**
   - "Andaman And Nicobar Islands" vs "Andaman and Nicobar Islands"
   - "Dadra And Nagar Haveli" merged with "Daman And Diu"
   - ⚠️ Risk: Duplicate or lost records in aggregation

3. **Missing/Incomplete Data**
   - Daman and Diu lacks biometric coverage
   - Some districts have zero records for entire categories
   - ⚠️ Risk: Geographic bias in analysis

4. **Pincode Variations**
   - Some districts recorded under multiple pincode formats
   - May inflate unique location count artificially

## 📊 Reports Generated

### Excel Report: `UIDAI_Anomaly_Report.xlsx`
Contains three detailed sheets:

1. **Biometric_Blackholes Sheet**
   - 906 districts with zero biometric activity
   - State, district, demographic count, enrollment count
   - Critical for equipment procurement/replacement planning

2. **Saturated_Districts Sheet**
   - 860 districts with only update activity
   - Identifies mature enrollment zones
   - Useful for resource reallocation decisions

3. **Adult_Enrollment_Outliers Sheet**
   - 4 districts with unusual adult enrollment patterns
   - May indicate data quality issues or special programs

## 🔧 Tools & Technologies Used

- **Python 3**: Core analysis language
- **Pandas**: Data manipulation and aggregation
- **NumPy**: Numerical computations
- **Matplotlib & Seaborn**: Data visualization
- **GeoPandas**: Geographic data analysis and mapping
- **Excel/Openpyxl**: Report generation
- **Google Colab**: Development and execution environment

## 📈 Statistical Summary

| Metric | Value |
|--------|-------|
| Total Enrollment Records | 277,868 |
| Total Biometric Records | 1,765,631 |
| Total Demographic Records | 958,662 |
| Unique Pincodes Analyzed | 60,783 |
| States Covered | 38 |
| Anomalous Pincodes | 51,783 (85.2%) |
| Clean Pincodes | 9,088 (14.8%) |

## 🎓 Key Insights & Recommendations

### Insights

1. **Equipment Crisis**: 22,900 pincodes (37.7%) have biometric infrastructure failures
2. **Geographic Inequality**: Urban centers show 10x higher enrollment than rural areas
3. **Population Saturation**: 23,391 pincodes (38.5%) show completed enrollment with only updates
4. **Data Quality Concerns**: Naming inconsistencies suggest integration issues across regional databases
5. **Age-Group Anomalies**: Adult enrollments exceed child enrollments in some districts (unusual pattern)

### Recommendations

1. **Immediate Action**:
   - Deploy technicians to 906 biometric blackhole districts
   - Audit equipment status in high-anomaly zones

2. **Medium-term**:
   - Standardize state naming across all systems
   - Implement cross-system validation checks
   - Consolidate regional databases with consistent schemas

3. **Long-term**:
   - Shift focus from saturated zones to underserved rural areas
   - Establish mobile enrollment units for low-activity zones
   - Create predictive maintenance schedule for equipment

## 🤝 Contributing

This is a hackathon project. For improvements or additional analysis:
1. Create a new notebook with clear naming
2. Document your analysis methodology
3. Update this README with new findings

## 📝 License

This project uses publicly available UDAI data for analytical purposes under fair use.

## 👤 Author

**Anjali RJ**  
Repository: [anjalirj27/UDAI-DATA-HACKATHON](https://github.com/anjalirj27/UDAI-DATA-HACKATHON)

---

## 🚀 Quick Start

1. **Clone the repository**:
   ```bash
   git clone https://github.com/anjalirj27/UDAI-DATA-HACKATHON.git
   cd UDAI-DATA-HACKATHON
   ```

2. **Install dependencies**:
   ```bash
   pip install pandas numpy matplotlib seaborn geopandas openpyxl
   ```

3. **Run the analysis**:
   - Open `Mergedataset.ipynb` in Jupyter Notebook or Google Colab
   - Execute cells sequentially
   - View generated visualizations and reports

4. **Review reports**:
   - Check `UIDAI_Anomaly_Report.xlsx` for detailed findings
   - Review visualization outputs in notebook cells

---

**Last Updated**: January 20, 2026  
**Data Range**: March 2025 onwards  
**Total Size**: ~130 MB (including raw data and processed CSVs)
