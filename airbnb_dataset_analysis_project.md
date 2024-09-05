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


Just like this. And you can even add internal coding blocks

```python
print('this is the python code I used to solve this problem')
```

### 2. You can add any images you'd like. 

<img src="images/dummy_thumbnail.jpg?raw=true"/>


