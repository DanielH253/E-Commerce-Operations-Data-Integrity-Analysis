# E-Commerce Operations & Data Engineering Analysis
An end-to-end data cleaning and analysis project using Excel, Power Query, and Power Pivot to analyze e-commerce orders, payments, and refunds.

![Dashboard Preview](images/Dashboard.png)

## Project Overview
This project demonstrates an end-to-end data pipeline: from **complex ETL (Extract, Transform, Load)** to visualization and analysis in Excel. 

The core challenge was to extract actionable business insights from a "poisoned" dataset. I used a Python script to intentionally inject real-world data issues—such as logical paradoxes, duplicate records, and inconsistent formatting—to test the robustness of the **Power Query cleaning engine** and **Power Pivot data model**.


---

## Tools Used

- Microsoft Excel
- Power Query (data cleaning and transformation)
- Power Pivot (data modeling and measures)
- Pivot Tables
- Excel Dashboard Design

---

## Dataset

The dataset includes three primary tables:

Orders
- Order ID
- Order Date
- Customer Name
- Email
- Country
- Product Name
- Quantity
- Price
- Payment Status
- Shipping Date

Payments
- Payment ID 
- Order ID 
- Amount Paid
- Currency
- Payment Date

Refunds
- Order ID 
- Refund Amount
- Refund Reason

---

## The "Chaos" Simulation (Python Engineering)
The source data was generated using a custom Python script (`Faker` & `Pandas`) to simulate the messy reality of raw e-commerce exports.

### Injected Data Challenges:
1.	**Logistical Paradoxes:** Delivery dates occurring before shipping dates via `timedelta(days=random.randint(-2, 12))`.
2.	**Semantic Noise:** 12+ variations of country names (e.g., "U.S.", "USA", "united states") and randomized string casing.
3.  **Integrity Gaps:** Intentional 1.5% row duplication in orders and payments, plus randomized null values in critical fields.

---

## The Transformation Pipeline (Power Query & Power Pivot)
To recover the truth from the noise, A multi-stage transformation engine was built using **Power Query**:

1.  **Data Cleaning:** The Orders and Payment tables went through a transformation process to clean unnecessary/junk data, including the removal of ID duplicates on both tables, replacing missing (null) orders with "1" (a reasonable assumption based on the fact that if an order exists in the first place, it stands to reason that at least one unit was made for that order), cleaning unwanted spaces from the Customer Name column, as well as correctly capitalizing first and last names, and finally, currency and payment status were standardized ("paid" or "PAID" -> Paid/"usd", "Usd" -> USD).

![Power Query Cleaning Steps](images/1-Power%20Query%20Steps%20(Orders).png)

2.	**Metric Creation:** Gross Revenue was calculated by multiplying Quantity x Price in this table, to have this ready for the creation of the fact table later on.

3.	**Country Standardization:** The Country column had inconsistent names for each country, such as U.S., US, united states, and so on. For this, a unique list of countries was extracted and loaded into a Mapping Reference, to ensure that all of the inconsistent names were all in one table. Then, the proper dimension table was made to categorize each country into a single, standardized name (United States, United Kingdom, Canada). This way, if a new, unique record is entered, one can simply add it to the mapping table, whereupon Power Query will automatically update the entered unique name into the list of admitted records for one of the countries in the dataset.

![Country Mapping Logic](images/Country%20Map%20Table.png)

4.	**Country Lookup Table Merge:** The standard, messy Country column was merged with the standardized, clean country table, so each name was mapped to the correct country. Evidently, only the clean Country column was kept, while the messy Country column was discarded.

5. **Finishing Touches for Fact Table:** Before the final table was created, each of the three source tables were each double-checked to ensure they had the correct data types, and renamed each column to have more presentable naming conventions (capitalized names without underscores). For the Refunds table, nulls were kept in the Refund Amount column, mainly to ensure there is one way to differentiate Unpaid records later.

6. **Fact Table Creation:** Both the Payment and Refund tables were joined with the Orders table, using Order ID as the common point between all three for joining. Four columns were added after the joins, those being Payment Date, Amount Paid, Refund Reason, and Refund Amount. Net Revenue was calculated by subtracting Amount Paid from Refund Amount. Since both columns had nulls, a custom code was created to handle said nulls, converting them to 0, so null arithmetic could be handled seamlessly. A similar technique was employed to create the Order Status column, which tracks the state of each individual order. 

Additionally, a helper column was created to categorize delays in payment. The reader can observe that there are a lot of "Data Error" records, which is the primary reason for the creation of this column. Such errors were caused by logical paradoxes, such as delays in payment being represented with negative numbers, which, in simple terms,this would suggest "time travel", which is impossible. This is due to the script used to generate this dataset returning such errors. While it is unlikely, but not impossible, that the errors may arise in real world scenarios in such volume, this project aims to demonstrate the capability to handle errors of such magnitude, if and when they do appear in real world cases.

Time intelligence metrics were created: Month, Month number (this column is for sorting the normal Month column, as otherwise it would show months in alphabetical order instead of the correct order), Year, and Year-Month (while this dataset only has records for 2024, it was still created in case records from different years were added in the future, effectively future-proofing the dataset if the need arises).

![Fact Table ETL Process](images/5-Power%20Query%20Steps%20(Fact%20Table).png)


Lastly, nulls in the Refund Reason column were replaced with "No Refund", to give the nulls meaning. To finish up, columns were renamed, reordered, and data types were validated. 

7.	**Importing to Data Model:** The fact table was loaded into Excel's data model (Power Pivot) to create Measures, as well as sorting month by its correct month index. Among the Measures created, these include: Total Gross Revenue, Total Net Revenue, Total Refund Amount, Total Orders, among others. A Measure for Data Accuracy % was created as well, its purpose being to showcase how much of the programmatically generated data did not have paradoxical logic errors, specifically negative days in payment delays. We can observe that the accuracy is 53.50%. This not only showcases transparency, but also an ability to spot possible data errors in datasets.

![DAX Measures and Data Model](images/Power%20Pivot%20Measures.png)

8.	**Final Report:** Once the transformation and modeling process was complete, creation of Pivot Tables to show key metrics (shown below) and the finalized dashboard could begin, ready for the final dashboard and report presentation.
---

## Business Insights & Key Metrics
Despite the intentional data corruption, the final dashboard reveals high-level operational truths:

| Metric | Value | Insight |
| :--- | :--- | :--- |
| **Gross Revenue** | $217,609.36 | Total volume before cleaning and refunds. |
| **Net Revenue** | $106,754.85 | The "Real" revenue after filtering junk data and returns. |
| **Data Accuracy %** | **53.5%** | Percentage of records that survived all integrity filters. |
| **Avg. Payment Delay**| **101.2 Days** | A significant cash-flow bottleneck identified in the logistics chain. |

### Refund Root-Cause Analysis
The analysis identified that **Damage ($2,304.80)** accounts for **45% of total refund value**. This suggests that revenue loss is driven by shipping/packaging issues rather than product quality (Customer Returns).

---

## Skills Demonstrated
- Data Cleaning
- Data Modeling
- KPI Development
- Business Metric Analysis
- Dashboard Design