# Timeclock & Employee Payroll Report — Power BI Sample

This sample Power BI report demonstrates how organizations can analyze employee **timeclock activity, worked hours**, and **payroll cost** at both a summary and employee‑level granularity. It is designed to support **HR**, **payroll processing teams**, and **operational managers** who need clean, reliable visibility into **labor trends**.

## 🎯 Key Capabilities:

- Daily, weekly, and pay‑period tracking
- Regular vs Overtime calculations
- Labor cost estimation using pay rates
- Department-level rollups for operational insights
- Dynamic date filters (Pay Period, Last 7 Days, Custom Range)
- Drill into individual employee activity and detail pages

> **Note:** This project uses entirely **synthetic** data.

---

# 🧩 Data Model Overview

The report uses a **clean, star‑schema‑based** data model with three dimension tables and one central fact table.  
This structure supports efficient DAX calculations, intuitive filtering, and scalable workforce analytics.

---

## 📁 **Tables and Their Roles**

### **🟦 Employees (Dimension)**  
Represents your employee master data.

**Contains attributes such as:**
- Employee ID  
- Hire Date / Exit Date  
- Department  
- Job Title  
- Employment Status  
- Pay Type (Hourly/Salary)  
- Hourly Rate  
- FTE  
- Schedule  
- Performance Rating  
- Tenure metrics  

**Role:**  
Used for slicing, grouping, costing, headcount logic, and employment-related calculations.

---

### **🟩 dimDate (Dimension)**  
Enterprise-grade date table supporting calendar, fiscal, and pay-period logic.

**Key fields include:**
- Date  
- Fiscal Month Start / End  
- Month Start / End  
- Pay Period Start / End  

**Role:**  
Enables all time intelligence, pay-period filtering, weekly/overtime logic, and trend analysis.

---

### **🟧 TimeClock (Fact Table)**  
Primary activity table capturing timeclock punches and hour calculations.

**Fields include:**
- Clock In / Clock Out  
- Work Date / Week Start / Pay Period Start & End  
- Break Minutes / Break Hours  
- Hours Worked  
- PTO Hours  
- Unpaid Hours  
- Day of Week  
- Is Hourly / Pay Type  

**Role:**  
Feeds worked hours, break time, PTO, unpaid time, shift analytics, and payroll cost models.

---

### **🟪 Measures (Calculation Group / Measure Table)**  
Stores all core business logic, including:

#### **Employee Status Measures**
- Active Headcount  
- Employees Worked  
- Employees on PTO  
- Employees Paid  

#### **Hours Measures**
- Hours Worked  
- Paid Hours  
- Standard Weekly Hours  
- Break Minutes / Break Hours  
- Average Shift Length  
- Late Arrivals  

#### **Payroll Measures**
- Annual Compensation  
- Annualized Compensation (Active)  
- Daily Labor Cost  
- Monthly Compensation  
- Hourly Run‑Rate  
- Variance to Run-Rate  

#### **Utilization & Efficiency**
- Utilization %  
- Working % Active  

This measure layer is the analytical engine of the model.

---

## 🔗 **Relationship Structure**

Your model uses a clean star schema with the following relationships:

- **Employees[Employee ID] → TimeClock[Employee ID]**  
  *Employees is the “one” side.*

- **dimDate[Date] → TimeClock[Work Date]**  
  *dimDate is the “one” side.*

- Additional optional relationships (inactive or role-based) may support pay‑period logic.

This schema ensures optimized filtering, predictable DAX behavior, and fast aggregations.

---

## 📊 Report Pages

The report includes four primary pages designed to give a complete view of employee timeclock activity, labor utilization, and payroll performance.

---

## **1 — Schedule WTD (Week‑to‑Date)**

This page provides a weekly operational snapshot of employee work activity, capacity, and utilization.

### Key Features
- **KPI Cards**
  - Employees Worked  
  - Hours Worked  
  - Utilization %  

- **Capacity vs. Hours Worked Chart**
  - Displays Capacity Hours, Hours Worked, and Utilization % for each day of the week.

- **Employees Paid & Daily Labor Cost**
  - Shows count of employees paid per day alongside labor cost trends.

- **Workforce Flow Diagram**
  - Breaks down Employees Worked → Date → Pay Type → Individual Names → Job Title.

- **Detailed Daily Timeclock Table**
  - Work Date, Employee ID, Name, FTE, Clock‑In/Out, Hours Worked, Pay Type.

### Purpose
Operational managers can quickly understand WTD staffing levels, work patterns, labor utilization, and individual employee activity.

---

## **2 — Capacity & Utilization**

This page delivers a high‑level performance view of labor efficiency, capacity planning, and cost.

### Key Features
- **KPI Cards**
  - Capacity Hours  
  - Hours Worked  
  - Utilization %  
  - Daily Labor Cost  
  - Avg Shift Length  
  - Employees Worked  
  - Working vs Active %  

- **Capacity & Utilization Time Series**
  - Combines Capacity Hours and Utilization% into a daily dual‑axis visual.

- **Daily Labor Cost Trend**
  - Displays total daily labor cost with peaks/valleys highlighted.

- **Large Callout Visual**
  - Shows Daily Labor Cost with variance (e.g., “$211K (‑1.50%)”) and number of days in range.

- **Performance Mini‑Charts**
  - Utilization %  
  - Capacity Hours  
  - Working % Active  

### Purpose
Helps leadership understand labor efficiency and cost over the selected pay period or custom date range.

---

## **3 — Pay Period Overview**

This page summarizes the entire pay period, combining hours, shifts, and scheduled activity.

### Key Features
- **KPI Cards**
  - Employees Worked  
  - Hours Worked  
  - Shifts Worked  
  - Avg Break Minutes  
  - Capacity Hours  
  - Utilization %  

- **Schedule by Job Title**
  - A horizontally-scrolling “schedule map” showing which job titles worked on each date.

- **Employees & Hours Worked by Date**
  - A dual-line chart plotting Hours Worked vs. Employees Worked.

- **Detailed Pay-Period Table**
  - Work Date, ID, Name, FTE, Clock In/Out, Hours Worked, Pay Type.

### Purpose
Ideal for payroll teams and managers needing a complete picture of activity across the entire pay period.

---

## **4 — Employee Drillthrough Page**

This page provides a detailed HR‑level view of a single employee’s current status and history.

### Key Features
- **Employee Master Data**
  - Name  
  - Employee ID  
  - Schedule  
  - Status (Active / Resigned)  
  - Last Work Date  
  - Location / Department  
  - Job Title  
  - Hire Date / Exit Date  
  - Pay Type, Hourly Rate, Annual Salary  

- **Full Employment & Work History Table**
  - Displays all relevant employment attributes and timeclock details for the employee.

### Purpose
Allows analysts and managers to drill into any employee from the main report pages and review their complete employment and timeclock profile.

---

## 🧮 Core DAX Measures
_Path: `/scripts/dax/measures.dax`_

### **Attendance & Productivity**
```DAX
// Active Headcount (point-in-time)
Active Headcount = 
    VAR AnchorDate = MAX ( 'dimDate'[Date] )
    RETURN
    CALCULATE (
        DISTINCTCOUNT ( Employees[Employee ID] ),
        KEEPFILTERS (
            FILTER (
                ALL ( Employees[Hire Date], Employees[Exit Date] ),
                Employees[Hire Date] <= AnchorDate
                    && ( ISBLANK ( Employees[Exit Date] ) || Employees[Exit Date] >= AnchorDate )
            )
        )
    )

// Employees Worked 
Employees Worked = 
    CALCULATE (
        DISTINCTCOUNT ( TimeClock[Employee ID] ),
        KEEPFILTERS ( TimeClock[Hours Worked] > 0 )
    )

// Shifts Worked
Shifts Worked = 
    CALCULATE (
        COUNTROWS( TimeClock ),
        KEEPFILTERS ( ( TimeClock[Hours Worked] > 0 ) || ( TimeClock[PTO Hours] > 0 ) )
    )

// Hours Worked 
Hours Worked = 
    SUM ( TimeClock[Hours Worked] )

// Employees on PTO
Employees on PTO = 
    CALCULATE (
        DISTINCTCOUNT ( TimeClock[Employee ID] ),
        KEEPFILTERS ( TimeClock[PTO Hours] > 0 )
    )
```

### **Breaks & Punctuality**
```DAX
Late Arrivals (> 930am) = 
    CALCULATE (
        COUNT ( TimeClock[Employee ID] ),
        KEEPFILTERS ( TimeClock[Clock In Minutes] >= 570 ), -- 9*60 + 30
        KEEPFILTERS ( TimeClock[Hours Worked] > 0 )
    )
```
### **Capacity & Utilization** 
```DAX
// Capacity Hours
Capacity Hours = 
    SUMX (
        VALUES ( TimeClock[Employee ID] ),
        DIVIDE ( SUM(Employees[FTE])/10, 80 )
    )

// Utilization %
Utilization % = 
    DIVIDE([Hours Worked], [Capacity Hours])

// Daily Labor Cost 
Daily Labor Cost = 
    ( SUM( TimeClock[Hours Worked] ) * AVERAGE(Employees[Hourly Rate]) ) 
```

### **Compensation**
```DAX
// Annual Compensation
Annual Compensation = 
    VAR YearStart = MINX ( STARTOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR YearEnd   = MAXX ( ENDOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR DaysInYear = DATEDIFF ( YearStart, YearEnd, DAY ) + 1
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR InWindow =
            Hire <= YearEnd && ( ISBLANK ( Exit ) || Exit >= YearStart )
        VAR Annualized =
            CALCULATE (
                IF (
                    SELECTEDVALUE ( Employees[Pay Type] ) = "Salary",
                    COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ),
                    COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 )
                        * COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 ) * 52
                )
            )
        VAR EffStart = MAX ( Hire, YearStart )
        VAR EffEnd   = MIN ( COALESCE ( Exit, YearEnd ), YearEnd )
        VAR ActiveDaysInYear =
            IF ( InWindow, MAX ( 0, DATEDIFF ( EffStart, EffEnd, DAY ) + 1 ), 0 )
        RETURN DIVIDE ( Annualized * ActiveDaysInYear, DaysInYear )
    )

// Annualized Pay (row-level normalization)
Annualized Compensation (Active) = 
    VAR d = MAX( dimDate[Date] )
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR Active =
            Hire <= d && ( ISBLANK ( Exit ) || Exit >= d )
        VAR Annualized =
            CALCULATE (
                IF (
                    SELECTEDVALUE ( Employees[Pay Type] ) = "Salary",
                    COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ),
                    COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 )
                        * COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 )
                        * 52
                )
            )
        RETURN IF ( Active, Annualized, 0 )
    )

// Monthly Compensation
Monthly Compensation = -- (Pro-Rated for exits and hires)
    VAR StartDate = MINX ( STARTOFMONTH ( dimDate[Date] ), dimDate[Date] )
    VAR EndDate   = MAXX(  ENDOFMONTH( dimDate[Date] ), dimDate[Date] )
    VAR YearStart = MINX ( STARTOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR YearEnd   = MAXX ( ENDOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR DaysInYear = DATEDIFF ( YearStart, YearEnd, DAY ) + 1
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR InWindow =
            Hire <= EndDate && ( ISBLANK ( Exit ) || Exit >= StartDate )
        VAR Annualized =
            CALCULATE (
                IF (
                    SELECTEDVALUE ( Employees[Pay Type] ) = "Salary",
                    COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ),
                    COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 )
                        * COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 ) * 52
                )
            )
        VAR EffStart = MAX ( Hire, StartDate )
        VAR EffEnd   = MIN ( COALESCE ( Exit, EndDate ), EndDate )
        VAR ActiveDaysInMonth =
            IF ( InWindow, MAX ( 0, DATEDIFF ( EffStart, EffEnd, DAY ) + 1 ), 0 )
        RETURN DIVIDE ( Annualized * ActiveDaysInMonth, DaysInYear )
    )

// Salary Run-Rate
Salary Run-Rate (Active) = 
    VAR d = MAX ( 'dimDate'[Date] )
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire     = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit     = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR IsSalary = CALCULATE ( SELECTEDVALUE ( Employees[Pay Type] ) = "Salary" )
        VAR Active   = Hire <= d && ( ISBLANK ( Exit ) || Exit >= d )
        VAR Annual   = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Annual Salary] ), 0 ) )
        RETURN IF ( Active && IsSalary, Annual, 0 )
    )

// Hourly Run-Rate
Hourly Run-Rate (Active) = 
    VAR d = MAX( dimDate[Date] )
    RETURN
    SUMX (
        VALUES ( Employees[Employee ID] ),
        VAR Hire     = CALCULATE ( SELECTEDVALUE ( Employees[Hire Date] ) )
        VAR Exit     = CALCULATE ( SELECTEDVALUE ( Employees[Exit Date] ) )
        VAR IsHourly = CALCULATE ( SELECTEDVALUE ( Employees[Pay Type] ) = "Hourly" )
        VAR Active   = Hire <= d && ( ISBLANK ( Exit ) || Exit >= d )
        VAR Rate     = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Hourly Rate] ), 0 ) )
        VAR Hours    = CALCULATE ( COALESCE ( SELECTEDVALUE ( Employees[Typical Hours] ), 0 ) )
        RETURN IF ( Active && IsHourly, Rate * Hours * 52, 0 )
    )

// Variance to Run-Rate 
Variance to Run-Rate (YTD) = 
    VAR StartDate = MINX ( STARTOFYEAR ( dimDate[Date] ), dimDate[Date] )
    VAR EndDate   = MAX ( dimDate[Date] )
    VAR MonthsElapsed = DATEDIFF ( StartDate, EndDate, MONTH ) + 1   // Jan=1, Feb=2, ...
    RETURN
    [Annual Compensation]
        - DIVIDE ( [Annualized Compensation (Active)], 12 ) * MonthsElapsed
```
---

## 📁 Repository Layout

```
/
|-- index.md
|-- README.md
|-- pbix/
|    └── Timeclock-Payroll-Report.pbix
|-- data/
|    └── fake_timeclock_payroll_dataset.xlsx
|-- scripts/
|    └── dax/
|         └── measures.dax
|-- documentation/
     ├── data-model.png
     ├── employee-drillthrough-page.png
     ├── Page1-PAY-PERIOD-OVERVIEW.png
     ├── Page2-CAPACITY&UTILIZATION.png
     └── Page3-SCHEDULE-WTD.png
```

## 📄 License
MIT
