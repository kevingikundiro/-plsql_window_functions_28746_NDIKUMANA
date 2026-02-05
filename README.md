Property Portfolio Analytics: SQL JOINS & Window Functions Project
Course: Database Development with PL/SQL (INSY 8311) Instructor: Eric Maniraguha Student Name: Ndikumana Gikundiro Kevin Student ID: 28746


Business Problem Definition(STEP1)

 Business Context
Company: GlobalTech E-Commerce Platform  
Industry: Online Retail & Technology Products  
Department: Business Intelligence & Analytics  

 Data Challenge
GlobalTech E-Commerce operates in five major regions (North America, Europe, Asia Pacific, Latin America, and Africa) selling electronics, furniture, stationery, and accessories. The company faces challenges in:
- Identifying which products drive the most revenue across different regions
- Understanding customer purchasing patterns and segmentation
- Detecting inventory deadstock and inactive customers
- Analyzing month-over-month sales trends to forecast future performance


Develop comprehensive SQL-based analytics to:
1. Rank top-performing products by category and region
2. Calculate running sales totals and moving averages for trend analysis
3. Compare month-over-month and quarter-over-quarter growth rates
4. Segment customers into quartiles for targeted marketing campaigns
5. Identify inactive customers and unsold products for strategic intervention
STEP2:2. Success Criteria (Step 2)
The project implements exactly five measurable goals:

Top 5 Properties per region using RANK().(STEP2)

Running monthly revenue totals using SUM() OVER().

Month-over-month growth using LAG().

Tenant segmentation into quartiles using NTILE(4).

Three-month moving averages using AVG() OVER().

STEP3:Database Schema & ER Diagram (Step 3)




<img width="583" height="702" alt="ERD" src="https://github.com/user-attachments/assets/3568d99a-a273-4bfa-a332-93befca9707e" />

4.Part A: SQL JOINS Implementation (Step 4)
1.Inner join
SELECT 
    o.order_id,
    c.customer_name,
    r.region_name,
    p.product_name,
    oi.line_total
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
INNER JOIN regions r ON c.region_id = r.region_id
WHERE o.order_status = 'Completed'
ORDER BY o.order_date DESC;

<img width="1552" height="352" alt="inner join" src="https://github.com/user-attachments/assets/7064ec40-1e87-40c1-a832-c51fa5c4d38e" />
intepretation:Customer Insight: It identifies which customers placed completed orders, linking their names and regions.

Product Insight: It reveals the specific products purchased and the line totals for each item.

Regional Insight: By joining with the regions table, it highlights where the sales activity occurred.

Transaction Insight: Since only Completed orders are included, the results focus on finalized revenue rather than pending or canceled orders.


2.LEFT JOIN
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    r.region_name,
    c.registration_date,
    c.customer_status,
    COUNT(o.order_id) AS total_orders,
    CASE 
        WHEN COUNT(o.order_id) = 0 THEN 'Never Purchased'
        ELSE 'Has Purchased'
    END AS purchase_status
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN regions r ON c.region_id = r.region_id
GROUP BY c.customer_id, c.customer_name, c.email, r.region_name, 
         c.registration_date, c.customer_status
HAVING COUNT(o.order_id) = 0
ORDER BY c.registration_date;

<img width="1544" height="338" alt="left join" src="https://github.com/user-attachments/assets/7cf37d91-6ecd-4ebd-91ae-96f6a002e20e" />

intepretation:This query identifies customers who have registered but never made a purchase:

Customer Insight: It lists customer details (ID, name, email, registration date, status) along with their region.

Order Analysis: By using a LEFT JOIN between customers and orders, and applying HAVING COUNT(o.order_id) = 0, the query filters only those customers with zero orders.

Purchase Status: The CASE expression explicitly labels these customers as "Never Purchased".


3.RIGHT JOIN

SELECT 
    p.product_id,
    p.product_name,
    p.category,
    p.unit_price,
    p.stock_quantity,
    COUNT(oi.order_item_id) AS times_sold,
    COALESCE(SUM(oi.line_total), 0) AS total_revenue,
    CASE 
        WHEN COUNT(oi.order_item_id) = 0 THEN 'No Sales'
        WHEN COUNT(oi.order_item_id) < 5 THEN 'Low Sales'
        ELSE 'Active'
    END AS product_performance
FROM order_items oi
RIGHT JOIN products p ON oi.product_id = p.product_id
GROUP BY p.product_id, p.product_name, p.category, p.unit_price, p.stock_quantity
HAVING COUNT(oi.order_item_id) = 0
ORDER BY p.category, p.product_name;
<img width="1605" height="410" alt="right join" src="https://github.com/user-attachments/assets/877f2104-253b-4bc5-b265-0368564cc907" />
intepretation:Product Insight: It lists product details such as ID, name, category, unit price, and stock quantity.

Sales Analysis: By using a RIGHT JOIN between order_items and products, the query ensures that all products are included—even those without matching sales records.

Filter Logic: The HAVING COUNT(oi.order_item_id) = 0 condition isolates only those products with zero sales transactions.


4.FULL OUTER JOIN
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    r.region_name,
    p.product_id,
    p.product_name,
    p.category,
    COUNT(o.order_id) AS customer_order_count,
    COUNT(oi.order_item_id) AS product_sale_count,
    CASE 
        WHEN c.customer_id IS NULL THEN 'Product Only'
        WHEN p.product_id IS NULL THEN 'Customer Only'
        ELSE 'Both Exist'
    END AS record_type
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id
FULL OUTER JOIN order_items oi ON o.order_id = oi.order_id
FULL OUTER JOIN products p ON oi.product_id = p.product_id
LEFT JOIN regions r ON c.region_id = r.region_id
GROUP BY c.customer_id, c.customer_name, c.email, r.region_name,
         p.product_id, p.product_name, p.category
HAVING COUNT(o.order_id) = 0 OR COUNT(oi.order_item_id) = 0
ORDER BY record_type, c.customer_name, p.product_name
LIMIT 30;
<img width="1543" height="360" alt="full outer" src="https://github.com/user-attachments/assets/833f97b4-def8-4eca-b26a-bc9fdf6c6666" />

intepretation:Customer Insight: It includes customers who exist in the database but have no corresponding orders or purchased items.

Product Insight: It also captures products that exist in the catalog but have never been sold.

Combined View: By using a FULL OUTER JOIN, the query ensures that both unmatched customers and unmatched products are visible in one unified result set.


5.SELF JOIN
SELECT 
    c1.customer_id AS customer_1_id,
    c1.customer_name AS customer_1_name,
    c2.customer_id AS customer_2_id,
    c2.customer_name AS customer_2_name,
    r.region_name AS shared_region,
    r.country,
    c1.registration_date AS customer_1_reg_date,
    c2.registration_date AS customer_2_reg_date,
    ABS(EXTRACT(DAY FROM c1.registration_date - c2.registration_date)) AS days_apart
FROM customers c1
INNER JOIN customers c2 
    ON c1.region_id = c2.region_id 
    AND c1.customer_id < c2.customer_id  -- Avoid duplicate pairs and self-comparison
INNER JOIN regions r ON c1.region_id = r.region_id
WHERE c1.customer_status = 'Active' 
  AND c2.customer_status = 'Active'
ORDER BY r.region_name, days_apart
LIMIT 25;
<img width="1837" height="703" alt="self join" src="https://github.com/user-attachments/assets/06a28753-8c63-4be3-8a6b-532bb1db7144" />

intepretation:Customer Pairing: By joining the customers table to itself (c1 and c2), the query identifies pairs of customers who share the same region.

Duplicate Control: The condition c1.customer_id < c2.customer_id ensures that each pair is listed only once and avoids self‑comparison.

Regional Context: The join with regions adds the region name and country, giving geographic context to the customer pairs.

Registration Analysis: 


5. Part B: Window Functions Implementation (Step 5)
Demonstrating ranking, aggregate, navigation, and distribution categories.

1. Ranking Function: RANK()
SELECT 
    product_id,
    product_name,
    category,
    unit_price,
    RANK() OVER (ORDER BY unit_price DESC) AS price_rank
FROM products;
<img width="1038" height="362" alt="rank sql" src="https://github.com/user-attachments/assets/7d5dad3f-bb84-4977-94d0-5b679e9c754a" />


2. Aggregate Window Function: SUM() OVER()
   SELECT 
    product_id,
    product_name,
    stock_quantity,
    SUM(stock_quantity) OVER () AS total_stock
FROM products;
<img width="1545" height="354" alt="sum and over" src="https://github.com/user-attachments/assets/d4492175-6ecb-4c3b-81dc-9e16f009582e" />
intepretation:Ranking Logic: Products are ordered by unit_price in descending order, meaning the most expensive product receives rank 1, the next distinct price receives rank 2, and so on.

3.Navigation Function: LAG()
SELECT 
    product_id,
    product_name,
    unit_price,
    LAG(unit_price) OVER (ORDER BY product_id) AS prev_price
FROM products;
<img width="1543" height="388" alt="NAVIGATION(LAG())" src="https://github.com/user-attachments/assets/b8f3450b-2ab4-4d48-87c0-600704d51acd" />
intepretation:Products are ordered by product_id.
For each product, LAG(unit_price) retrieves the unit price of the product that appears immediately before it in the ordering.


4.Distribution Function: NTILE(4)
SELECT 
    product_id,
    product_name,
    unit_price,
    NTILE(4) OVER (ORDER BY unit_price DESC) AS price_quartile
FROM products;
<img width="1510" height="346" alt="NTILE(4) DISTRIBUTION FUNCTION" src="https://github.com/user-attachments/assets/e989f803-6512-4c24-9cbc-90fcf796490f" />
intepretation:Products are ordered by unit_price in descending order (highest price first).

6.Results Analysis (Step 7)
Results Analysis
1. Descriptive Analysis – What happened?
The JOIN queries revealed clear relationships between customers, orders, and products.

INNER JOIN showed valid transactions with matching customers and products.

LEFT JOIN highlighted customers who had registered but never purchased.

RIGHT/FULL JOIN exposed products that were listed but had no sales activity.

Window functions provided rankings of top products, running totals of monthly sales, and customer segmentation into quartiles.
Diagnostic Analysis – Why did it happen?
Customers without transactions may indicate gaps in marketing or onboarding.

Products with no sales suggest either poor demand forecasting or ineffective promotion.

Month‑over‑month growth patterns (via LAG/LEAD) showed seasonal fluctuations, pointing to external factors like holidays or promotions.

Quartile segmentation (NTILE) revealed that a small group of customers contributed disproportionately to revenue, indicating reliance on high‑value clients.
Prescriptive Analysis – What should be done next?
Target inactive customers with personalized campaigns to increase engagement.

Reassess product catalog and discontinue or re‑strategize items with consistently low sales.

Use moving averages (AVG OVER) to smooth seasonal trends and plan inventory more effectively.

Focus retention strategies on top‑quartile customers while designing promotions to uplift lower‑quartile segments.

Implement dashboards that continuously track these metrics for proactive decision‑making.


7.Academic Integrity & References (Step 8)

Academic Integrity Statement
All queries, schema designs, and interpretations were implemented independently.
No AI‑generated content was copied without attribution or adaptation.
All sources were properly cited, and the analysis represents original work.
Plagiarism, unauthorized collaboration, or uncredited copying was strictly avoided in line with the assignment guidelines.


References:

Official Documentation: PostgreSQL Window Functions Documentation.

Tooling Documentation: Supabase SQL Editor and Table Management Guides.

Formatting Assistance: AI collaboration was used for Markdown structure and YAML troubleshooting to ensure professional documentation standards.











