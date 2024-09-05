##  AirBnb dataset analysis 🏡

**Project description:** 
<br> The objective will be to load the data, clean and format it, before exploring and analyzing it.</br>
<br> Here is the purpose of the analysis:</br>

<b>- Price difference between the type of rent</b>
<br><i> Is the average price difference between "entire home" and "private room" properties significant?</i>

<b>- Relationship between host's response time and final review score</b>
<br><i> Does a host's response time affect its final review score?</i>

<b>- Price trend analysis</b>
<br><i> Are hosts taking advantage of the calendar to set their prices?</i>

___
### Data description 

Detailed tables description: 
<br><b> calendar</b> : Table gathering reservation calendar information. Contains prices and availability for coming year.
- *listing_id*: id of the accommodation
- *date*: day of possible reservation
- *available*: availability/unavailability of the accommodation
- *price*: price according to the day

<br><b> listing</b> : Table with an overview of the accomodations.</br>
- *id* : Airbnb's unique identifier for the listing (accomodation)
- *host_response_time* : The average response time of the host for this accommodation
- *room_type* : Entire home/apt | Private room | Shared room
- *review_scores_value* : The average reviews score that the listing has

___
### Import of data and preview of 10 first rows

```python
import pandas as pd
calendar = pd.read_csv("calendar.csv")
calendar.head(10))
```
<img src="images/calendar.png?raw=true"/> 

```python
import pandas as pd
calendar = pd.read_csv("listing.csv")
listing.head(10))
```
<img src="images/listing.png?raw=true"/>

___
### Data transformation 
A few transformation were required in order to analyse the data; for example the date column was not in a datetime format,
it is required to convert the data into a datetime object. I've also extracted to month into a specific column 

```python
calendar["date"] = pd.to_datetime(calendar["date"])
```
<img src="images/date_datetime.png?raw=true"/>

```python
calendar["month"] = calendar["date"].dt.month
```
The price column was not in the correct format. To do so, I've replaced the "$" with "" and the "," with ".". 
To do so, I've created a fucntion to automate the process. This column was not in the correct format, it was
required to convert the data type as a float 

```python
def clean_price(price):
  if "$" in price:
    result = price.replace("$","")
    result = result.replace(",","")
    return float(result)
```

```python
#Verify if clean_price works
price = "$2300.00"
clean_price(price)
```
<img src="images/price.png?raw=true"/>

```python
#Apply the clean_price function on price column to modify and transform the dtype
calendar["price"] = calendar["price"].apply(clean_price)
```
Here is the final result: 
```python
calendar.head(5)
```
<img src="images/calendar_finish.png?raw=true"/>

___
### Data exploration 
In this part, I will display graphs to set the context of the analysis 
** Box graph** : 
```python
import plotlyexpress as px
px.box(calendar,
       x="date",
       title = "Date insights")
```
<img src="images/boxplot.png?raw=true"/>
Data collection started on Sep 9th, 2022 and ended on Jan 31, 2023. 

** Number of accomodations** : 
```python
calendar["listing_id"].nunique()
```
There was 500 accomodations available during this period of data collection. 

** A few information about the period** : 
```python
calendar["price"].describe()
```
<img src="images/calendar_statistics.png?raw=true"/>
During this data collection period, 72100 reservations have been made. The mean price was 158.00 euros per night and the lowest 
price was 30 euros. It also shows that the highest price per night was over 4000 euros. It would be interesting to check if this is a
legit price or not. 

What is the average availability of an accommodation during this period? 
<i>Data type available is booleen, we choose t for True == available accomodation</i>
```python
total_availability = calendar["available"].count()
available_days = calendar[calendar["available"] == "t"].count()
average_availability = (available_days / total_availability).mean()*100
```
An accommodation was available 24 days during this period. 

** Number of accomodations by room_type** : 
According to listing table, there was 3 types of room available: 
- Entire home/appartement
- Private room
- Shared room
<i> What is the repartition of accomodation by room-type?</i> Beforehard it was required to join listing and calendar tables called reservation.
```python
reservation = pd.merge(calendar, listing, left_on = ["listing_id"], right_on =["id"])
```
```python
acc_room_type = reservation.groupby("room_type")["room_type"].count()
acc_room_type

px.histogram(x = acc_room_type.index,
             y= acc_room_type.values,
             color = acc_room_type.index,
             labels = {"x" : "Room_type", "y" : "Total number of accomodation"},
             title = "Total accomodations by room_type")
```
<img src="images/accomodation_roomtype.png?raw=true"/>
Between the Sep 9, 2022 and Jan 31, 2023 almost 90% of the accomodations available on the market were entire home/appartements. 

___
### Data analysis 
<b>- Price difference between the type of rent</b>
<br><i> Is the average price difference between "entire home" and "private room" properties significant?</i>

 

