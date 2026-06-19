
1️⃣ Repo Structure
UOIE/
│
├── README.md
├── LICENSE.md
├── requirements.txt
├── data/
│   ├── tasks.csv
│   ├── vendors.csv
│   ├── overrides.csv
│   └── costs.csv
├── notebooks/
│   └── demo_v1.0.ipynb
├── src/
│   ├── models.py
│   ├── drift.py
│   ├── narrative.py
│   └── dashboard.py
└── reports/
    └── executive_brief.pdf

2️⃣ Demo CSV Files (data/)
tasks.csv
task_id,operation_id,planned_start,planned_end,actual_start,actual_end,vendor_id
T1,OP1,2026-02-01 08:00,2026-02-01 09:00,2026-02-01 08:05,2026-02-01 09:10,V1
T2,OP1,2026-02-01 10:00,2026-02-01 11:30,2026-02-01 10:10,2026-02-01 11:50,V2
T3,OP1,2026-02-01 12:00,2026-02-01 13:30,2026-02-01 12:20,2026-02-01 13:45,V1
T4,OP2,2026-02-02 09:00,2026-02-02 10:00,2026-02-02 09:10,2026-02-02 10:15,V3
T5,OP2,2026-02-02 11:00,2026-02-02 12:30,2026-02-02 11:20,2026-02-02 12:50,V2
T6,OP2,2026-02-02 13:00,2026-02-02 14:30,2026-02-02 13:15,2026-02-02 14:40,V3
overrides.csv
override_id,task_id,reason,authority_level,timestamp
O1,T2,Resource delay,Manager,2026-02-01 10:20
O2,T3,Vendor issue,Manager,2026-02-01 12:30
O3,T5,Schedule change,Director,2026-02-02 11:25
costs.csv
task_id,planned_cost,actual_cost
T1,1000,1050
T2,1200,1300
T3,1500,1600
T4,900,950
T5,1100,1250
T6,1300,1350
vendors.csv
vendor_id,vendor_name,reliability_score
V1,Vendor A,0.95
V2,Vendor B,0.9
V3,Vendor C,0.85

3️⃣ Notebook (notebooks/demo_v1.0.ipynb)
Contains:
* Load CSVs
* Drift & stress calculation
* Visualizations (stress score, heatmaps)
* Executive narrative generator
You can copy the enhanced interactive notebook code I wrote earlier into this file.

4️⃣ Drift & Stress Module (src/drift.py)
import pandas as pd

def calculate_drift_and_stress(tasks, costs, overrides):
    tasks = tasks.merge(costs[['task_id','planned_cost','actual_cost']], on='task_id', how='left')
    tasks['time_drift_min'] = (tasks['actual_end'] - tasks['actual_start'] - (tasks['planned_end'] - tasks['planned_start'])).dt.total_seconds()/60
    tasks['cost_drift'] = tasks['actual_cost'] - tasks['planned_cost']
    override_counts = overrides.groupby('task_id').size().reset_index(name='override_count')
    tasks = tasks.merge(override_counts, on='task_id', how='left').fillna(0)
    tasks['time_norm'] = tasks['time_drift_min']/tasks['time_drift_min'].max()
    tasks['cost_norm'] = tasks['cost_drift']/tasks['cost_drift'].max()
    tasks['override_norm'] = tasks['override_count']/tasks['override_count'].max().replace({0:1})
    tasks['stress_score'] = (0.5*tasks['time_norm'] + 0.3*tasks['cost_norm'] + 0.2*tasks['override_norm'])*100
    return tasks

5️⃣ Executive Narrative (src/narrative.py)
def generate_executive_brief(tasks_df):
    narrative = ["Executive Brief:"]
    for op_id, group in tasks_df.groupby('operation_id'):
        narrative.append(f"\nOperation {op_id}:")
        for _, row in group.iterrows():
            narrative.append(
                f"  Task {row['task_id']} → Time drift: {row['time_drift_min']:.1f} min, "
                f"Cost variance: ${row['cost_drift']:.0f}, "
                f"Overrides: {int(row['override_count'])}, "
                f"Stress Score: {row['stress_score']:.0f}%"
            )
    narrative.append("\nNotes: Tasks above 50% stress score indicate rising execution risk. Monitor vendors and human overrides.")
    return "\n".join(narrative)

6️⃣ Streamlit Dashboard (src/dashboard.py)
Use the Streamlit dashboard code I wrote earlier.
* Filters for operation/vendor
* Stress score chart, time drift heatmap, overrides heatmap
* Executive brief text area
* PDF export button

7️⃣ README.md
# Universal Operations Intelligence Engine (UOIE v1.0)

## Overview
Enterprise pilot-ready system to detect operational drift, stress, and overrides across operations and vendors. Generates interactive dashboards and executive briefs.

## Setup
1. Clone repo
```bash
git clone <repo-url>
cd UOIE
2. Install dependencies
pip install -r requirements.txt
3. Run dashboard
streamlit run src/dashboard.py
Features
* Drift and stress calculation
* Visualizations: bar charts & heatmaps
* Executive brief generation (LLM-ready)
* PDF export for board review
* Pilot-ready multi-operation dataset
License
MIT / Proprietary
---
