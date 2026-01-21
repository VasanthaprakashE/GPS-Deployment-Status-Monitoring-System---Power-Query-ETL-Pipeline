# GPS-Deployment-Status-Monitoring-System---Power-Query-ETL-Pipeline
ETL pipeline for monitoring GPS device deployment status. Extracts raw GPS data from PostgreSQL, transforms it, enriches with vehicle info, and prepares it for reporting.


GPS Deployment Status Monitoring System - Power Query ETL Pipeline
📋 Overview
This Power Query (M language) script implements a comprehensive ETL (Extract, Transform, Load) pipeline for monitoring GPS device deployment status across a vehicle fleet. The system extracts raw GPS data from a PostgreSQL database, transforms it according to business rules, enriches it with master vehicle information, and prepares it for reporting and analysis.

🎯 Business Purpose
Primary Objective: Track and monitor GPS device health and deployment status across company vehicles to ensure operational efficiency and timely maintenance.

Key Business Questions Answered:

Which GPS devices are currently not working?

What are the reasons for device failures?

Which technicians are responsible for which devices?

What is the deployment status across different facilities?

Which devices require immediate attention based on inactivity?

🗂️ Data Sources
1. Primary Data Source (show_gps_deployment_status())
Source: PostgreSQL database via ODBC connection

Connection: DSN PostgreSQL35W

Function: Stored procedure returning GPS deployment data

2. Reference Data (Master table)
Purpose: Vehicle master data containing reference information

Key Fields: Register Number, Facility, Technician, Zone

Join Key: Vehicle Registration No ↔ Register Number

🔄 ETL Pipeline Architecture
Stage 1: Extraction & Initial Transformation
text
Raw PostgreSQL Data → SQL Transformation → Base Power Query Table
Stage 2: Business Logic Application
text
Base Table → Status Classification → Remarks Standardization → Data Enrichment
Stage 3: Presentation Layer
text
Enriched Data → Column Reorganization → Sorting → Final Report
📊 Data Transformation Logic
1. SQL-Level Transformations (Database Layer)
sql
CASE 
    -- Standardize working statuses to '-'
    WHEN "Remarks" IN ('Working', 'B shift', '-') THEN '-'
    
    -- BOV exception handling for line issues
    WHEN "Vehicle Type" != 'BOV' AND "Remarks" = 'Line issue' THEN '-'
    
    -- Group similar fault types
    WHEN "Remarks" IN ('Line issue', 'Converter Fault') THEN 'Line issue'
    
    -- Preserve GPS-specific issues
    WHEN "Remarks" IN ('Gps Issue', 'GPS Missing') THEN "Remarks"
    
    -- Deployed devices with non-standard remarks
    WHEN "Remarks" NOT IN ('Line issue', 'Gps Issue', 'Converter Fault', 
                          'GPS Missing', 'Working') 
         AND "Deployed Status" LIKE 'Deployed%' THEN 'N'
    
    -- Non-working devices with non-standard remarks
    WHEN "Remarks" NOT IN ('Line issue', 'Gps Issue', 'Converter Fault', 
                          'GPS Missing', 'Working') THEN 'BreakDown'
    
    -- Default fallback
    ELSE COALESCE("Remarks", '-')
END AS "Updated Remarks"
2. Power Query Transformations (Application Layer)
Status Classification
powerquery
if [Age] >= 2 then "Not Working" else "Working"
Logic: Devices inactive for 2+ days are flagged as "Not Working"

Purpose: Proactive monitoring of device health

Data Enrichment
Left Join between operational data and master vehicle data

Purpose: Augment operational data with reference information (Facility, Technician, Zone)

Column Naming Convention
Master. prefix: Primary/master data from vehicle registry

Ref. prefix: Original reference data from GPS system

No prefix: Calculated/transformed fields

📈 Column Structure & Organization
Group 1: Core Identification
Column	Source	Purpose
Date	GPS System	Report/Extract date
GPS IMEI No	GPS System	Unique device identifier
Vehicle Registration No	GPS System	Vehicle identifier
V ID	GPS System	Vehicle ID
Vehicle Type	GPS System	Vehicle classification
Group 2: Location & Assignment
Column	Source	Purpose
Master.Faclity	Master Data	Primary assigned facility
Master.Zone	Master Data	Geographic zone
Master.Technician	Master Data	Responsible technician
Group 3: Operational Status
Column	Source	Purpose
Last Log Received At	GPS System	Last communication timestamp
Status	Calculated	Working/Not Working (based on Age)
Age	GPS System	Days since last log
Updated Remarks	Calculated	Standardized remark category
Group 4: Reference Data (Original Values)
Column	Original Name	Purpose
Ref.Remarks	Remarks	Original remarks
Ref.Deployed Status	Deployed Status	Original deployment status
Ref.Facility	Facility	Original facility assignment
Ref.Technician	Technician	Original technician assignment
Ref.Remarks Date	Remarks Date	Original remark date
🔢 Business Rules Matrix
Remark Categorization Logic
Original Remark	Vehicle Type	Deployment Status	Result Category
'Working'	Any	Any	'-'
'B shift'	Any	Any	'-'
'-'	Any	Any	'-'
'Line issue'	Non-BOV	Any	'-'
'Line issue'	BOV	Any	'Line issue'
'Converter Fault'	Any	Any	'Line issue'
'Gps Issue'	Any	Any	'Gps Issue'
'GPS Missing'	Any	Any	'GPS Missing'
Other	Any	Deployed%	'N'
Other	Any	Not Deployed	'BreakDown'
Status Determination
Age (Days)	Status	Action Required
< 2	Working	Monitoring only
≥ 2	Not Working	Investigate/Repair
⚙️ Technical Implementation Details
Connection Configuration
powerquery
Odbc.Query("dsn=PostgreSQL35W", [SQL Query])
Driver: PostgreSQL ODBC Driver

Authentication: Configured via DSN

Query Timeout: Default system settings

Performance Considerations
SQL-Level Processing: CASE logic processed at database level for efficiency

Join Optimization: Left outer join prevents loss of operational data

Column Selection: Only necessary columns expanded from master table

Error Handling
COALESCE used in SQL to handle NULL remarks

Left Join ensures operational data preservation

Default values: '-' for standardized working status

📋 Output Specifications
Sorting Hierarchy
Primary: Master.Technician (A-Z)

Secondary: Master.Faclity (A-Z)

Tertiary: Updated Remarks (Z-A, placing critical issues first)

Data Quality Checks
All original records preserved through left joins

No data truncation in transformations

Clear distinction between calculated and original values

Consistent null handling

🚀 Deployment & Usage
Prerequisites
PostgreSQL ODBC driver installed

DSN PostgreSQL35W configured

Master query/table available in Power Query environment

Necessary database permissions

Refresh Schedule
Recommended: Daily refresh for operational monitoring

Frequency: Can be adjusted based on business needs

Timing: Off-peak hours recommended

Output Consumption
Direct Analysis: In Excel via pivot tables/charts

Dashboard Integration: Connect to Power BI/PowerPivot

Reporting: Export to CSV/PDF for distribution

Alerting: Filter for "Not Working" status for action items

🔍 Troubleshooting Guide
Common Issues & Solutions
Issue	Possible Cause	Solution
Connection failed	DSN not configured	Verify ODBC Data Source configuration
Missing master data	Join field mismatch	Verify Vehicle Registration No format
Incorrect status	Age calculation	Check Last Log Received At data quality
SQL errors	Query syntax	Validate PostgreSQL compatibility
Data Validation Steps
Verify record count matches source system

Check for nulls in key fields (IMEI, Vehicle Registration)

Validate status distribution (Working vs Not Working)

Confirm master data join success rate

📈 Monitoring & Metrics
Key Performance Indicators
Device Uptime: % of devices with status "Working"

Response Time: Days to resolve "Not Working" status

Data Completeness: % of records with complete master data

Refresh Reliability: Success rate of pipeline execution

Quality Metrics
Data freshness (max Age in dataset)

Remark categorization accuracy

Master data coverage percentage

🔮 Future Enhancements
Planned Improvements
Automated Alerting: Email notifications for critical issues

Historical Trending: Track device health over time

Predictive Analytics: Forecast device failures

Mobile Access: Dashboard accessible via mobile devices

Scalability Considerations
Partitioning strategy for large datasets

Incremental refresh implementation

Query performance optimization
