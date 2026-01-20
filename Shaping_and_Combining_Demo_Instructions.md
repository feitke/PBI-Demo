# Shaping and Combining Demo - Step-by-Step Instructions

## Overview
This Power BI exercise demonstrates essential data shaping and combining techniques using the Northwind OData service and an Excel file. You'll learn how to:
- Connect to OData and Excel data sources
- Expand nested tables
- Filter and remove columns
- Merge queries to combine data
- Create calculated columns
- Build hierarchies
- Establish relationships

---

## Prerequisites
- Power BI Desktop installed
- Internet connection (for OData feed)
- Excel file: `Products.xlsx` (should contain Products sheet with product data). Download from https://download.microsoft.com/download/1/4/E/14EDED28-6C58-4055-A65C-23B4DA81C4DE/Products.xlsx and save the file.

---

## Part 1: Set Up the Categories Query (Parameter Query)

### Step 1: Connect to Northwind OData Feed
1. Open Power BI Desktop
2. Click **Get Data** > **OData Feed**
3. Enter URL: `https://services.odata.org/V3/Northwind/Northwind.svc/`
4. Set **Implementation** to **2.0**
5. Click **OK**

### Step 2: Select Categories Table
1. In the Navigator window, find and check **Categories**
2. Click **Transform Data** (NOT Load)
3. This opens Power Query Editor

### Step 3: Configure Categories as Parameter Query
1. In Power Query Editor, select the **Categories** query
2. Right-click on the Categories query in the Queries pane
3. Select **Disable Load** (this prevents it from loading as a separate table)
4. This query will be used as a reference in the Products query

**Expected Columns in Categories:**
- CategoryID (Int64)
- CategoryName (String)
- Description (String)
- Picture (Binary)

---

## Part 2: Create the Orders Table

### Step 1: Connect to Orders (Same OData Source)
1. In Power Query Editor, click **Home** > **New Source** > **OData Feed**
2. Enter URL: `https://services.odata.org/V3/Northwind/Northwind.svc/`
3. Set **Implementation** to **2.0**
4. Click **OK**

### Step 2: Select Orders Table
1. In Navigator, check **Orders**
2. Click **OK**

### Step 3: Expand Order_Details
The Orders table has a nested table column called `Order_Details` that contains line item information.

1. Click on the expand icon (double arrows) in the **Order_Details** column header
2. Select only these columns:
   - ProductID
   - UnitPrice
   - Quantity
3. **Uncheck** "Use original column name as prefix"
4. Click **OK**
5. Columns will be added as: `Order_Details.ProductID`, `Order_Details.UnitPrice`, `Order_Details.Quantity`

### Step 4: Remove Unnecessary Columns
1. Select these columns: **Customer**, **Employee**, **Shipper**
2. Right-click and select **Remove Columns**

### Step 5: Add Calculated LineTotal Column
1. Click **Add Column** > **Custom Column**
2. Name: `LineTotal`
3. Formula:
   ```
   [Order_Details.UnitPrice] * [Order_Details.Quantity]
   ```
4. Click **OK**

### Step 6: Change LineTotal Data Type
1. Right-click the **LineTotal** column header
2. Select **Change Type** > **Fixed Decimal Number** (or Currency)

**Expected Final Columns in Orders:**
- OrderID (Int64)
- CustomerID (String)
- EmployeeID (Int64)
- OrderDate (DateTime)
- RequiredDate (DateTime)
- ShippedDate (DateTime)
- ShipVia (Int64)
- Freight (Double)
- ShipName (String)
- ShipAddress (String)
- ShipCity (String)
- ShipRegion (String)
- ShipPostalCode (String)
- ShipCountry (String)
- Order_Details.ProductID (Int64)
- Order_Details.UnitPrice (Double)
- Order_Details.Quantity (Int64)
- LineTotal (Currency)

---

## Part 3: Create the Products Table

### Step 1: Connect to Excel File
1. In Power Query Editor, click **Home** > **New Source** > **Excel Workbook**
2. Browse to and select `Products.xlsx`
3. Click **Open**

### Step 2: Select Products Sheet
1. In Navigator, check **Products** (the table/sheet)
2. Click **OK**

### Step 3: Change Column Types
Power BI may auto-detect types, but verify these:
1. Click **Home** > **Detect Data Type** if needed
2. Ensure these types:
   - ProductID: Whole Number
   - ProductName: Text
   - SupplierID: Whole Number
   - CategoryID: Whole Number
   - QuantityPerUnit: Text
   - UnitPrice: Decimal Number
   - UnitsInStock: Whole Number
   - UnitsOnOrder: Whole Number
   - ReorderLevel: Whole Number
   - Discontinued: True/False

### Step 4: Filter Out Discontinued Products
1. Click the dropdown arrow on the **Discontinued** column
2. **Uncheck** `true` (keep only false)
3. Click **OK**

### Step 5: Remove Discontinued Column
1. Right-click the **Discontinued** column
2. Select **Remove Columns**

### Step 6: Merge with Categories Query
This is where we combine data from two sources!

1. Click **Home** > **Merge Queries**
2. In the merge dialog:
   - Top table: `Products` (already selected)
   - Select column: **CategoryID**
   - Bottom dropdown: Select **Categories**
   - In Categories table, select: **CategoryID**
   - Join Kind: **Inner**
3. Click **OK**

A new column called `Categories` (nested table) is added.

### Step 7: Expand Categories Column
1. Click the expand icon (double arrows) on the **Categories** column
2. Select only:
   - CategoryName
   - Description
3. Check "Use original column name as prefix"
4. Click **OK**
5. Columns will be added as: `Categories.CategoryName` and `Categories.Description`

### Step 8: Rename Expanded Columns
1. Right-click **Categories.CategoryName** > **Rename**
2. Rename to: `Category`
3. Right-click **Categories.Description** > **Rename**
4. Rename to: `Category Description`

### Step 9: Remove CategoryID Column
1. Right-click the **CategoryID** column
2. Select **Remove Columns**

**Expected Final Columns in Products:**
- ProductID (Int64)
- ProductName (String)
- SupplierID (Int64)
- QuantityPerUnit (String)
- UnitPrice (Double)
- UnitsInStock (Int64)
- UnitsOnOrder (Int64)
- ReorderLevel (Int64)
- Category (String)
- Category Description (String)

---

## Part 4: Load Data to Model

### Step 1: Close & Apply
1. In Power Query Editor, click **Home** > **Close & Apply**
2. Wait for data to load

### Step 2: Verify Tables Loaded
In the Data view or Model view, you should see:
- **Orders** (18 columns)
- **Products** (10 columns)

---

## Part 5: Create Relationships

Power BI should auto-detect the main relationship, but verify:

### Step 1: Open Model View
1. Click the **Model** icon on the left sidebar

### Step 2: Verify Relationships
You should see these relationships (auto-created):

**Main Relationship:**
- From: `Orders[Order_Details.ProductID]`
- To: `Products[ProductID]`
- Cardinality: Many-to-One (*)
- Cross-filter direction: Single

**Date Relationships (Auto-created):**
Power BI creates hidden date tables for date columns:
- Orders[OrderDate] → LocalDateTable (hidden)
- Orders[RequiredDate] → LocalDateTable (hidden)
- Orders[ShippedDate] → LocalDateTable (hidden)

### Step 3: Create Relationship Manually (if needed)
If the Orders-Products relationship wasn't auto-detected:
1. Drag **Order_Details.ProductID** from Orders table
2. Drop onto **ProductID** in Products table
3. In the relationship dialog:
   - Cardinality: Many to One (*:1)
   - Cross filter direction: Single
   - Make this relationship active: **Checked**
4. Click **OK**

---

## Part 6: Create Category Hierarchy

### Step 1: Switch to Report View
1. Click the **Report** icon on the left sidebar

### Step 2: Create Hierarchy
1. In the **Data** pane (right side), find the **Products** table
2. Right-click on **Category** column
3. Select **New hierarchy**
4. A hierarchy called **Category Hierarchy** is created

### Step 3: Add ProductName to Hierarchy
1. Find **ProductName** in the Products table
2. Drag **ProductName** onto the **Category Hierarchy**
3. Drop it below Category

**Expected Hierarchy Structure:**
- Category Hierarchy
  - Category
  - ProductName

---

## Part 7: Test the Model

### Create a Sample Visual

**Test 1: Products by Category**
1. Add a **Matrix** visual
2. Add to Rows: **Category Hierarchy** (expand to ProductName)
3. Add to Values: **Orders[LineTotal]**
4. You should see categories with products and their sales

**Test 2: Sales Over Time**
1. Add a **Line Chart**
2. X-axis: **Orders[OrderDate]**
3. Y-axis: **Orders[LineTotal]**
4. You should see sales trends over time

**Test 3: Product Details**
1. Add a **Table** visual
2. Add columns:
   - Products[ProductName]
   - Products[Category]
   - Products[UnitPrice]
   - Products[UnitsInStock]

---

## Key Concepts Demonstrated

### Data Shaping Techniques
1. **Filtering Rows**: Removed discontinued products
2. **Removing Columns**: Eliminated unnecessary fields
3. **Expanding Nested Tables**: Flattened Order_Details
4. **Adding Calculated Columns**: Created LineTotal
5. **Changing Data Types**: Ensured proper column types

### Data Combining Techniques
1. **Merging Queries**: Combined Products with Categories
2. **Inner Join**: Kept only matching records
3. **Expanding Merged Columns**: Brought in category information
4. **Reference Queries**: Used Categories as a parameter query

### Data Modeling
1. **Relationships**: Connected Orders to Products
2. **Hierarchies**: Created drill-down capability
3. **Star Schema**: Fact table (Orders) with dimension (Products)

---

## Power Query M Code Reference

### Categories Query (Parameter)
```m
let
    Source = OData.Feed("https://services.odata.org/V3/Northwind/Northwind.svc/", null, [Implementation="2.0"]),
    Categories_table = Source{[Name="Categories",Signature="table"]}[Data]
in
    Categories_table
```

### Products Query
```m
let
    Source = Excel.Workbook(File.Contents("C:\Path\To\Products.xlsx"), null, true),
    Products_Table = Source{[Item="Products",Kind="Table"]}[Data],
    #"Changed Type" = Table.TransformColumnTypes(Products_Table,{{"ProductID", Int64.Type}, {"ProductName", type text}, {"SupplierID", Int64.Type}, {"CategoryID", Int64.Type}, {"QuantityPerUnit", type text}, {"UnitPrice", type number}, {"UnitsInStock", Int64.Type}, {"UnitsOnOrder", Int64.Type}, {"ReorderLevel", Int64.Type}, {"Discontinued", type logical}}),
    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each ([Discontinued] = false)),
    #"Removed Columns" = Table.RemoveColumns(#"Filtered Rows",{"Discontinued"}),
    #"Merged Queries" = Table.NestedJoin(#"Removed Columns", {"CategoryID"}, Categories, {"CategoryID"}, "Categories", JoinKind.Inner),
    #"Expanded Categories" = Table.ExpandTableColumn(#"Merged Queries", "Categories", {"CategoryName", "Description"}, {"Categories.CategoryName", "Categories.Description"}),
    #"Renamed Columns" = Table.RenameColumns(#"Expanded Categories",{{"Categories.CategoryName", "Category"}, {"Categories.Description", "Category Description"}}),
    #"Removed Columns1" = Table.RemoveColumns(#"Renamed Columns",{"CategoryID"})
in
    #"Removed Columns1"
```

### Orders Query
```m
let
    Source = OData.Feed("https://services.odata.org/V3/Northwind/Northwind.svc/", null, [Implementation="2.0"]),
    Orders_table = Source{[Name="Orders",Signature="table"]}[Data],
    #"Expanded Order_Details" = Table.ExpandTableColumn(Orders_table, "Order_Details", {"ProductID", "UnitPrice", "Quantity"}, {"Order_Details.ProductID", "Order_Details.UnitPrice", "Order_Details.Quantity"}),
    #"Removed Columns" = Table.RemoveColumns(#"Expanded Order_Details",{"Customer", "Employee", "Shipper"}),
    #"Added Custom" = Table.AddColumn(#"Removed Columns", "LineTotal", each [Order_Details.UnitPrice]*[Order_Details.Quantity]),
    #"Changed Type" = Table.TransformColumnTypes(#"Added Custom",{{"LineTotal", Currency.Type}})
in
    #"Changed Type"
```

---

## Troubleshooting Tips

**Issue: Categories merge returns no data**
- Ensure CategoryID columns match in both tables
- Verify Categories query has data before merging
- Check that both CategoryID columns are the same data type

**Issue: Relationship not working**
- Verify ProductID in Products is unique (no duplicates)
- Check that Order_Details.ProductID and ProductID have same data type
- Ensure relationship is active (solid line, not dotted)

**Issue: Can't load OData feed**
- Check internet connection
- Verify the OData URL is correct
- Try using Implementation="2.0" in advanced options

**Issue: Excel file not found**
- Update the file path in Products query
- Use absolute path: `C:\Users\...\Products.xlsx`
- Ensure the Excel file exists and is accessible

---

## Next Steps and Extensions

Once you've completed this exercise, try these enhancements:

1. **Add More Calculated Columns**
   - Profit margin in Products
   - Days to ship in Orders

2. **Create Measures**
   - Total Sales: `SUM(Orders[LineTotal])`
   - Average Order Value: `AVERAGE(Orders[LineTotal])`
   - Product Count: `COUNTROWS(Products)`

3. **Add More Tables**
   - Customers (from OData)
   - Employees (from OData)
   - Create additional relationships

4. **Build Reports**
   - Sales by Category
   - Top 10 Products
   - Sales by Customer
   - Shipping analysis

---

## Summary

You've successfully created a Power BI model that demonstrates:
- ✅ Connecting to multiple data sources (OData and Excel)
- ✅ Shaping data with filters and column operations
- ✅ Combining data through merge queries
- ✅ Creating calculated columns
- ✅ Building relationships in the data model
- ✅ Creating hierarchies for drill-down analysis
- ✅ Using parameter queries efficiently

This foundation prepares you for more advanced Power BI development!
