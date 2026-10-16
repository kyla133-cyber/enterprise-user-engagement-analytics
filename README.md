# enterprise-user-engagement-analytics
This case study demonstrates an end-to-end data analytics solution built to address a critical SaaS retention challenge: identifying enterprise accounts experiencing sudden drops in active user engagement before renewal periods. Using Snowflake, Advanced SQL (Window Functions & CTEs), and Tableau, this project automates cohort drop-off tracking and surfaces root-cause drivers (feature adoption vs. seat utilization) for Customer Success and Product teams.

🛠️ Tech Stack & Architecture
Data Warehouse: Snowflake (Dimensional Star Schema, Table Clustering, Incremental Loading)

Transformation & Querying: Advanced SQL (CTEs, LAG(), PERCENT_RANK(), Multi-table Joins)

Data Visualization & BI: Tableau Server (Role-based row-level permissions, Automated weekly extracts)

Version Control: Git / GitHub

🔍 1. The Business Problem
SaaS enterprise accounts often show subtle declines in user engagement weeks before formally deciding not to renew their contracts. Product Operations and Customer Success teams lacked a proactive, automated mechanism to:

Automatically flag accounts deviating from their historical usage baselines.

Differentiate between macro-level seat shrinkage versus isolated feature-level friction points.

Prioritize high-risk accounts for CSM intervention.

💻 2. Data Engineering & SQL Implementation
To handle millions of high-volume granular event logs efficiently, the query layer was optimized to run against pre-aggregated summary layers partitioned by date and organization ID.

Sample Core Logic (Drop-off Detection)
The following production-style SQL snippet utilizes window functions to compare current month-over-month active usage against a rolling 6-month historical baseline, automatically flagging anomalies:
