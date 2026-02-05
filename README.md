Business Problem Definition

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

Top 5 Properties per region using RANK().

Running monthly revenue totals using SUM() OVER().

Month-over-month growth using LAG().

Tenant segmentation into quartiles using NTILE(4).

Three-month moving averages using AVG() OVER().

STEP3:Database Schema & ER Diagram (Step 3)























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


