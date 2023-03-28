## SQL Website Traffic Analysis

**Project description:** Website traffic analysis using SQL. Pulling monthly trends, checking the performance of various campaigns, checking conversion rates of landing pages, and doing full funnel conversion analysis. I used Google Looker Studio to visualize the results that I got. I also used MYSQL to do my queries and manage my database. 

## The dataset that is used here is comprised of the following tables: 
<ul>
  <li>order_items_refunds</li>
  <li>order_items</li>
  <li>orders</li>
  <li>products</li>
  <li>website_pageviews</li>
  <li>website_sessions</li>
</ul>

## Click on the images to view the reports on Google Looker Studio.<br>
Limitations: Data is from March 2012 to November 2012 (2012-11-27). 

### 1.  Pull monthly trends for gsearch sessions and orders so that we can showcase the growth here.

```SQL
SELECT
    YEAR(website_sessions.created_at) AS year,
    MONTH(website_sessions.created_at) AS month, 
    COUNT(DISTINCT website_sessions.website_session_id) AS monthly_sessions,
    COUNT(orders.items_purchased) AS total_orders
FROM 
	website_sessions 
		LEFT JOIN orders 
			ON website_sessions.website_session_id = orders.website_session_id 
WHERE 
	website_sessions.utm_source = 'gsearch'
    AND website_sessions.created_at < '2012-11-27'
GROUP BY 
	1,2;
```
<a href="https://lookerstudio.google.com/reporting/0eefca3d-9f2e-4e06-8a2d-11b8aa40fb49">
<img src="images/Q1.jpg?raw=true"/>
</a>

### 2. Similar monthly trend for gsearch but splitting out the brand and nonbrand campaigns. 

```SQL
SELECT
    website_sessions.utm_campaign,
    MONTH(website_sessions.created_at) AS month, 
    COUNT(DISTINCT website_sessions.website_session_id) AS monthly_sessions,
    COUNT(orders.items_purchased) AS total_orders
FROM 
	website_sessions 
		LEFT JOIN orders 
			ON website_sessions.website_session_id = orders.website_session_id 
WHERE 
	website_sessions.utm_source = 'gsearch'
    AND website_sessions.created_at < '2012-11-27'
    AND website_sessions.utm_campaign IN ('nonbrand', 'brand')
GROUP BY 
	1, 2;
```
<a href="https://lookerstudio.google.com/reporting/e59a198e-cc2d-478d-b659-3dece8702153">
<img src="images/Q22.jpg?raw=true"/>
</a>
<img src="images/Q222.jpg?raw=true"/>

### 3. Checking gsearch and nonbrand monthly trend for sessions and orders filtered by device type. 

```SQL
SELECT
    website_sessions.device_type,
    MONTH(website_sessions.created_at) AS month, 
    COUNT(DISTINCT website_sessions.website_session_id) AS monthly_sessions,
    COUNT(orders.items_purchased) AS total_orders
FROM 
	website_sessions 
		LEFT JOIN orders 
			ON website_sessions.website_session_id = orders.website_session_id 
WHERE 
	website_sessions.utm_source = 'gsearch'
    AND website_sessions.created_at < '2012-11-27'
    AND website_sessions.utm_campaign IN ('nonbrand')
GROUP BY 
	1, 2;
```
<a href="https://lookerstudio.google.com/reporting/8e3734e2-4c7e-41f9-8f1d-92df96bdc59e">
<img src="images/Q3.jpg?raw=true"/>

### 4. Comparing website traffic sources (paid vs organic searches)

```SQL
SELECT
    YEAR(website_sessions.created_at) AS year,
    MONTH(website_sessions.created_at) AS month, 
    COUNT(DISTINCT CASE WHEN website_sessions.utm_source = 'gsearch' THEN website_sessions.website_session_id ELSE NULL END) AS gsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN website_sessions.utm_source = 'bsearch' THEN website_sessions.website_session_id ELSE NULL END) AS bsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN website_sessions.utm_source IS NULL AND http_referer IS NOT NULL THEN website_sessions.website_session_id ELSE NULL END) AS organic_search_sessions,
    COUNT(DISTINCT CASE WHEN website_sessions.utm_source IS NULL AND http_referer IS NULL THEN website_sessions.website_session_id ELSE NULL END) AS direct_type_in_sessions
FROM 
	website_sessions 
		LEFT JOIN orders 
			ON website_sessions.website_session_id = orders.website_session_id 
WHERE 
    website_sessions.created_at < '2012-11-27'
GROUP BY 
	1, 2;
```
<a href="https://lookerstudio.google.com/reporting/4ab05325-c81c-479e-8698-d547e7378491">
<img src="images/Q4.jpg?raw=true"/>

### 5. Pulling sessions to orders conversion rates per month for the first 8 months of the website. 
```SQL
CREATE TEMPORARY TABLE session_to_ty
SELECT
	website_sessions.website_session_id, 
	website_pageviews.pageview_url, 
	website_pageviews.created_at AS pageview_created_at,
    CASE WHEN website_pageviews.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END as ty_page
FROM 
	website_sessions
		LEFT JOIN website_pageviews 
			ON website_sessions.website_session_id = website_pageviews.website_session_id 
WHERE
	 website_sessions.created_at < '2012-11-27'
GROUP BY 
	1, 2, 3;

SELECT* 
FROM session_to_ty;

-- create temp table for only the sessions that made it to ty page
CREATE TEMPORARY TABLE conversion_table
SELECT 
	website_session_id, 
    MAX(ty_page) AS TY
FROM (
SELECT
	website_sessions.website_session_id, 
	website_pageviews.pageview_url, 
	website_pageviews.created_at AS pageview_created_at,
    CASE WHEN website_pageviews.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END as ty_page
FROM 
	website_sessions
		LEFT JOIN website_pageviews 
			ON website_sessions.website_session_id = website_pageviews.website_session_id 
WHERE
	 website_sessions.created_at < '2012-11-27'
GROUP BY 
	1, 2, 3
) made_ty
GROUP BY 
	1;

-- create conversion rate 
SELECT 
	MONTH(website_sessions.created_at) AS month,
	COUNT(DISTINCT conversion_table.website_session_id) AS sessions,
	COUNT(DISTINCT CASE WHEN conversion_table.TY = 1 THEN conversion_table.website_session_id ELSE NULL END) AS orders,
    COUNT(DISTINCT CASE WHEN conversion_table.TY = 1 THEN conversion_table.website_session_id ELSE NULL END)  / COUNT(DISTINCT website_sessions.website_session_id) AS conversion_rate
FROM 
	website_sessions
		INNER JOIN conversion_table 
			ON website_sessions.website_session_id = conversion_table.website_session_id 
GROUP BY 
	1;
```
<a href="https://lookerstudio.google.com/reporting/63dd1b31-841c-4c74-9710-fa1509f95223">
<img src="images/Q5.jpg?raw=true"/>

### 6. Comparing conversion rate between two landing pages for a set period of time. 
	
```SQL
SELECT
MIN(website_pageview_id) AS first_test_pv
FROM website_pageviews
WHERE pageview_url = '/lander-1';



-- for this step, we'll find the first pageview id 

CREATE TEMPORARY TABLE first_test_pageviews
SELECT
    website_pageviews.website_session_id, 
    MIN(website_pageviews.website_pageview_id) AS min_pageview_id
FROM website_pageviews 
	INNER JOIN website_sessions 
		ON website_sessions.website_session_id = website_pageviews.website_session_id
		AND website_sessions.created_at < '2012-07-28' -- prescribed by the assignment
		AND website_pageviews.website_pageview_id >= 23504 -- first page_view
        AND utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
GROUP BY 
	website_pageviews.website_session_id;

-- next, we'll bring in the landing page to each session, like last time, but restricting to home or lander-1 this time
CREATE TEMPORARY TABLE nonbrand_test_sessions_w_landing_pages
SELECT 
    first_test_pageviews.website_session_id, 
    website_pageviews.pageview_url AS landing_page
FROM first_test_pageviews
	LEFT JOIN website_pageviews 
		ON website_pageviews.website_pageview_id = first_test_pageviews.min_pageview_id
WHERE website_pageviews.pageview_url IN ('/home','/lander-1'); 

-- SELECT * FROM nonbrand_test_sessions_w_landing_pages;

-- then we make a table to bring in orders
CREATE TEMPORARY TABLE nonbrand_test_sessions_w_orders
SELECT
    nonbrand_test_sessions_w_landing_pages.website_session_id, 
    nonbrand_test_sessions_w_landing_pages.landing_page, 
    orders.order_id AS order_id

FROM nonbrand_test_sessions_w_landing_pages
LEFT JOIN orders 
	ON orders.website_session_id = nonbrand_test_sessions_w_landing_pages.website_session_id
;

SELECT * FROM nonbrand_test_sessions_w_orders;

-- to find the difference between conversion rates 
SELECT
    landing_page, 
    COUNT(DISTINCT website_session_id) AS sessions, 
    COUNT(DISTINCT order_id) AS orders,
    COUNT(DISTINCT order_id)/COUNT(DISTINCT website_session_id) AS conv_rate
FROM nonbrand_test_sessions_w_orders
GROUP BY 1; 
```
<a href="https://lookerstudio.google.com/reporting/9e891676-bb7f-4846-8346-c123f061a3db">
<img src="images/Q6.jpg?raw=true"/>

### 7. Full conversion funnel for the two landing pages analyzed in item 5. 
	
```SQL
SELECT
    website_sessions.website_session_id, 
    website_pageviews.pageview_url, 
    website_pageviews.created_at AS pageview_created_at, 
    CASE WHEN pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
    CASE WHEN pageview_url = '/lander-1' THEN 1 ELSE 0 END AS custom_lander,
    CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
    CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page, 
    CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
    CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
    CASE WHEN pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
    CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions 
	LEFT JOIN website_pageviews 
		ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.utm_source = 'gsearch' 
	AND website_sessions.utm_campaign = 'nonbrand' 
    AND website_sessions.created_at < '2012-07-28'
		AND website_sessions.created_at > '2012-06-19'
ORDER BY 
	website_sessions.website_session_id,
    website_pageviews.created_at;


CREATE TEMPORARY TABLE session_level_made_it_flagged
SELECT
	website_session_id, 
    MAX(homepage) AS saw_homepage, 
    MAX(custom_lander) AS saw_custom_lander,
    MAX(products_page) AS product_made_it, 
    MAX(mrfuzzy_page) AS mrfuzzy_made_it, 
    MAX(cart_page) AS cart_made_it,
    MAX(shipping_page) AS shipping_made_it,
    MAX(billing_page) AS billing_made_it,
    MAX(thankyou_page) AS thankyou_made_it
FROM(
SELECT
    website_sessions.website_session_id, 
    website_pageviews.pageview_url, 
    website_pageviews.created_at AS pageview_created_at, 
    CASE WHEN pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
    CASE WHEN pageview_url = '/lander-1' THEN 1 ELSE 0 END AS custom_lander,
    CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
    CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page, 
    CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
    CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
    CASE WHEN pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
    CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions 
	LEFT JOIN website_pageviews 
		ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.utm_source = 'gsearch' 
	AND website_sessions.utm_campaign = 'nonbrand' 
    AND website_sessions.created_at < '2012-07-28'
		AND website_sessions.created_at > '2012-06-19'
ORDER BY 
	website_sessions.website_session_id,
    website_pageviews.created_at
) AS pageview_level

GROUP BY 
	website_session_id
;

-- then this would produce the final output, part 1
SELECT
	CASE 
	WHEN saw_homepage = 1 THEN 'saw_homepage'
        WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
        ELSE 'uh oh... check logic' 
	END AS segment, 
    COUNT(DISTINCT website_session_id) AS sessions,
    COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS to_products,
    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS to_cart,
    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS to_billing,
    COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
FROM session_level_made_it_flagged 
GROUP BY 1
;


-- then this as final output part 2 - click rates

SELECT
	CASE 
	WHEN saw_homepage = 1 THEN 'saw_homepage'
        WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
        ELSE 'uh oh... check logic' 
	END AS segment, 
	COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT website_session_id) AS lander_click_rt,
    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS products_click_rt,
    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS mrfuzzy_click_rt,
    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS cart_click_rt,
    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS shipping_click_rt,
    COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS billing_click_rt
FROM session_level_made_it_flagged
GROUP BY 1;
```
<a href="https://lookerstudio.google.com/reporting/863ed3b7-3916-4bd0-bade-d8f4db1f9b21">
<img src="images/Q7.jpg?raw=true"/>

### 8. Revenue per billing page. Comparing two versions of the website's billing page. 
```SQL
CREATE TEMPORARY TABLE billing_pagezz
SELECT 
	website_sessions.website_session_id, 
    website_pageviews.pageview_url, 
    website_pageviews.created_at AS pageview_created_at,
    CASE WHEN website_pageviews.pageview_url = '/billing' THEN 1 ELSE 0 END as billing_page,
    CASE WHEN website_pageviews.pageview_url = '/billing-2' THEN 1 ELSE 0 END as billing2_page,
    CASE WHEN website_pageviews.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END as ty_page
FROM 
	website_sessions
		LEFT JOIN website_pageviews 
			ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE 
	website_sessions.created_at BETWEEN '2012-09-10' AND '2012-11-10'
    AND website_pageviews.pageview_url IN ('/billing', '/billing-2')
ORDER BY 
	1, 3;

SELECT 
	MONTH(billing_pagezz.pageview_created_at) AS month,
    billing_pagezz.pageview_url,
	COUNT(billing_pagezz.website_session_id) AS sessions,
    SUM(orders.price_usd) AS total_sales,
    SUM(orders.items_purchased) AS total_goods_sold
FROM 
	billing_pagezz
		LEFT JOIN orders
			ON billing_pagezz.website_session_id = orders.website_session_id
GROUP BY 
	1,2;
```
<a href="https://lookerstudio.google.com/reporting/80eeec39-cb33-4408-8951-e6b411af94d5">
	<img src="images/Q8.jpg?raw=true"/></a>

## Insights
<ul>
  <li>There is steady growth with the website sessions and orders from March to November.</li>
  <li>Nonbrand campaigns have brought more web visitors than the brand campaigns.</li>
  <li>We're getting more desktop users than mobile users visit our website.</li>
  <li>Gsearch paid sessions bring majority of our web traffic. We should invest more here.</li>
  <li>There's a huge gap between sessions and orders. Meaning we're not converting enough people from web visitor to customer.</li>
  <li>The lander-1 landing page has a higher conversion rate than the home page.</li>
  <li>There is a decline in the amount of web visitors that reach our billing pages from September to November. This also correlates to our sales.</li>
</ul>


For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
