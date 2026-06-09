# Student Lifestyle Dataset — End-to-End Data Analysis Project

> A complete data analyst workflow: raw CSV → quality audit → Excel cleaning → pivot dashboard → strategic recommendations

https://docs.google.com/spreadsheets/d/e/2PACX-1vQzlE_sNIj1enmltWT_2va2nHYZdwvtYLgnLoUJdUyJzdrlwvNvSPFZd1-CKHeDF-xmmbGiOK6jnpi4/pubhtml


![Excel](https://img.shields.io/badge/Microsoft_Excel-Cleaned_%26_Dashboarded-217346?style=flat-square&logo=microsoftexcel&logoColor=white)
![Status](https://img.shields.io/badge/Project_Status-Complete-2E7D32?style=flat-square)
![Records](https://img.shields.io/badge/Dataset-2%2C000_students-1565C0?style=flat-square)

---

## Overview

This project demonstrates a production-quality data analyst workflow applied to a real student lifestyle dataset. Every phase of professional analysis is covered — from systematic data quality auditing through Excel-formula-based cleaning, statistical exploration, interactive dashboard development, and evidence-based strategic recommendations.

**Business question:** What lifestyle factors most strongly predict student GPA and stress levels? What can academic institutions do about it?

**Dataset:** 2,000 students × 8 features. All five daily-hour columns sum to exactly 24 (fixed time budget per student).

---

## Key Findings at a Glance

| Finding | Metric | Insight |
|---|---|---|
| Study → GPA | r = +0.734 | Strongest single predictor of GPA |
| Study → Stress | r = +0.740 | Same behaviour drives grades AND stress |
| High stress rate | 51.4% | Majority experience, not a minority problem |
| Exercise → GPA | r = −0.341 | Time displacement, not a health effect |
| Sleep → GPA | r = −0.004 | Near-zero — sleep predicts wellbeing, not grades |
| Low-stress avg GPA | 2.82 | Lowest of all three groups — under-engagement risk |

---

## Project Structure

```
student-lifestyle-analysis/
├── data/
│   ├── student_lifestyle_dataset.csv          # Raw source data
│   └── student_lifestyle_cleaned.csv          # Cleaned output (Python)
├── excel/
│   ├── student_lifestyle_excel.xlsx           # 5-sheet cleaning workbook
│   │   ├── Raw Data                           # Original unmodified rows
│   │   ├── Cleaned Data                       # All fixes as live formulas
│   │   ├── Analysis                           # AVERAGEIF / QUARTILE / CORREL
│   │   ├── Data Quality Report                # 10-check audit table
│   │   └── Formula Reference                  # All 19 cleaning formulas
│   └── student_dashboard_excel.xlsx           # 9-chart Excel dashboard
├── dashboard/
│   └── student_lifestyle_dashboard.html       # Responsive interactive dashboard
├── clean.py                                   # Python cleaning script
├── eda.ipynb                                  # Exploratory analysis notebook
├── requirements.txt
└── README.md
```

---

## Dataset

| Column | Type | Description |
|---|---|---|
| Student_ID | int | Unique student identifier |
| Study_Hours_Per_Day | float | Daily hours spent studying |
| Extracurricular_Hours_Per_Day | float | Daily extracurricular hours |
| Sleep_Hours_Per_Day | float | Daily sleep hours |
| Social_Hours_Per_Day | float | Daily social hours |
| Physical_Activity_Hours_Per_Day | float | Daily physical exercise hours |
| GPA | float | Grade point average — range: 2.24 to 4.0 |
| Stress_Level | string | Categorical: Low / Moderate / High |

All five hour columns sum to exactly 24.0 per student.

---

## Phase 1 — Data Quality Audit

A 10-point systematic audit before any cleaning:

| # | Check | Finding | Result |
|---|---|---|---|
| 1 | Missing values | 0 nulls across 16,000 cells | ✅ PASS |
| 2 | Duplicate rows | 0 exact duplicates | ✅ PASS |
| 3 | Stress_Level format | Consistent — High / Moderate / Low only | ✅ PASS |
| 4 | GPA range [0–4] | All values in range [2.24, 4.0] | ✅ PASS |
| 5 | Negative hour values | Zero negatives across all 5 columns | ✅ PASS |
| 6 | Float precision artifact | 393 rows: sum = 24.000000000000004 | ⚠️ MINOR |
| 7 | Physical activity outliers | 5 rows exceed IQR upper fence (>11.65h) | ⚠️ FLAG |
| 8 | GPA low outliers | 3 rows below IQR lower fence (<2.25) | ⚠️ FLAG |
| 9 | Zero-hour activity rows | 50 students with zeros in activity cols | ℹ️ NOTE |
| 10 | Stress_Level encoding | String type — ordinal needed for modelling | ℹ️ NOTE |

---

## Phase 2 — Data Cleaning

### Excel (live formulas — Cleaned Data sheet references Raw Data)

| Operation | Formula | Purpose |
|---|---|---|
| Round hours | `=ROUND('Raw Data'!B4, 1)` | Fix IEEE 754 float artifact |
| Cap GPA at 4.0 | `=ROUND(MIN('Raw Data'!G4, 4), 2)` | Defensive upper-bound guard |
| Normalise Stress_Level | `=PROPER(TRIM('Raw Data'!H4))` | Strip spaces, fix capitalisation |
| Encode ordinal | `=IF(H4="High",3,IF(H4="Moderate",2,IF(H4="Low",1,"Unknown")))` | Numeric for Pearson correlation |
| Validate hours | `=ROUND(B4+C4+D4+E4+F4, 1)` | Confirm 24h budget integrity |
| PA outlier flag | `=IF(F4>Analysis!$B$16,"OUTLIER","OK")` | IQR method — upper fence |
| GPA outlier flag | `=IF(G4 Q3 + 1.5 * (Q3 - Q1)

def iqr_flag_lower(series):
    Q1, Q3 = series.quantile([0.25, 0.75])
    return series < Q1 - 1.5 * (Q3 - Q1)

df['PA_Outlier_Flag']  = iqr_flag_upper(df['Physical_Activity_Hours_Per_Day'])
df['GPA_Outlier_Flag'] = iqr_flag_lower(df['GPA'])
df['Total_Daily_Hours'] = df[hour_cols].sum(axis=1).round(1)

df.to_csv('data/student_lifestyle_cleaned.csv', index=False)
print(f"Saved {len(df)} rows, {df.shape[1]} columns")

---

## Phase 3 — Analysis

### Stress level distribution

| Group | Count | Share |
|---|---|---|
| High | 1,029 | 51.4% |
| Moderate | 674 | 33.7% |
| Low | 297 | 14.8% |

### Average lifestyle profile by stress group

| Metric | High Stress | Moderate Stress | Low Stress |
|---|---|---|---|
| Avg GPA | **3.26** | 3.03 | 2.82 |
| Avg Study Hrs/Day | **8.39h** | 6.97h | 5.47h |
| Avg Sleep Hrs/Day | 7.05h | 7.95h | **8.06h** |
| Avg Physical Act | 3.96h | 4.34h | **5.58h** |
| Avg Social Hrs | 2.63h | 2.74h | **2.89h** |
| Avg Extracurricular | 1.98h | 2.01h | 1.99h |

### Correlations with GPA (Pearson r)

| Variable | r | Interpretation |
|---|---|---|
| Study_Hours_Per_Day | **+0.734** | Strongest positive predictor of GPA |
| Stress_Level_Encoded | **+0.550** | Higher stress group → higher GPA |
| Physical_Activity_Hours | **−0.341** | Time displacement effect — not harmful |
| Social_Hours_Per_Day | −0.086 | Weak negative |
| Extracurricular_Hours | −0.032 | Negligible |
| Sleep_Hours_Per_Day | −0.004 | Essentially zero |

### Five key insights

**1. Studying simultaneously maximises GPA and stress.**
Study hours carry the strongest positive correlations with both GPA (r = +0.734) and stress level (r = +0.740). The performance-wellbeing tradeoff is not anecdotal — it is quantified. Pushing students to study more hours without also teaching efficient study methods is pushing them toward both higher grades and higher distress.

**2. High stress is the dominant student experience.**
51.4% of students are classified High stress. The distribution is not symmetric — it skews heavily toward stress. Institutions treating student stress as an outlier or minority issue are systematically misdiagnosing the scope of the problem.

**3. Physical activity competes with GPA through time displacement.**
The negative correlation between exercise and GPA (r = −0.341) is not a finding about health — it is a finding about scheduling. In a fixed 24-hour budget, every hour of exercise displaces a potential study hour. The implication is institutional: scheduled exercise (mandatory wellness blocks, PE credit) removes the perceived trade-off entirely.

**4. Sleep does not predict GPA — it predicts stress.**
Sleep-GPA correlation: −0.004 (negligible). Sleep-stress correlation: −0.30 (moderate). The "sleep more for better grades" messaging widely used by academic institutions is not supported by this data. The evidence supports "sleep more for lower stress" — a fundamentally different and more accurate frame.

**5. Low-stress students are academically at risk.**
Low-stress students have the lowest average GPA (2.82) of all three groups. This is counter-intuitive but statistically clear. They are not thriving — they are likely under-engaged. Standard GPA-based early-alert systems will flag them only after significant academic decline. Engagement metrics (attendance, activity hours, social connection) are better leading indicators for this group.

---

## Phase 4 — Dashboard

### Excel dashboard (9 charts + ChartData sheet)

| Chart | Type | ChartData Range |
|---|---|---|
| Stress distribution | Doughnut (60% hole) | A3:B6 |
| GPA & Study Hrs by group | Clustered column, dual Y-axis | A8:C12 |
| GPA distribution | Histogram-style column | A22:B30 |
| Study hours distribution | Gradient column | A40:B47 |
| Study vs GPA scatter | XY scatter — 3 series | A57:C242 |
| Group comparison table | Formatted with mini bars | Dashboard sheet |

### Interactive web dashboard

Single-page responsive HTML / CSS / Chart.js dashboard:
- Stress-level filter updates scatter highlight and donut emphasis in real time
- 6 KPI metric cards
- All 5 chart types from the Excel version
- 4 filters
- Mobile-responsive layout

---

## Strategic Recommendations

Based on the statistical findings, five evidence-based recommendations for academic institutions:

1. **Teach efficient study techniques.** The data shows that raw study hours drive GPA, but at a proportional stress cost. Teaching spaced repetition, active recall, and the Pomodoro technique enables GPA improvements without requiring proportionally more hours — bending the GPA-stress curve.

2. **Reframe sleep promotion around stress, not grades.** The data does not support a sleep-GPA link (r = −0.004). Continuing to market sleep for grades is misleading. The evidence supports sleep for stress reduction (r = −0.30) — frame it accordingly.

3. **Institutionalise physical activity scheduling.** Voluntary exercise is consistently crowded out by academic pressure in a fixed-hour budget. Mandatory wellness periods, PE credit integration, or subsidised gym time removes the perceived study-vs-fitness trade-off without requiring students to choose.

4. **Target Low-stress / Low-GPA students with proactive outreach.** This is the under-engaged group. Average GPA 2.82, average study hours 5.47/day. They are not yet failing, but trajectory is poor. Early coaching intervention before academic crisis is significantly cheaper than dropout recovery.

5. **Audit social isolation signals.** 18 students report zero social hours. This is a known mental health risk factor independent of academic metrics. Cross-referencing with counselling intake and attendance records should be standard.

---

## Tech Stack

| Tool | Version | Purpose |
|---|---|---|
| openpyxl | 3.x | Excel workbook generation |
| Microsoft Excel | 365 | Live formula cleaning, pivot tables, charts |


---

## Author

**Adeking** — Data Analyst

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](linkedin.com/in/akehinmi-adeshola/)
[![Twitter](https://img.shields.io/badge/Twitter-Follow-000000?style=flat-square&logo=x)](https://x.com/codeMadeIt)

---

*Dataset: student_lifestyle_dataset.csv · 2,000 records · 8 features · All hours sum to 24.0 per student*
