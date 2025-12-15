# 📊 Dynamic Pareto Analysis in Power BI (Power Query + DAX)
An end-to-end Power BI project demonstrating how to transform messy real-world data into a fully dynamic Pareto Analysis dashboard using Power Query (M) and advanced DAX.
<br>
- This project focuses on :
  - Robust data cleaning
  - Normalization of inconsistent identifiers
  - Dynamic ranking & cumulative calculations
  - Interactive Pareto thresholds
  - Industry-standard visualization patterns
## 🔍 Problem Statement
Pareto (80/20) analysis is widely used to identify the vital few contributors driving the majority of impact — revenue, complaints, delays, etc.
- However, in real-world projects:
  - Data is messy
  - Categories change (Customer / Product / Supplier)
  - Metrics change (Revenue / Complaints / Delay)
  - Pareto thresholds are not fixed (not always 80%)
- Most examples online show static, hard-coded Pareto charts that:
  - Break when slicers change
  - Fail with duplicates
  - Produce incorrect cumulative percentages
  - Cannot dynamically highlight the “vital few”
- This project demonstrates how to build a **fully dynamic**, **production-ready Pareto Analysis** in **Power BI**, starting from **messy CSV data**.
## 📑 Table of Contents
- [Project Overview](#project-overview)
- [Dataset Description](#dataset-description)
- [Data Cleaning Using Power Query](#data-cleaning-using-power-query)
- [Power Query (M) Logic Explained](#power-query-(m)-logic-explained)
- [Data Modeling](#data-modeling)
- [Dynamic Pareto Logic (DAX)](#dynamic-pareto-logic-(dax))
- [Interactive Dashboard Features](#interactive-dashboard-features)
- [Key Business Insights](#key-business-insights)
- [Final Outcome](#final-outcome)
### 📘 Project Overview
Pareto analysis (80/20 rule) is a common analytical technique, but most implementations are static.
- This project demonstrates how to build a fully dynamic Pareto dashboard in Power BI that allows users to:
  - Switch analysis between Customer / Product / Supplier
  - Switch metrics between Revenue / Complaints / Delivery Delay
  - Dynamically adjust the Pareto threshold (48%–53%)
  - Automatically highlight the critical contributors
  - All calculations are measure-driven, not column-based.
### 📂 Dataset Description
- [Raw Input Data (Messy)](https://github.com/sumanndass/Dynamic-Pareto-Analysis-in-Power-BI-Power-Query-DAX/blob/main/MessyParetoData.csv)
- The original CSV file contained:
  - Duplicate records
  - Inconsistent casing (cust-002, CUST002, Cust-02)
  - Inconsistent separators (-, ., spaces)
  - Missing values (?, ??)
  - Partial records
  - Mixed numeric/text values
  - Zero values mixed with missing values
  - Examples
    ```m
    cust-002   prod B   sup-2   8500   12   8
    CUST002   Product-B   SUP2   8500   12   8
    cust-007  Prod?E   SUP-5   6000   ??   5
    ```
- Columns:
  - Customer
  - Product
  - Supplier
  - Revenue
  - Complaints
  - Delivery Delay (days)
### 🧹 Data Cleaning Using Power Query
Power Query is used as the single source of truth for all data standardization.
- Key Cleaning Objectives
  - Normalize Customer, Product, Supplier IDs
  - Remove duplicate business records
  - Convert invalid values (?) to clean numeric values
  - Ensure consistent formatting for reporting
  - Prepare fact-level data for Pareto logic
### 🧠 Power Query (M) Logic Explained
- 1️⃣ Source
  ```m 
  = Csv.Document(File.Contents("C:\Users\user\Desktop\MessyParetoData.csv"),[Delimiter=",", Encoding=1252, QuoteStyle=QuoteStyle.None])
  ```
  - **Note:** this is not the best wat to do fetch source rather make a blank query and name it "Path" then use this as a parameter and then use this parameter to fetch source.
- 2️⃣ Duplicate the Source
  ```m
  let
      Source = Csv.Document(File.Contents("C:\Users\user\Desktop\MessyParetoData.csv"),[Delimiter=",", Columns=6, Encoding=1252, QuoteStyle=QuoteStyle.None]),
      Custom1 = Table.ColumnNames(Source)
  in
      Custom1
  ```
  - ✔ to find the column names and and make it dynamic for rest of the codes
- 3️⃣ Trim All Columns Dynamically
  ```m
  = Table.TransformColumns(
    Source,
    List.Transform(ColList, each {_, each Text.Trim(_)})
  )
  ```
  - ✔ Removes leading/trailing spaces from all columns
  - ✔ Prevents hidden duplicates
- 4️⃣ Promote Headers
  ```m
  = Table.PromoteHeaders(#"Trim Each Col", [PromoteAllScalars=true])
  ```
- 5️⃣ Replace Invalid Characters (?, ??)
  ```m
  = Table.TransformColumns(
    #"Promoted Headers",
    List.Transform(
        Table.ColumnNames(#"Promoted Headers"),
        each { _, (v) => if v is null then null else if v is text then Text.Replace(v, "?", "") else v }
    )
  )
  ```
- 6️⃣ Customer Column Standardization  
  ```m
  = Table.TransformColumns(
    #"Replace ? and ??",
    {{"Customer", each 
        let
            clean = Text.Upper(Text.Replace(_, "-", " ")),
            spaced = Text.Combine(
                        List.Transform(
                            Text.SplitAny(clean, " "),
                            each 
                                let
                                    letters = Text.Select(_, {"A".."Z"}),
                                    digits = Text.Select(_, {"0".."9"})
                                in
                                    Text.Combine({letters, digits}, " ")
                        )
                    , " "),
            trimmed = Text.Trim(Text.Combine(List.Select(Text.Split(spaced, " "), each _ <> ""), " ")),
            proper = Text.Proper(trimmed)
        in
            proper
    }}
  )
  ```
  ```m
  Cust-001 → Cust 001
  CUST002 → Cust 002
  ```
  - Logic applied:
    - Uppercase
    - Remove separators
    - Extract letters + digits
    - Rebuild in consistent format
  - ✔ Ensures accurate grouping & ranking
- 7️⃣ Product & Supplier Normalization
  ```m
  = Table.TransformColumns(
    #"Customer Col Fix",
    {
        {
            "Product",
            each
                let
                    first = Text.Start(_, 4),
                    last = Text.End(_, 1),
                    combine = Text.Combine({first, last}, " "),
                    proper = Text.Proper(combine)
                in
                    proper
        },
        {
            "Supplier",
            each
                let
                    first = Text.Upper(Text.Start(_, 3)),
                    last = Text.End(_, 1),
                    combine = Text.Combine({first, last}, " ")                
                in
                    combine
        }
    }
  )
  ```
  ```m
  Prod-A → Prod A
  SUP.10 → SUP 10
  ```
  - ✔ Prevents slicer duplication
  - ✔ Improves visual clarity
- 8️⃣ Remove Duplicates & Invalid Rows
  ```m
  = Table.Distinct(#"Product, Supplier Col Fix")
  ```
  - ✔ Eliminates duplicate business records
  - ✔ Filters blank identifiers
- 9️⃣ Filtering Rows
  ```m
  = Table.SelectRows(#"Removed Duplicates", each ([Customer] <> "") and ([Product] <> " ") and ([Supplier] <> " "))
  ```
- 1️⃣0️⃣ Replace Missing Numerics with Zero
  ```m
  = Table.ReplaceValue(#"Filtered Rows","","0",Replacer.ReplaceValue,{"Revenue", "Complaints", "Delivery Delay (days)"})
  ```
  - ✔ Enables numeric aggregation
  - ✔ Prevents DAX calculation errors
- 1️⃣1️⃣ Final Data Types
  ```m
  = Table.TransformColumnTypes(#"Replaced Value",{{"Customer", type text}, {"Product", type text}, {"Supplier", type text}, {"Revenue", Int64.Type}, {"Complaints", Int64.Type}, {"Delivery Delay (days)", Int64.Type}})
  ```
  ```m
  = Table.TransformColumnTypes(#"Replaced Value",{{"Customer", type text}, {"Product", type text}, {"Supplier", type text}, {"Revenue", Int64.Type}, {"Complaints", Int64.Type}, {"Delivery Delay (days)", Int64.Type}}, [MissingField = MissingField.Ignore]) // newer version May-2025
  ```
  - ✔ Model-ready dataset
- 1️⃣2️⃣ Final Cleaned Dataset
  - [Final Dataset](https://github.com/sumanndass/Dynamic-Pareto-Analysis-in-Power-BI-Power-Query-DAX/blob/main/CleanedParetoData.xlsx)
  - Example
    ```m
    | Customer | Product | Supplier | Revenue | Complaints | Delivery Delay |
    | -------- | ------- | -------- | ------: | ---------: | -------------: |
    | Cust 012 | Prod H  | SUP 1    |   21000 |         17 |             29 |
    | Cust 011 | Prod H  | SUP 1    |   20000 |         18 |             30 |
    | Cust 001 | Prod A  | SUP 1    |   12000 |          5 |             15 |
    | Cust 018 | Prod L  | SUP 1    |   12000 |         25 |             20 |
    | Cust 019 | Prod L  | SUP 1    |   12000 |         25 |             20 |
    | ...      | ...     | ...      |     ... |        ... |            ... |
    ```
### 🧩 Data Modeling
- This project uses a single fact table approach:
  - Suitable for Pareto analysis
  - Avoids unnecessary complexity
  - Optimized for dynamic ranking
- All calculations are performed using measures, not calculated columns.
### 📐 Dynamic Pareto Logic (DAX)
- Core Concepts Used
  - RANKX
  - ALLSELECTED
  - Dynamic SWITCH logic
  - Cumulative percentage calculation
  - Closest-to-target threshold logic
- Key Measures
  - Dynamic Rank
  - Cumulative Value
  - Cumulative Percentage
  - Pareto Highlight Flag
  - Dynamic Titles
- ✔ Ranking updates per slicer
- ✔ No hard-coded logic
- ✔ No static thresholds
### 🎛 Interactive Dashboard Features (User Controls)
- Category Selector
  - Customer
  - Product
  - Supplier
- Metric Selector
  - Total Revenue
  - Total Complaints
  - Total Delivery Delay
- Pareto Threshold Selector
  - 48% – 53%
### 📊 Visual Components
- Dynamic Pareto Bar + Line Chart
- Highlighted critical contributors
- Dynamic matrix showing:
  - Rank
  - Cumulative Value
  - Cumulative %
- Fully dynamic titles:
  - Top 5 out of 18 Customers account for 58% of Total Revenue (₹77,000)
### 📈 Key Business Insights
- From the dashboard:
  - 58% of total revenue comes from just 5 customers
  - Revenue concentration risk is clearly visible
  - Priority customers are immediately identifiable
  - Switching metrics reveals different Pareto drivers
- This mirrors real enterprise analytics use cases.
### ✅ Final Outcome
- ✔ Messy real-world data cleaned using Power Query
- ✔ Fully dynamic Pareto analysis using DAX
- ✔ Interactive slicers and thresholds
- ✔ Production-quality Power BI dashboard
- ✔ Reusable design for any Pareto scenario
### 🚀 Conclusion
- This project demonstrates how Power BI + Power Query + DAX can be used to:
  - Handle dirty datasets
  - Build robust analytical logic
  - Deliver executive-ready insights
- It goes beyond textbook Pareto charts and reflects real BI engineering practices.
### 🔗 Possible Extensions
- Add drill-through pages
- Convert to star schema
- Publish as Power BI template
- Add performance optimization benchmarks
