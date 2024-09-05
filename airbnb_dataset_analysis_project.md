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
<br>*Box graph:* 
```python
import plotlyexpress as px
px.box(calendar,
       x="date",
       title = "Date insights")
```
<img src="images/boxplot.png?raw=true"/>
Data collection started on Sep 9th, 2022 and ended on Jan 31, 2023. 


<b>*Number of accomodations:*</b>
```python
calendar["listing_id"].nunique()
```
There was 500 accomodations available during this period of data collection. 


<b> *A few information about the period:*</b>
```python
calendar["price"].describe()
```
<img src="images/calendar_statistics.png?raw=true"/>
During this data collection period, 72100 reservations have been made. The mean price was 158.00 euros per night and the lowest 
price was 30 euros. It also shows that the highest price per night was over 4000 euros. It would be interesting to check if this is a
legit price or not. 

What is the average availability of an accommodation during this period? 
<br><i>Data type available is booleen, we choose t for True == available accomodation</i>
```python
total_availability = calendar["available"].count()
available_days = calendar[calendar["available"] == "t"].count()
average_availability = (available_days / total_availability).mean()*100
```
An accommodation was available 24 days during this period. 


<b> *Number of accomodations by room_type:* </b>
<br> According to listing table, there was 3 types of room available: 
- Entire home/appartement
- Private room
- Shared room
<i> What is the repartition of accomodation by room-type?</i>
<br> Beforehand it was required to join listing and calendar tables called reservation.
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
<b>Price difference between the type of rent</b>
<br><i> Is the average price difference between "entire home" and "private room" properties significant?</i>
We want to compare one continuous value (review score) vs one categorical value (room_type) and n>30 (n=500), in order to show a difference between those two variables we can do a z-test
<br>But first, we need to create two dataframes home and private_room so we can perform the z-test

```python
home = listing[listing["room_type"]== "Entire home/apt"] #only select entire home/apt
private_room = listing[listing["room_type"] == "Private room"] #only select private_room
```
Parameters: 
<br><b><i>Hypothesis Ho: "There is no difference of review score bewteen renting a home vs renting a private-room"</i></b>
<br><b><i>Hypothesis H1: "There is a difference of review score bewteen renting a home vs renting a private-room"</i></b>
<br><b><i>a = 5%</i></b>
<br>Then we perfom the z-test: 
```python
import numpy as np
from statsmodels.stats.weightstats import ztest
z_score, p_value = ztest(home["review_scores_value"], private_room["review_scores_value"])
```
p_value = 0.047917748274801955

With a = 5% and p_value = 0.0479, p_value < a :
<br> We reject Ho and we accept H1, which means it exists a statistical difference between the review score and the type of accommodation .


<b>Does a host's response time affect its final review score?</b>
<br>Here is an overview of the accomodation proportion with long response time catgorized as "a few days or more". This way we should have a quick understanding of how reactive are the hosts. 
```python
total_host_responsive = listing["host_response_time"].count()
long_response_time_host = listing[listing["host_response_time"] == "a few days or more"].count()
long_response_host = (long_response_time_host/total_host_responsive).mean()*100
```
The proportion of accommodations with a long responsive host is 4.3999999999999995 %. Overall hosts take less than a few days to answer to their clients. 

<br> In order to assess the correlation between the host response time and the review scores value, we should transform the canonical values into quantitative ones, according to the description below:

*   1 : within an hour
*   2 : within a few hours
*   3 : within a day
*   4 : a few days or more
<br> To do so, I've created a fucntion to automate the process.
```python
def replace_value(value):
   if value == "within an hour" :
    return 1
   elif value == "within a few hours":
    return 2
   elif value == "within a day":
    return 3
   elif value == "a few days or more":
    return 4
   else:
    return "issue"
```
```python
#Apply the replace_value function on host_response_time_num column to modify and transform the values
listing["host_response_time_num"] = listing["host_response_time"].apply(replace_value)

#Verify modification is OK +Check repartition of host_response_time_num
listing["host_response_time_num"].value_counts()
```
<img src="images/classification.png?raw=true"/>

To assess the correlation etween the host response time and the review scores value, we'll determine the correlation score using Pearson's method.
```python
corr_host_review = listing["host_response_time_num"].corr(listing["review_scores_value"],method = "pearson")
```
Pearson's score = -0.18039981452312104
<br>  We can show that there is a little negative correlation between the response time and the final score. 

```python
#Boxplot to showcase review_score_value by host_response_time_num
px.box (x=listing["host_response_time_num"],
        y = listing["review_scores_value"],
        color = listing["host_response_time_num"],
        labels = {"x" : "host response time", "y" : "review scores values"},
        title = "Score value by host response time")
```
<img src="images/score_evolution.png?raw=true"/>
According to the Pearson's coefficient (r = -0,18) and the graph above, the most a host takes time to answer, the smallest the review score will be. There is no difference if hosts respond within an hour or day. 


<b>- Price trend analysis</b>
<br><i> Are hosts taking advantage of the calendar to set their prices?</i>

To analyse the price trend over time, I need to create a dataframe from the calendar dataframe and set the date column as index. 
```python
calendar.set_index(calendar["date"], inplace = False)
```
<img src="images/calendar_date_index.png?raw=true"/>

```python
#Calculate the average price per day
mean_price = calendar.resample("D", on= "date")["price"].mean()
mean_price
```
```python
#Display the evolution of the average daily price
px.line(mean_price,
        title = "Evolution of the average daily price")
```
<img src="images/price_trend.png?raw=true"/>

From Sep 2022 until 21-22 December 2022, before the Christmas holidays, we can see the average daily price is 157 euros.
<br>Starting 22 Dec, there's an increase of the price peaking at 170euros on 23-24-25 December.
<br>Then we observe a huge decrease of the average price reaching 162 euros.
<br>We observe a second significant increase of the average daily price on the last 3 days of December reaching out its peak on 31 Dec2022, with a value over 170 euros.
<br>Then, we observe a huge decrease of the average price on the 1st January 2023 returning to its average value observed before Christmas.
<br> In other words, hosts seem to increase their prices during the holiday season (+7,6% of the avergae price)

###What is the percentage of hosts that never change their home price?
```python
#Calculate the percentage of properties that never change their home price
total_change = calendar.groupby("listing_id")["price"].std()
never_change = total_change[total_change == 0].count()
pct_never_change = (never_change/len(total_change))*100 #we're using len so we can select the null values
```
47.0% of the hosts never change their price over time. In other words, 53% of hosts adjust their prices and take advantage of the calendar to set their rates.  
