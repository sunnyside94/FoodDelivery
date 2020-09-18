# **Food Delivery**
Analyze a CSV file of on-demand prepared food deliveries. 
This data resembles businesses like GrubHub, DoorDash, UberEats etc

# Data
**Customer_placed_order_datetime** = Day and time customer placed order (Ex: 09 12:34:57) <br/>
**Placed_order_with_restaurant_datetime** = Day and time restaurant received order (Ex: 09 12:45:51) <br/>
**Driver_at_restaurant_datetime** = Day and time driver arrived to restuarant (Ex: 09 1:02:12) <br/>
**Delivered_to_consumer_datetime** = Day and time food was delivered to customer (Ex: 09 1:34:19) <br/>
**Restaurant_ID** = ID of restaurant <br/>
**Driver_ID** = ID of the person delivering the food <br/>
**Consumer_ID** = Customer ID <br/>
**Delivery_Region** = Area food was delivered too <br/>
**Is_ASAP** = True if the order is meant to be delivered immediately FALSE if at another time <br/>
**Order_Total** = Total of food order <br/>
**Amount_of_Discount** = Total discount of order <br/>
**Amount_of_Tip** = How much the customer tipped the driver <br/>
**Refund_amount** = How much the customer was refunded <br/>

# Issues with DATETIME
The DATETIME in the CSV only consists of the Day. <br/>
DATETIME is manipulated in queries to fit what is needed

# SQLite Queries
**NUMBER OF ORDERS PER DELIVERY REGION**  <br/>
SELECT COUNT(*), delivery_region FROM FoodDeliveryOrders  <br/>
GROUP BY delivery_region <br/>
<br/>
3760  Mountain View <br/>
26    NONE <br/>
11434 Palo Alto <br/>
2859  San Jose <br/>

**NUMBER OF RESTAURANTS**  <br/>
SELECT count(restaurant_id) FROM (SELECT DISTINCT restaurant_id from FoodDeliveryOrders) as restaurants <br/>
314 restuarants<br/>

**NUMBER OF DRIVERS** <br/>
SELECT COUNT(driver_id) FROM (SELECT DISTINCT driver_id from FoodDeliveryOrders) as drivers<br/>
297 distinct drivers<br/>

**NUMBER OF ORDERS** <br/>
SELECT COUNT(consumer_id) FROM FoodDeliveryOrders <br/>
18079 total orders<br/>

**LIST OF RETURNING CUSTOMERS AND NUMBER OF ORDERS** <br/>
SELECT consumer_id, COUNT(*) AS Num_ORDERs FROM FoodDeliveryOrders <br/>
GROUP BY consumer_id <br/>
having Num_ORDERs > 1 <br/>
ORDER BY Num_ORDERs DESC <br/>
<br/>
consumer id: 514	Number of Orders: 66 <br/>
consumer id: 929	Number of Orders: 50 <br/>
consumer id: 11956	Number of Orders: 47 <br/>
consumer id: 2469	Number of Orders: 43 <br/>

**TOP 5 RESTAURANTS WITH NUMBER OF ORDERS AND ORDER TOTALS** <br/>
SELECT COUNT(*) AS Num_of_ORDERs, SUM(ORDER_total) AS Total_ORDER, restaurant_id FROM FoodDeliveryOrders <br/>
GROUP BY restaurant_id <br/>
ORDER BY Num_of_ORDERs DESC, Total_ORDER DESC <br/>
LIMIT 5 <br/>
<br/>
Number of Orders: 747	Total: 38386.19	restuarant ID: 8<br/>
Number of Orders: 724	Total: 32852.18	restuarant ID: 20<br/>
Number of Orders: 717	Total: 39374.93	restuarant ID: 9<br/>
Number of Orders: 494	Total: 26310.39	1restuarant ID: 07<br/>
Number of Orders: 472	Total: 15421.88	restuarant ID: 12<br/>

**TOP 5 AVERAGE ORDER TOTALS OF RESTAURANT** <br/>
SELECT  AVG(ORDER_total) AS AVG_ORDER_Total, restaurant_id FROM FoodDeliveryOrders <br/>
GROUP BY restaurant_id <br/>
ORDER BY AVG_ORDER_Total DESC <br/>
LIMIT 5 <br/>
<br/>
Average order total: 501.24		restaurant ID: 11 <br/>
Average order total: 106.49		restaurant ID: 356 <br/>
Average order total: 99.75		restaurant ID: 257 <br/>
Average order total: 98.33		restaurant ID: 390 <br/>
Average order total: 98.17		restaurant ID: 378 <br/>
	

**TOTAL REFUNDED AMOUNT** <br/>
SELECT  SUM(refunded_amount) FROM FoodDeliveryOrders <br/>
where refunded_amount > 0 <br/>
Total Refunded: 11065.39 <br/>

**TOP 5 RESTAURANTS WITH THE MOST REFUNDED AMOUNT** <br/>
SELECT  SUM(refunded_amount) AS total_refunded, restaurant_id FROM FoodDeliveryOrders <br/>
where refunded_amount > 0 <br/>
GROUP BY restaurant_id <br/>
ORDER BY total_refunded DESC <br/>
LIMIT 5 <br/>
<br/>
Total Refunded: 822.27		Restaurant ID: 20 <br/>
Total Refunded: 757.88		Restaurant ID: 63 <br/>
Total Refunded: 533.81		Restaurant ID: 107 <br/>
Total Refunded: 418.12		Restaurant ID: 8 <br/>
Total Refunded: 326.33		Restaurant ID: 9 <br/>

**TOP 5 RESTAURANTS WITH RETURNING CUSTOMERS** <br/>
SELECT COUNT(consumer_id) AS Num_of_returning_cust, restaurant_id FROM (SELECT consumer_id, COUNT(*) AS Num_ORDERS, restaurant_id FROM FoodDeliveryOrderS <br/>
GROUP BY consumer_id, restaurant_id <br/>
having Num_ORDERS > 1) AS returning_at_restaurant <br/>
GROUP BY restaurant_id <br/>
ORDER BY Num_of_returning_cust DESC <br/>
LIMIT 5 <br/>
<br/>
Number of Returning Customers: 153	Restaurant ID: 8 <br/>
Number of Returning Customers: 137	Restaurant ID: 9 <br/>
Number of Returning Customers: 128	Restaurant ID: 20 <br/>
Number of Returning Customers: 83	Restaurant ID: 107 <br/>
Number of Returning Customers: 82	Restaurant ID: 12 <br/>


**TOP 5 DAYS OF THE MOST ORDERS** <br/>
SELECT COUNT(*) AS Num_Orders, Day FROM (SELECT SUBSTR(customer_placed_order_datetime, 1, 2) AS Day, *  FROM FoodDeliveryOrders) as Day_Order <br/>
GROUP BY Day <br/>
ORDER BY Num_Orders DESC <br/>
LIMIT 5 <br/>
<br/>
Orders: 722	Day: 25 <br/>
Orders: 720	Day: 26 <br/>
Orders: 700	Day: 27 <br/>
Orders: 687	Day: 12 <br/>
Orders: 645	Day: 23 <br/>

**NUMBER OF ASAP ORDERS** <br/>
SELECT COUNT(*)Now_or_Later, is_asap from FoodDeliveryOrders <br/>
GROUP BY is_asap <br/>
<br/>
NOT ASAP Orders: 3643	False <br/>
ASAP Orders:     14436	True <br/>


**ELAPSED TIME BETWEEN CUSTOMER PLACING ORDER AND ORDER BEING DELIVERED WHERE ORDER IS ASAP** <br/>
SELECT  restaurant_id, driver_id, delivery_region, <br/>
CASE WHEN (DD - CD) + 31 = 1  <br/>
	 THEN TIME((STRFTIME('%s', TO24) + STRFTIME('%s', DT)), 'unixepoch') <br/>
         WHEN (DD - CD)  < 0 <br/>e
     	 THEN DAYH || SUBSTR(SAMEDAY, -6) <br/>
	 WHEN (DD - CD) = 1  <br/>
      	 THEN  TIME((STRFTIME('%s', TO24) + STRFTIME('%s', DT)), 'unixepoch') <br/>
     	 WHEN (DD - CD) > 1 <br/>
     	 THEN  FEWDAYS || SUBSTR(SAMEDAY, -6) <br/>
     	 WHEN (DD - CD) = 0 <br/>
     	 THEN SAMEDAY <br/>
	 END AS ELAPSED_TIME <br/>
		FROM (SELECT  restaurant_id, driver_id, delivery_region, SUBSTR(customer_placed_order_datetime, 1, 2) AS CD,  <br/>
      		TIME((STRFTIME('%s', "24:00:00") - STRFTIME('%s', TIME(SUBSTR(customer_placed_order_datetime, -8)))), 'unixepoch') AS TO24, <br/>
	  	SUBSTR(delivered_to_consumer_datetime, 1, 2) AS DD,  <br/>
      		TIME(SUBSTR(delivered_to_consumer_datetime, -8)) AS DT, <br/>
      		TIME(SUBSTR(customer_placed_order_datetime, -8)) AS CT, <br/>
      		TIME((STRFTIME('%s', TIME(SUBSTR(delivered_to_consumer_datetime, -8)))  <br/>
            	- STRFTIME('%s', TIME(SUBSTR(customer_placed_order_datetime, -8)))), 'unixepoch') AS SAMEDAY,  <br/>
      		SUBSTR(TIME((STRFTIME('%s', TIME((STRFTIME('%s', "24:00:00") - STRFTIME('%s', TIME(SUBSTR(customer_placed_order_datetime, -8)))), 'unixepoch'))  <br/>
            	+ STRFTIME('%s', TIME(SUBSTR(delivered_to_consumer_datetime, -8)))), 'unixepoch'), 1, 2)  <br/>
      	    	+ (((SUBSTR(delivered_to_consumer_datetime, 1, 2)) - (SUBSTR(customer_placed_order_datetime, 1, 2))) * 24) AS FEWDAYS,  <br/>
	  	(((SUBSTR(delivered_to_consumer_datetime, 1, 2) - SUBSTR(customer_placed_order_datetime, 1, 2)) + 31 - 1) * 24) <br/>
      		+ SUBSTR(TIME((STRFTIME('%s', TIME(SUBSTR(delivered_to_consumer_datetime, -8)))  <br/>
            	- STRFTIME('%s', TIME(SUBSTR(customer_placed_order_datetime, -8)))), 'unixepoch'), 1, 2) AS DAYH <br/>
		FROM FoodDeliveryOrders <br/>
		WHERE is_asap LIKE "TRUE") AS TIMES <br/>
ORDER BY ELAPSED_TIME DESC
