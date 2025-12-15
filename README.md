# 📊 Dynamic Pareto Analysis in Power BI (Power Query + DAX)
An end-to-end Power BI project demonstrating how to transform messy real-world data into a fully dynamic Pareto Analysis dashboard using Power Query (M) and advanced DAX.
<br>
- This project focuses on :
  - Robust data cleaning
  - Normalization of inconsistent identifiers
  - Dynamic ranking & cumulative calculations
  - Interactive Pareto thresholds
  - Industry-standard visualization patterns
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
[Raw Input Data (Messy)](https://github.com/sumanndass/Dynamic-Pareto-Analysis-in-Power-BI-Power-Query-DAX/blob/main/MessyParetoData.csv)
- The original CSV file contained:
  - Duplicate records
  - Inconsistent casing (cust-002, CUST002)
  - Inconsistent separators (-, ., spaces)
  - Missing values (?, ??)
  - Partial records
  - Mixed numeric/text values
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
  Csv.Document(File.Contents("C:\Users\user\Desktop\MessyParetoData.csv"),[Delimiter=",", Encoding=1252, QuoteStyle=QuoteStyle.None])
  ```
  - Note: this is not the best wat to do fetch source rather make a blank query and name it "Path" then use this as a parameter and then use this parameter to fetch source.
- 2️⃣ Trim All Columns Dynamically
  ```m
  Table.TransformColumns(
    Source,
    List.Transform(ColList, each {_, each Text.Trim(_)})
  )
  ```
  - ✔ Removes leading/trailing spaces from all columns
  - ✔ Prevents hidden duplicates
