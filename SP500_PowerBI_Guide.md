# Visualizing S&P 500 Returns by Decade and Year in Power BI

## **Objective**
Visualize the S&P 500 returns since 1970 by decade and year. Enable users to:
- Filter on a range of years.
- Drill down in the graph from decade to individual years.

---

## **Steps**

### **1. Install Power BI Desktop**
- Download Power BI Desktop from the Microsoft Store:
  - Direct link: https://aka.ms/pbidesktopstore
  - Or open Microsoft Store and search for **Power BI Desktop**.
- Install and launch the application.

---

### **2. Connect to Data**
- Open **Power BI Desktop**.
- Click **Get Data** → Select **Web** under *Common Data Sources*.
- Enter the following URL:  
  `https://www.slickcharts.com/sp500/returns`
- Click **OK** and choose **Table 1** from the available tables.

---

### **3. Transform Data**
- Click **Transform Data** to open Power Query Editor.
- Apply the following transformations:
  - **Filter rows**: Set `Year >= 1970`.
  - **Add Decade Column**:
    - Use **Column From Examples**.
    - For example:
      - For year `2020` and later → Enter `2020s`.
      - For year `2019` → Enter `2010s`.
    - Power BI will auto-fill the decade for all rows.
- Click **Close & Apply** to load the transformed data.

---

### **4. Adjust Table Settings**
- Switch to **Table View**:
  - Change **Return** column:
    - Format to **2 decimal places**.
    - Set aggregation to **Average**.
  - Change **Year** column:
    - Set to **No summarization**.

---

### **5. Create Hierarchy**
- Create a new hierarchy called **Decade Hierarchy**:
  - Add **Decade** as the top level.
  - Add **Year** under Decade.

---

### **6. Build Visualization**
- Switch to **Report View**.
- Add a **Stacked Column Chart**:
  - **X-axis**: Decade Hierarchy.
  - **Y-axis**: Average of Total Return.
- Format the visual:
  - Turn **Data Labels** ON for clarity.

---

### **7. Add Interactivity**
- Add a **Slicer** for Year:
  - This allows filtering by a range of years.
- Enable **Drill Down** on the chart:
  - Users can click on a decade to view yearly returns.

---

## **Result**
You will have an interactive Power BI report that:
- Displays S&P 500 returns by decade.
- Allows drill-down to yearly data.
- Supports filtering by year range.
