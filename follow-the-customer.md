##  Follow-the-customer analysis + dashboard 🍴🍸🍕

**Project description:** 
<br> The objective will be to load the data, clean and format it, before exploring and analyzing it.
<br> Here is the purpose of the analysis:

- <b> Global analysis of the business</b>

- <b> Is there a flux of clients between the two establisments? </b>

___
### Data description 
These data have been anonymized for confidentiality reasons. 
However, the purpose of this analysis has been asked by a real business.
<br> The biggest challenge was to find a way to generate a key in order to create a unique ID to establish the clientele flux. 

Detailed tables description: 
<br> 7 excel tables (csv) gathering information such as: 
- Monthly transactions regarding the business activity by establishments (October til January),
- e-mail listings
  
I'm going to focus on these two establishments:
- "La Terrasse ensoleillée" which is a modern-restaurant,with family-friendly and welcoming atmosphere and;
- "L'épicumiam" which is a bar-restaurant with friendly atmosphere, with colleagues and friendly atmosphere, games, TV...
Those two establishments are located within 10km. 

___
### Creation of a unique file

We aggregated all the tables in order to crate a unique file called aout-jan_clean1.csv
```python
import pandas as pd
import numpy as np

#Creation of one dataframe by month
aout = pd.read_csv('/content/AOUT.csv', sep=';', header=1 )
septembre = pd.read_csv('/content/SEPTEMBRE.csv', sep=';', header=1 )
octobre = pd.read_csv('/content/OCTOBRE.csv', sep=';', header=1 )
novembre = pd.read_csv('/content/NOVEMBRE.csv', sep=';', header=1 )
decembre = pd.read_csv('/content/DECEMBRE.csv', sep=';', header=1 )
janvier = pd.read_csv('/content/JANVIER.csv', sep=';', header=1 )

#Concatenation of all the files into a unique file

global = pd.concat([aout,septembre,octobre,novembre,decembre,janvier], ignore_index=True)

#Check if concat is OK
global.info()

#Export of the global table as csv
global.to_csv('aout-dec.csv', index=False) 

#Preliminary cleaning step to replace "," into "." in the "montant" column
global["montant"] = global["montant"].replace("," , ".", regex=True)

#Export final table clean as csv
global_clean.to_csv('aout-janv_clean1.csv', index=False, sep=';') 
```


___
### Data cleaning and transformation - General 

#Display the 5 first rows + information about aout_jan 
```python
import pandas as pd
aout_jan = pd.read_csv("aout-dec_clean1.csv",header = 0) 

aout_jan.head(5)
aout_jan.shape #(431083 lines, 10 columns)
aout_jan.info()
```

#A few transformation were required in order to analyse the data; for example date columns were not in a datetime format,
it is required to convert data into datetime objects. 
```python
aout_jan["date_locale_de_transaction"] = pd.to_datetime(aout_jan["date_locale_de_transaction"])
aout_jan["date_expiration"] = pd.to_datetime(aout_jan["date_expiration"])
```
#Identification of null values and corrective actions
```python
#Verify null values
aout_jan.isnull().sum()

#Display lines with null vales in numero_carte column using isnull() and loc
aout_jan.loc[aout_jan["numéro_carte"].isnull()]

#Display lines with null vales in nom_commercial column and create an inter dataframe
using .loc 
inter = aout_jan.loc[aout_jan["nom_commercial"].isnull()]

#Display distinct nom_commercial name using unique()
aout_jan["nom_commercial"].unique()

#Display lines where nom_commercial = NaN using a filter function
#There were different names for the same establishmen. I've managed to display the incorreclty spelled names into a filter
and register then into a new variable called resultat

filtre = (aout_jan["nom_commercial"] != "La Terrasse ensoleillée") ....
resultat = aout_jan[filtre] #n = 97053 data were mispelled 

#Delete a spefic line where 60% of the data were missing using ILOC
aout_jan = aout_jan.drop(aout_jan.iloc[45441].name,axis = 0)

#Some nom_commercial items were missing but I was able to retreive the information using data from B column
but first I needed to list all the nunique values of B (where no value was missing). 

resultat["B"].unique() #n = 14 for 10 establishments, renaming is required

#I then used the list to complete the missing values in nom_commercial. 
aout_jan.loc[aout_jan["B"] == "la terrasse ensoleillée", ["nom_commercial"]] = "LA TERRASSE ENSOLEILLEE"
```
#Creation of a unique identifier (pk_full) for each client 
In accordance wth the client and the data available, we have tested further combination of data such as 
nom_commercial + time_category + elements of transactions + contract number. 
We also verfied that this identifer was unique.

#Creation of time_categorie (morning/lunch/afternoon/diner/evening) from datetime
```sql
SELECT
*,
EXTRACT(hour FROM date_locale_de_transaction) AS hour,
CONCAT(EXTRACT(DAY FROM date_locale_de_transaction), '-', EXTRACT(MONTH FROM date_locale_de_transaction)) AS jour_du_mois 
FROM `project`
```
```sql
WITH temp_table AS(SELECT
    *,
    CASE
        WHEN CAST(date_locale_de_transaction AS TIME) BETWEEN '03:00:00' AND '11:30:00' THEN 'Matin' #morning
        WHEN CAST(date_locale_de_transaction AS TIME) BETWEEN '11:30:00' AND '15:30:00' THEN 'Déjeuner' #lunch
        WHEN CAST(date_locale_de_transaction AS TIME) BETWEEN '15:30:00' AND '17:00:00' THEN 'Après_midi' #afternoon
        WHEN CAST(date_locale_de_transaction AS TIME) BETWEEN '17:00:00' AND '20:00:00' THEN 'Dîner' #diner
        WHEN CAST(date_locale_de_transaction AS TIME) BETWEEN '19:00:00' AND '22:00:00' THEN 'Dîner' #diner
        WHEN CAST(date_locale_de_transaction AS TIME) > "22:00:00" THEN 'Soirée' #evening
        WHEN CAST(date_locale_de_transaction AS TIME) < "03:00:00" THEN 'Soirée' #evening
    END AS time_categorie
FROM `project`
WHERE
  type = "DEBIT" AND
  (nom_commercial LIKE "LA TERRASSE ENSOLEILLEE" OR
  nom_commercial LIKE "L'EPICUMIAM")
)
#This part is intented to verify that all data have been casted
SELECT 
  *, 
FROM 
  temp_table
WHERE
  time_categorie IS NULL
```
#Transactions ranking of client ordered by date (=ordre_visite)
```sql
SELECT
*,
ROW_NUMBER() OVER(PARTITION BY id ORDER BY date_locale_de_transaction) AS ordre_visite
FROM 'project'
```

___
### Data cleaning and transformation - Focus on the flux 

To verify that a flux was existing between each establishement, we calculated a recurrence rate. 
We estimated that with an overall reccurence rate above 20%, we could suggest the existence of a clientele flux.

```sql
SELECT
count(id) as total_clients,
count(distinct id) as unique_clients,
Round((count(distinct id) / count(id)),2) as recurrence_rate,
FROM `project`
```
Results : reccurence_rate = 0.43; we identified 43% of reccurent clients all establishments combined. 
<br> NB: the reccurence rate between "La Terrasse ensoleillée" and "L'épicumiam" was 0.6. 

Once all the general cleaning was done, python was used to generate the flux


```python
#Import table using pandas and generate table test

# Order data by ascending order, date and visit_ranking(= ordre_visite)
test_1 = test.sort_values(["date_locale_de_transaction", "ordre_visite"],ascending = True)

#Creation of the flux and replace null values by 0
test_2 = (test_1.set_index(["id",test_2.groupby("id").cumcount()])["pk_full"].unstack(fill_value ="0").add_prefix("pk_full").reset_index())

#Check if column 1 only contains establishments visited in first
test_2["pk_full0"].value_counts()

#Extract only the first 5 visits representing the average number of visits per customer
test_final = test_2[["id", "pk_full0", "pk_full1", "pk_full2", "pk_full3", "pk_full4"]]

#Rename columns
nom_colonnes = ["id","visite_1","visite_2","visite_3","visite_4","visite_5"]
test_final.rename(columns =dict(zip(test_final.columns, nom_colonnes)), inplace = True)

#Table export 
test_final.to_csv("flux_table.csv", sep =",", header = 1)
```
Here is a visual of the csv: 
<img src="images/flux.png?raw=true"/> 


___
### Dashboard

Here is the dashboard created with the data. 

[Follow-the-customer-dashboard](/[airbnb_dataset_analysis_project.md](https://public.tableau.com/views/follow-the-customer-project/Dashboard_global?:language=fr-FR&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link))

<img src="images/dashboard_1.png?raw=true"/> 

<img src="images/dashboard_2.png?raw=true"/> 

___ 
##  Some recommendations
- The business should start a reservation system to better follow clients between their establishments
- Since there is an actual flux between the establishments, it could be interesting to set a fidelity program since clients seem to comeback at least 2 times over 4 months.  
