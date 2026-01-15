## Power BI Data Transformation and Merge Instructions

### **Step 1: Download Data**

*   Go to **Data.Medicaid.gov** and download the **weekly drug price data** at **NADAC (National Average Drug Acquisition Cost) 2025** https://data.medicaid.gov/dataset/f38d0706-1239-442c-a3cc-40ef1b686ac0.
*   Click **Download full dataset (CSV)**.
*   Note: NDC Codes are provided, but **drug manufacturer information is missing**.
*   Download the **Drug Manufacturer Contacts** CSV file separately from https://data.medicaid.gov/dataset/9fcb14ec-d5f0-536e-9938-3f0024531e5b.

***

### **Step 2: Load and Profile NADAC Data**

*   Load the **NADAC CSV file** into Power BI by clicking Get Data, choosing Text/CSV in the dropdown and selecting the downloaded file.
*   Click **Transform Data** to open Power Query.
*   In the **View** menu, enable:
    *   **Column Quality**
    *   **Column Distribution**
    *   **Column Profile**
*   **Important:** Column profiling defaults to the first 1,000 rows.  
    To profile the entire dataset:
    *   Click the message at the bottom of the screen.
    *   Be aware this may be slow for large tables.

***

### **Step 3: Rename and Transform Columns**

*   Change the **Query Name** to `Drug Costs`.
*   For the **NDC column**:
    *   Change its type to **Text**.
    *   Split it after 5 characters.
*   Rename the split column:
    *   `NDC.1` → `Labeler NDC`.
    *   Ensure its type is **Text**.
*   Note that leading 0s are still dropped, for example row 10 shows 378 instead of 00378. To fix this delete the step "Chnaged Type1" that Power BI added after the Split Column by Position step. 

***

### **Step 4: Load Manufacturer Contacts**

*   Load the **Drug Manufacturer Contacts** CSV file.
*   Change the **Labeler Code** column type to **Text**.

***

### **Step 5: Merge Queries**

*   In **Drug Costs**, click **Merge Queries** (Home menu).
*   Select `3Q2025LabelerContacts` as the second table.
*   Match:
    *   **Labeler NDC** (first table)
    *   **Labeler Code** (second table)
*   Keep **Join Kind** as **Left Outer**.
*   Wait for match count to appear at the bottom, then click **OK**.

***

### **Step 6: Expand Columns**

*   Click the **expander icon** on the rightmost column.
*   Select:
    *   `Labeler Name`
    *   `Legal Corporation`
*   Uncheck **Use original column name as prefix**.

***

### **Step 7: Apply Filter**

*   Add a **Text Filter** on `Legal Corporation`:
    *   **Begins With** = `"BRISTOL"`  
        *(Note: Power Query is case-sensitive)*

***

### **Step 8: Disable Load for Contacts Table**

*   Right-click `3Q2025LabelerContacts`.
*   Uncheck **Enable Load**.

***

