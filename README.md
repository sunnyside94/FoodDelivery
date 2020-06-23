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

**NUMBER OF RESTAURANTS**  <br/>
SELECT COUNT(restaurant_id) FROM FoodDeliveryOrders <br/>

**NUMBER OF DRIVERS** <br/>
SELECT COUNT(driver_id) FROM FoodDeliveryOrders <br/>

**NUMBER OF CUSTOMERS** <br/>
SELECT COUNT(consumer_id) FROM FoodDeliveryOrders <br/>

**NUMBER OF RETURNING CUSTOMERS** <br/>
SELECT consumer_id, COUNT(*) AS Num_ORDERs FROM FoodDeliveryOrders <br/>
GROUP BY consumer_id <br/>
having Num_ORDERs > 1 <br/>

**TOP 5 RESTAURANTS WITH NUMBER OF ORDERS AND ORDER TOTALS** <br/>
SELECT COUNT(*) AS Num_of_ORDERs, SUM(ORDER_total) AS Total_ORDER, restaurant_id FROM FoodDeliveryOrders <br/>
GROUP BY restaurant_id <br/>
ORDER BY Num_of_ORDERs DESC, Total_ORDER DESC <br/>
LIMIT 5 <br/>

**TOP 5 AVERAGE ORDER TOTALS OF RESTAURANT** <br/>
SELECT  AVG(ORDER_total) AS AVG_ORDER_Total, restaurant_id FROM FoodDeliveryOrders <br/>
GROUP BY restaurant_id <br/>
ORDER BY AVG_ORDER_Total DESC <br/>
LIMIT 5 <br/>

**TOTAL REFUNDED AMOUNT** <br/>
SELECT  SUM(refunded_amount) FROM FoodDeliveryOrders <br/>
where refunded_amount > 0 <br/>

**TOP 5 RESTAURANTS WITH THE MOST REFUNDED AMOUNT** <br/>
SELECT  SUM(refunded_amount) AS total_refunded, restaurant_id FROM FoodDeliveryOrders <br/>
where refunded_amount > 0 <br/>
GROUP BY restaurant_id <br/>
ORDER BY total_refunded DESC <br/>
LIMIT 5 <br/>

**TOP 5 RESTAURANTS WITH RETURNING CUSTOMERS** <br/>
SELECT COUNT(consumer_id) AS Num_of_returning_cust, restaurant_id FROM (SELECT consumer_id, COUNT(*) AS Num_ORDERS, restaurant_id FROM FoodDeliveryOrderS <br/>
GROUP BY consumer_id, restaurant_id <br/>
having Num_ORDERS > 1) AS returning_at_restaurant <br/>
GROUP BY restaurant_id <br/>
ORDER BY Num_of_returning_cust DESC <br/>
LIMIT 5 <br/>

**TOP 5 DAYS OF THE MOST ORDERS** <br/>
SELECT COUNT(*) AS Num_Orders, Day FROM (SELECT SUBSTR(customer_placed_order_datetime, 1, 2) AS Day, *  FROM FoodDeliveryOrders) as Day_Order <br/>
GROUP BY Day <br/>
ORDER BY Num_Orders DESC <br/>
LIMIT 5 <br/>

**NUMBER OF ASAP ORDERS** <br/>
SELECT COUNT(*)Now_or_Later, is_asap from FoodDeliveryOrders <br/>
GROUP BY is_asap <br/>


**ELAPSED TIME BETWEEN CUSTOMER PLACING ORDER AND ORDER BEING DELIVERED WHERE ORDER IS ASAP** <br/>
SELECT  restaurant_id, driver_id, delivery_region, <br/>
CASE WHEN (DD - CD) + 31 = 1  <br/>
	 THEN TIME((STRFTIME('%s', TO24) + STRFTIME('%s', DT)), 'unixepoch') <br/>
          WHEN (DD - CD)  < 0 <br/>
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
