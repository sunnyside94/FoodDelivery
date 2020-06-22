# FoodDelivery
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
The DATETIME in the CSV only consists of the Day. 
DATETIME is manipulated in queries to fit what is needed
