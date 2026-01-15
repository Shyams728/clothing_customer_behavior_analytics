# DAX Measures for Clothing Customer Behavior Analytics

This file documents a collection of DAX measures for analyzing customer behavior in the "public customer" table. Copy these measures into your Power BI model (or Analysis Services tabular model). Replace or adapt column names if your schema differs.

## How to use
- Create each measure in your model under the measure table or the 'public customer' table.
- The measures reference the table name 'public customer' and columns like [customer_id], [purchase_amount], [gender], [age_group], etc. Update column names if necessary.
- Where a Date table is referenced, ensure you have a proper date table named 'Date' with a [Date] column.

## DAX Measures

```dax
-- CUSTOMER BASE METRICS --
Total Customers = DISTINCTCOUNT('public customer'[customer_id])

Customers by Gender = 
CALCULATE(
    [Total Customers],
    ALLEXCEPT('public customer', 'public customer'[gender])
)

Customers by Age Group = 
CALCULATE(
    [Total Customers],
    ALLEXCEPT('public customer', 'public customer'[age_group])
)

Customer Distribution % = 
DIVIDE(
    [Total Customers],
    CALCULATE([Total Customers], ALL('public customer'))
)

-- SALES & REVENUE METRICS --
Total Revenue = SUM('public customer'[purchase_amount])

Avg Purchase Amount = AVERAGE('public customer'[purchase_amount])

Avg Purchase by Category = 
CALCULATE(
    [Avg Purchase Amount],
    ALLEXCEPT('public customer', 'public customer'[category])
)

Revenue per Customer = 
DIVIDE([Total Revenue], [Total Customers])

Total Discounts = SUM('public customer'[discount_applied])

Discount Rate = 
DIVIDE(
    [Total Discounts],
    [Total Revenue] + [Total Discounts]
)

-- PURCHASE BEHAVIOR METRICS --
Total Purchases = COUNTROWS('public customer')

Avg Previous Purchases = AVERAGE('public customer'[previous_purchases])

Purchase Frequency = 
DIVIDE(
    [Total Purchases],
    DISTINCTCOUNT('public customer'[customer_id])
)

Repeat Purchase Rate = 
DIVIDE(
    COUNTROWS(
        FILTER(
            SUMMARIZE('public customer', 'public customer'[customer_id], "PurchaseCount", COUNTROWS('public customer')),
            [PurchaseCount] > 1
        )
    ),
    [Total Customers]
)

-- CUSTOMER SATISFACTION METRICS --
Avg Review Rating = AVERAGE('public customer'[review_rating])

Rating Distribution = 
VAR RatingValues = VALUES('public customer'[review_rating])
RETURN
ADDCOLUMNS(
    RatingValues,
    "Customer Count", CALCULATE([Total Customers]),
    "Percentage", DIVIDE(
        CALCULATE([Total Customers]),
        [Total Customers]
    )
)

High Satisfaction Customers = 
CALCULATE(
    [Total Customers],
    'public customer'[review_rating] >= 4
)

Satisfaction Rate = 
DIVIDE([High Satisfaction Customers], [Total Customers])

-- CATEGORY & PRODUCT ANALYSIS --
Top Category by Revenue = 
VAR MaxRevenue = MAXX(ALL('public customer'[category]), [Total Revenue])
RETURN
CONCATENATEX(
    FILTER(
        SUMMARIZE(
            'public customer',
            'public customer'[category],
            "Revenue", [Total Revenue]
        ),
        [Revenue] = MaxRevenue
    ),
    'public customer'[category],
    ", "
)

Category Market Share = 
DIVIDE(
    [Total Revenue],
    CALCULATE([Total Revenue], ALL('public customer'[category]))
)

-- SEASONAL ANALYSIS --
Revenue by Season = 
CALCULATE(
    [Total Revenue],
    ALLEXCEPT('public customer', 'public customer'[season])
)

Seasonal Growth = 
VAR CurrentSeasonRevenue = [Total Revenue]
VAR PreviousSeasonRevenue = 
    CALCULATE(
        [Total Revenue],
        PREVIOUSQUARTER('Date'[Date])  -- Requires a date table named 'Date'
    )
RETURN
DIVIDE(
    CurrentSeasonRevenue - PreviousSeasonRevenue,
    PreviousSeasonRevenue
)

-- CUSTOMER SEGMENTATION --
Active Customers (Last 90 Days) = 
CALCULATE(
    [Total Customers],
    DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -90, DAY)
)

Subscription Rate = 
DIVIDE(
    CALCULATE([Total Customers], 'public customer'[subscription_status] = "Yes"),
    [Total Customers]
)

-- PAYMENT & SHIPPING ANALYSIS --
Preferred Payment Method = 
VAR MaxCount = MAXX(ALL('public customer'[payment_method]), [Total Customers])
RETURN
CONCATENATEX(
    FILTER(
        SUMMARIZE(
            'public customer',
            'public customer'[payment_method],
            "CustomerCount", [Total Customers]
        ),
        [CustomerCount] = MaxCount
    ),
    'public customer'[payment_method],
    ", "
)

Avg Shipping Time (Estimated) = 
SWITCH(
    SELECTEDVALUE('public customer'[shipping_type]),
    "Express", 2,
    "Standard", 7,
    "Next Day", 1,
    5  -- Default
)

-- CUSTOMER LIFETIME VALUE (CLV) ESTIMATE --
Avg CLV Estimate = 
[Avg Purchase Amount] * [Avg Previous Purchases] * [Purchase Frequency]

-- COLOR & SIZE PREFERENCE --
Most Popular Color = 
VAR MaxColor = MAXX(ALL('public customer'[color]), CALCULATE([Total Purchases]))
RETURN
CONCATENATEX(
    FILTER(
        SUMMARIZE(
            'public customer',
            'public customer'[color],
            "PurchaseCount", [Total Purchases]
        ),
        [PurchaseCount] = MaxColor
    ),
    'public customer'[color],
    ", "
)

Size Distribution = 
ADDCOLUMNS(
    VALUES('public customer'[size]),
    "Customer Count", CALCULATE([Total Customers]),
    "Percentage", DIVIDE(
        CALCULATE([Total Customers]),
        [Total Customers]
    )
)

-- LOYALTY METRICS --
Loyalty Score = 
AVERAGEX(
    SUMMARIZE(
        'public customer',
        'public customer'[customer_id],
        "Score", 
        [Avg Previous Purchases] * 0.4 + 
        [Avg Review Rating] * 0.3 + 
        [Purchase Frequency] * 0.3
    ),
    [Score]
)

-- LOCATION ANALYSIS --
Revenue by Location = 
CALCULATE(
    [Total Revenue],
    ALLEXCEPT('public customer', 'public customer'[location])
)

Top Performing Location = 
VAR MaxRevenueLocation = MAXX(ALL('public customer'[location]), [Revenue by Location])
RETURN
CONCATENATEX(
    FILTER(
        SUMMARIZE(
            'public customer',
            'public customer'[location],
            "Revenue", [Revenue by Location]
        ),
        [Revenue] = MaxRevenueLocation
    ),
    'public customer'[location],
    ", "
)  
``` 

## Notes & Adjustments
- If your model uses a different Date table name, update the DAX measures that reference 'Date'.
- Some measures (e.g., Rating Distribution, Size Distribution) produce tables; use them in visuals like tables or matrix visuals.
- Validate calculated measures that use other measures as dependencies after adding them to ensure evaluation order is correct.

---
