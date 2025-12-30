# Medicaid Drug Data Viz Exercise - Power BI Desktop Instructions

## Overview
This guide provides step-by-step instructions to create the Medicaid State Drug Utilization Data (SDUD) visualization project in Power BI Desktop.

---

## Prerequisites

1. **Power BI Desktop** installed (latest version recommended)
2. **Data Files**:
   - SDUD 2025 CSV data (accessible via web URL: https://download.medicaid.gov/data/sdud-2025-updated-dec2025.csv)
   - MDRP Products CSV file (download from https://data.medicaid.gov/dataset/0ad65fe5-3ad3-5d79-a3f9-7893ded7963a and save as a local file, for example `C:\Data\MDRP Products.csv`)
3. **Internet connection** for downloading SDUD data

---

## Part 1: Create New Power BI Desktop File

1. Open **Power BI Desktop**
2. Create a new blank report
3. Save the file as `Medicaid Drug Data Viz Exercise.pbix`

---

## Part 2: Set Up Parameters

### 2.1 Create SDUD_2025_CSV_URL Parameter

1. Go to **Home** tab → **Transform data** (Power Query Editor opens)
2. In Power Query Editor, go to **Home** tab → **Manage Parameters** → **New Parameter**
3. Configure the parameter:
   - **Name**: `SDUD_2025_CSV_URL`
   - **Description**: `State Drug Utilization Data link to the SDUD 2025 CSV export`
   - **Required**: ✓ (checked)
   - **Type**: Text
   - **Current Value**: `https://download.medicaid.gov/data/sdud-2025-updated-dec2025.csv`
4. Click **OK**

### 2.2 Create MDRP_PRODUCTS_CSV_URL Parameter

1. In Power Query Editor, go to **Home** tab → **Manage Parameters** → **New Parameter**
2. Configure the parameter:
   - **Name**: `MDRP_PRODUCTS_CSV_URL`
   - **Description**: `MDRP Drug Products – quarterly snapshots`
   - **Required**: ✓ (checked)
   - **Type**: Text
   - **Current Value**: `C:\Data\MDRP Products.csv`
   - *(Note: Update this path to match your local file location)*
3. Click **OK**

---

## Part 3: Create Data Queries

### 3.1 Create MDRP_Products Query

1. In Power Query Editor, go to **Home** tab → **New Source** → **Blank Query**
2. Rename the query to `MDRP_Products`
3. Go to **View** tab → **Advanced Editor** (or create the steps manually using the Power Query editor)
4. Replace the content with:

```m
let
    MDRP_PRODUCTS_CSV_URL = #"MDRP_PRODUCTS_CSV_URL",
    Source = Csv.Document(File.Contents(MDRP_PRODUCTS_CSV_URL), [Delimiter=",", Encoding=1252, QuoteStyle=QuoteStyle.Csv]),
    PromoteHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(PromoteHeaders,{{"FDA Approval Date", type date}, {"Market Date", type date}})
in
    #"Changed Type"
```

5. Click **Done**
6. The query will load the MDRP Products data

### 3.2 Create SDUD_2025 Fact Table

1. In Power Query Editor, go to **Home** tab → **New Source** → **Blank Query**
2. Rename the query to `SDUD_2025`
3. Go to **View** tab → **Advanced Editor**
4. Replace the content with:

```m
let
    SDUD_2025_CSV_URL = #"SDUD_2025_CSV_URL",
    Source = Csv.Document(Web.Contents(SDUD_2025_CSV_URL), [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
    PromoteHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    Renamed = Table.TransformColumnTypes(PromoteHeaders, {
        {"Utilization Type", type text},
        {"State", type text},
        {"NDC", type text},
        {"Package Size", Int64.Type},
        {"Year", Int64.Type},
        {"Quarter", Int64.Type},
        {"Product Name", type text},
        {"Units Reimbursed", type number},
        {"Number of Prescriptions", type number},
        {"Total Amount Reimbursed", type number},
        {"Medicaid Amount Reimbursed", type number},
        {"Non Medicaid Amount Reimbursed", type number}
    })
in
    Renamed
```

5. Click **Done**
6. Wait for the data to load (this may take a minute since it's downloading from the web)

### 3.3 Create DimLableler Dimension Table

1. In Power Query Editor, go to **Home** tab → **New Source** → **Blank Query**
2. Rename the query to `DimLableler`
3. Go to **View** tab → **Advanced Editor**
4. Replace the content with:

```m
let
    Source = MDRP_Products,
    #"Removed Other Columns" = Table.SelectColumns(Source,{"Labeler Name", "Labeler Code"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Other Columns", {"Labeler Name"}),
    #"Reordered Columns" = Table.ReorderColumns(#"Removed Duplicates",{"Labeler Code", "Labeler Name"})
in
    #"Reordered Columns"
```

5. Click **Done**

### 3.4 Create DimProduct Dimension Table

1. In Power Query Editor, go to **Home** tab → **New Source** → **Blank Query**
2. Rename the query to `DimProduct`
3. Go to **View** tab → **Advanced Editor**
4. Replace the content with:

```m
let
    Source = MDRP_Products,
    #"Removed Other Columns" = Table.SelectColumns(Source,{"NDC", "FDA Product Name"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Other Columns", {"NDC"})
in
    #"Removed Duplicates"
```

5. Click **Done**

### 3.5 Finalize Query Loading

1. **Right-click** on `MDRP_Products` query → **Enable load** (uncheck it - this is a staging query)
2. Verify that `SDUD_2025`, `DimLableler`, and `DimProduct` are set to load
3. Click **Close & Apply** to load all data into the model

---

## Part 4: Configure Data Model

### 4.1 Hide Columns

1. In **Model view** (left sidebar icon), select the `SDUD_2025` table
2. In the Properties pane, hide the following columns:
   - `Labeler Code`
   - `Product Code`

3. Select the `DimLableler` table and hide:
   - `Labeler Code`

### 4.2 Create Relationships

Power BI should auto-detect these relationships, but verify or create them manually:

1. **Relationship 1**: `SDUD_2025[Labeler Code]` (Many) → `DimLableler[Labeler Code]` (One)
   - Cardinality: Many-to-One
   - Cross filter direction: Single
   - Make this relationship active: ✓

2. **Relationship 2**: `SDUD_2025[NDC]` (Many) → `DimProduct[NDC]` (One)
   - Cardinality: Many-to-One
   - Cross filter direction: Single
   - Make this relationship active: ✓

To create a relationship manually:
- In Model view, drag `Labeler Code` from `SDUD_2025` table to `Labeler Code` in `DimLableler` table
- Drag `NDC` from `SDUD_2025` table to `NDC` in `DimProduct` table

### 4.3 Create Calculated Column

1. Select the `SDUD_2025` table
2. In the **Table tools** tab, click **New column**
3. Enter the following DAX formula:

```dax
Labeler Name = RELATED(DimLableler[Labeler Name])
```

4. Press **Enter**

### 4.4 Create Hierarchy

1. In the `SDUD_2025` table, right-click on `Labeler Name` column
2. Select **Create hierarchy**
3. Rename the hierarchy to `Labeler Name Hierarchy`
4. Drag `Product Name` column into the hierarchy under `Labeler Name`

The hierarchy should have two levels:
- Level 1: Labeler Name
- Level 2: Product Name

---

## Part 5: Create Measures

In the `SDUD_2025` table, create the following measures:

### 5.1 Basic Measures

1. Click on `SDUD_2025` table
2. Go to **Table tools** tab → **New measure**
3. Create each measure:

**Units**
```dax
Units = SUM('SDUD_2025'[Units Reimbursed])
```

**Prescriptions**
```dax
Prescriptions = SUM('SDUD_2025'[Number of Prescriptions])
```

**TotalReimbursed**
```dax
TotalReimbursed = SUM('SDUD_2025'[Total Amount Reimbursed])
```

**MedicaidReimbursed**
```dax
MedicaidReimbursed = SUM('SDUD_2025'[Medicaid Amount Reimbursed])
```

**NonMedicaidReimbursed**
```dax
NonMedicaidReimbursed = SUM('SDUD_2025'[Non Medicaid Amount Reimbursed])
```


### 5.3 Dynamic Title Measure

**Product Detail Title**
```dax
Product Detail Title = "Product Detail for " & SELECTEDVALUE(SDUD_2025[Labeler Name])
```

---

## Part 6: Create Custom Theme

### 6.1 Create Theme JSON File

1. Open a text editor (Notepad, VS Code, etc.)
2. Create a new file and paste the following JSON content:

```json
{
  "name": "Medicaid Drug Demo Theme",
  "foreground": "#2F2F2F",
  "background": "#FFFFFF",
  "tableAccent": "#6F2DBD",
  "visualStyles": {},
  "dataColors": [
    "#6F2DBD",
    "#048BA8",
    "#F18F01",
    "#2E4057",
    "#8E5572",
    "#00A7E1",
    "#9CC69B",
    "#D95D39",
    "#4F6D7A",
    "#B5A886",
    "#4AC5BB",
    "#5F6B6D",
    "#FB8281",
    "#F4D25A",
    "#7F898A",
    "#A4DDEE",
    "#FDAB89",
    "#B687AC"
  ]
}
```

3. Save the file as `Company_Theme.json` in an easily accessible location (e.g., your Documents folder)

### 6.2 Understanding the Theme Settings

The theme includes these key elements:
- **name**: "Demo Theme" - the display name in Power BI
- **foreground**: "#2F2F2F" - dark gray text color
- **background**: "#FFFFFF" - white background
- **tableAccent**: "#6F2DBD" - purple color for table headers
- **dataColors**: Array of 18 custom colors for data visualization

**Primary Colors in the Palette:**
- **Purple** (#6F2DBD) - Primary brand color
- **Teal** (#048BA8) - Secondary color
- **Orange** (#F18F01) - Accent color
- **Dark Blue** (#2E4057) - Supporting color

### 6.3 Import the Theme into Power BI

1. In Power BI Desktop, go to **View** tab → **Themes** → **Browse for themes**
2. Navigate to where you saved `Company_Theme.json`
3. Select the theme file and click **Open**
4. The theme will be applied immediately to your report

### 6.4 Verify Theme Application

After importing:
- Check that chart colors use the purple, teal, and orange palette
- Verify text appears in dark gray (#2F2F2F)
- Confirm table accent colors are purple

---

## Part 7: Create Report Pages

### 7.1 Create Page 1: "Reimbursed by Labeler"

1. Rename **Page 1** to `Reimbursed by Labeler`
2. **Add a Clustered Column Chart**:
   - **X-axis**: `SDUD_2025[Labeler Name Hierarchy]` (Labeler Name level)
   - **Y-axis**: `TotalReimbursed` measure
   - **Sort by**: TotalReimbursed (Descending)
   - **Enable drill-down** for the hierarchy by clicking the arrow down (click to turn on Drill down)
   - **Position**: Center-large, taking most of the page
   
3. **Add a Visual Filter** to the column chart:
   - Field: `DimLableler[Labeler Name]`
   - Filter type: Advanced filtering
   - Condition: "is not blank" (excludes null values)

### 7.2 Create Page 2: "Product Detail"

1. Add a new page and rename it to `Product Detail`

2. **Add a Page Filter**:
   - In the Filters pane (page level), add `SDUD_2025[Labeler Name]`
   - Set initial filter to a specific labeler (user can change this)

3. **Add a Card Visual** (Dynamic Title):
   - Add the measure `Product Detail Title`
   - **Position**: Stretch across top of page
   - Format as card (icon with 123 and no lightning bolt). Click ... in visualization list and "Restore default visuals" if it does not appear

4. **Add a Table Visual**:
   - **Columns** (in order):
     - `SDUD_2025[Product Name]`
     - `MedicaidReimbursed`
     - `NonMedicaidReimbursed`
     - `Prescriptions`
     - `TotalReimbursed`
     - `Units`
   - **Position**: Right side of page (large table)
   - **Add a visual filter**: Exclude blank `Product Name` values

6. **Add a Treemap**:
   - **Category**: `SDUD_2025[Product Name]`
   - **Values**: `TotalReimbursed`
   - **Position**: Left side of page
---

## Part 8: Format and Polish

### 8.1 Page Settings

For both pages:
1. Go to **View** tab → **Page view** → **Fit to page**
2. Standard size: 16:9 (1280x720)

### 8.2 Visual Formatting

Apply consistent formatting across visuals:
- **Title**: Show titles on all visuals
- **Background**: Light gray or white
- **Border**: Subtle borders for separation
- **Font**: Use theme defaults (should match theme)
- **Number Format**: 
  - Currency values: Display in millions with $ prefix (e.g., $2.5M)
  - Whole numbers: Add thousand separators

---

## Part 9: Test and Validate

### 9.1 Test Interactions

1. On "Reimbursed by Labeler" page:
   - Click on a bar to drill down to products
   - Verify that drilling filters work correctly
   - Test the hierarchy expansion

2. Test "Product Detail" page:
   - Right-click on a bar for a labeler and choose Drill through > Product Detail to open the Product Detail page
   - Verify that all visuals update accordingly
   - Check that the dynamic title changes

### 9.2 Verify Data

1. Compare totals across pages for consistency
2. Spot-check a few data points against source data
3. Ensure no blank or null values appear in key visuals

---

## Part 10: Save and Publish

### 10.1 Save the Report

1. **File** → **Save As**
2. Save as: `Medicaid Drug Data Viz Exercise.pbix`

### 10.2 Publish to Power BI Service (Optional)

1. **Home** tab → **Publish**
2. Select your workspace
3. Click **Select**

---

## Tips and Best Practices

### Performance Optimization
- Use Import mode for better performance with this dataset
- Create aggregations if dataset grows significantly
- Minimize use of calculated columns; prefer measures

### Maintenance
- Update the `SDUD_2025_CSV_URL` parameter if the data source URL changes
- Refresh data regularly (monthly for SDUD data)
- Document any custom changes or additional measures

### Common Issues

**Issue**: "Couldn't find file" error for MDRP Products
- **Solution**: Update the `MDRP_PRODUCTS_CSV_URL` parameter with correct local path

**Issue**: Relationships not working
- **Solution**: Verify that Labeler Code and NDC columns are clean (no extra spaces)

**Issue**: Web data not loading
- **Solution**: Check internet connection; verify URL is correct; check firewall settings

---

## Additional Resources

- **Data Source**: [Medicaid State Drug Utilization Data](https://www.medicaid.gov/medicaid/prescription-drugs/state-drug-utilization-data/index.html)
- **Power BI Documentation**: [Microsoft Power BI Docs](https://docs.microsoft.com/power-bi/)
- **DAX Reference**: [DAX Function Reference](https://dax.guide/)

---

## Appendix: Complete Measure List

Quick reference for all measures created:

| Measure Name | Formula |
|-------------|---------|
| Units | `SUM('SDUD_2025'[Units Reimbursed])` |
| Prescriptions | `SUM('SDUD_2025'[Number of Prescriptions])` |
| TotalReimbursed | `SUM('SDUD_2025'[Total Amount Reimbursed])` |
| MedicaidReimbursed | `SUM('SDUD_2025'[Medicaid Amount Reimbursed])` |
| NonMedicaidReimbursed | `SUM('SDUD_2025'[Non Medicaid Amount Reimbursed])` |
| Product Detail Title | `"Product Detail for " & SELECTEDVALUE(SDUD_2025[Labeler Name])` |

---

**Document Version**: 1.0  
**Last Updated**: December 30, 2025  
**Created For**: Medicaid Drug Data Visualization Exercise

