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
**NUMBER OF ORDERS PER DELIVERY REGION**
SELECT COUNT(*), delivery_region FROM FoodDeliveryOrders
GROUP BY delivery_region

**NUMBER OF RESTAURANTS**
SELECT COUNT(restaurant_id) FROM FoodDeliveryOrders

**NUMBER OF DRIVERS**
SELECT COUNT(driver_id) FROM FoodDeliveryOrders

**NUMBER OF CUSTOMERS**
SELECT COUNT(consumer_id) FROM FoodDeliveryOrders

**NUMBER OF RETURNING CUSTOMERS**
SELECT consumer_id, COUNT(*) AS Num_ORDERs FROM FoodDeliveryOrders
GROUP BY consumer_id
having Num_ORDERs > 1

**TOP 5 RESTAURANTS WITH NUMBER OF ORDERS AND ORDER TOTALS**
SELECT COUNT(*) AS Num_of_ORDERs, SUM(ORDER_total) AS Total_ORDER, restaurant_id FROM FoodDeliveryOrders
GROUP BY restaurant_id
ORDER BY Num_of_ORDERs DESC, Total_ORDER DESC
LIMIT 5

**TOP 5 AVERAGE ORDER TOTALS OF RESTAURANT**
SELECT  AVG(ORDER_total) AS AVG_ORDER_Total, restaurant_id FROM FoodDeliveryOrders
GROUP BY restaurant_id
ORDER BY AVG_ORDER_Total DESC
LIMIT 5

**TOTAL REFUNDED AMOUNT**
SELECT  SUM(refunded_amount) FROM FoodDeliveryOrders
where refunded_amount > 0

**TOP 5 RESTAURANTS WITH THE MOST REFUNDED AMOUNT**
SELECT  SUM(refunded_amount) AS total_refunded, restaurant_id FROM FoodDeliveryOrders
where refunded_amount > 0
GROUP BY restaurant_id
ORDER BY total_refunded DESC
LIMIT 5

**TOP 5 RESTAURANTS WITH RETURNING CUSTOMERS**
SELECT COUNT(consumer_id) AS Num_of_returning_cust, restaurant_id FROM (SELECT consumer_id, COUNT(*) AS Num_ORDERS, restaurant_id FROM FoodDeliveryOrderS
GROUP BY consumer_id, restaurant_id
having Num_ORDERS > 1) AS returning_at_restaurant
GROUP BY restaurant_id
ORDER BY Num_of_returning_cust DESC
LIMIT 5

**TOP 5 DAYS OF THE MOST ORDERS**
SELECT COUNT(*) AS Num_Orders, Day FROM (SELECT SUBSTR(customer_placed_order_datetime, 1, 2) AS Day, *  FROM FoodDeliveryOrders) as Day_Order
GROUP BY Day
ORDER BY Num_Orders DESC
LIMIT 5

**NUMBER OF ASAP ORDERS**
SELECT COUNT(*)Now_or_Later, is_asap from FoodDeliveryOrders
GROUP BY is_asap


**ELAPSED TIME BETWEEN CUSTOMER PLACING ORDER AND ORDER BEING DELIVERED WHERE ORDER IS ASAP**
SELECT  restaurant_id, driver_id, delivery_region,
CASE WHEN (DD - CD) + 31 = 1 
	 THEN TIME((STRFTIME('%s', TO24) + STRFTIME('%s', DT)), 'unixepoch')
          WHEN (DD - CD)  < 0
     	 THEN DAYH || SUBSTR(SAMEDAY, -6)
	 WHEN (DD - CD) = 1 
      	 THEN  TIME((STRFTIME('%s', TO24) + STRFTIME('%s', DT)), 'unixepoch')
     	 WHEN (DD - CD) > 1
     	 THEN  FEWDAYS || SUBSTR(SAMEDAY, -6)
     	 WHEN (DD - CD) = 0
     	 THEN SAMEDAY
	 END AS ELAPSED_TIME
		FROM (SELECT  restaurant_id, driver_id, delivery_region, SUBSTR(customer_placed_order_datetime, 1, 2) AS CD, 
      		TIME((STRFTIME('%s', "24:00:00") - STRFTIME('%s', TIME(SUBSTR(customer_placed_order_datetime, -8)))), 		'unixepoch') AS TO24,
	  	SUBSTR(delivered_to_consumer_datetime, 1, 2) AS DD, 
      		TIME(SUBSTR(delivered_to_consumer_datetime, -8)) AS DT,
      		TIME(SUBSTR(customer_placed_order_datetime, -8)) AS CT,
      		TIME((STRFTIME('%s', TIME(SUBSTR(delivered_to_consumer_datetime, -8))) 
            	- STRFTIME('%s', TIME(SUBSTR(customer_placed_order_datetime, -8)))), 'unixepoch') AS SAMEDAY, 
      		SUBSTR(TIME((STRFTIME('%s', TIME((STRFTIME('%s', "24:00:00") - STRFTIME('%s', 		TIME(SUBSTR(customer_placed_order_datetime, -8)))), 'unixepoch')) 
            	+ STRFTIME('%s', TIME(SUBSTR(delivered_to_consumer_datetime, -8)))), 'unixepoch'), 1, 2) 
      	    	+ (((SUBSTR(delivered_to_consumer_datetime, 1, 2)) - (SUBSTR(customer_placed_order_datetime, 1, 2))) * 24) AS 		FEWDAYS, 
	  	(((SUBSTR(delivered_to_consumer_datetime, 1, 2) - SUBSTR(customer_placed_order_datetime, 1, 2)) + 31 - 1) * 		24)
      		+ SUBSTR(TIME((STRFTIME('%s', TIME(SUBSTR(delivered_to_consumer_datetime, -8))) 
            	- STRFTIME('%s', TIME(SUBSTR(customer_placed_order_datetime, -8)))), 'unixepoch'), 1, 2) AS DAYH
		FROM FoodDeliveryOrders
		WHERE is_asap LIKE "TRUE") AS TIMES
ORDER BY ELAPSED_TIME DESC
