https://github.com/cgatama/SpaceX-Falcon-9-1st-stage-Success-Landing-Prediction
# How to connect to Data Sources
1. Direct RestAPI
```python
import requests

url = "https://api.spacexdata.com/v4/launches/past"
response = requests.get(url)   # Send GET request to API
data = response.json()         # Parse response into Python dict/list
data = pd.json_normalize(response.json()) # DATA WRANGLING, Ensure to normalize the data into data frame 

print(data[0])   # Example: print first launch record
```
However, for API endpoint connections, we must connect to the right points.
The **for loops** are used here because the SpaceX launch data JSON only contains IDs, not the descriptive names or details.

Example: in the launch dataset you get:

| Field     | Value                                   |
|-----------|-----------------------------------------|
| rocket    | 5e9d0d95eda69973a809d1ec                |
| launchpad | 5e9e4502f5090995de566f86                |
| payloads  | ["5eb0e4b5b6c3bb0006eeb1e1"]            |


```python
# Takes the dataset and uses the launchpad column to call the API and append the data to the list
def getLaunchSite(data):
    for x in data['launchpad']:
       if x:
         response = requests.get("https://api.spacexdata.com/v4/launchpads/"+str(x)).json()
         Longitude.append(response['longitude'])
         Latitude.append(response['latitude'])
         LaunchSite.append(response['name'])
```

These are just foreign keys (like database IDs).

To make the table human-readable, you have to look up each ID by calling the appropriate API endpoint:

/v4/rockets/<id> → gives you rocket name

/v4/launchpads/<id> → gives you site name, lat, lon

/v4/payloads/<id> → gives you mass, orbit

The status code is the HTTP response code returned by the server after your GET request.

```python
response.status_code #returns the request status.
```

✅ 200 means OK → the request was successful, and the server returned the JSON data correctly.

Other common codes you might see:

| Status Code | Meaning                          |
|-------------|----------------------------------|
| 404         | Not Found (URL is wrong or resource doesn’t exist) |
| 401 / 403   | Unauthorized / Forbidden (need credentials or access denied) |
| 500         | Internal Server Error (problem on the server side) |

2. Webscrapping

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

url = "https://en.wikipedia.org/wiki/List_of_Falcon_9_and_Falcon_Heavy_launches"
response = requests.get(url)
soup = BeautifulSoup(response.text, "html.parser")

# Find the first table
table = soup.find("table", {"class": "wikitable"})
df = pd.read_html(str(table))[0]   # Convert HTML table → DataFrame

print(df.head())
```
Parsing the Data and Describing Its Features

1. Dataframe
```python
# Setting this option will print all collumns of a dataframe
pd.set_option('display.max_columns', None)
# Setting this option will print all of the data in a feature
pd.set_option('display.max_colwidth', None)
```

2. Summarizing
```python
df = pd.DataFrame() #Convert into dataframe first
df.describe() #describes statistical summary
df.info() #indicates data type
```

3. Filtering

```python
data[data['BoosterVersion'] != 'Falcon 1']
````

this is called **boolean indexing**. It works in two steps:

  1. **Condition check**

   ```python
   data['BoosterVersion'] != 'Falcon 1'
   ```

   This compares each row in the `BoosterVersion` column against `"Falcon 1"`.
   It returns a Series of `True`/`False` values:

   ```
   0     True
   1     True
   2    False
   3     True
   Name: BoosterVersion, dtype: bool
   ```

  2. **Filtering with `data[ ... ]`**
   By passing this boolean Series back into the DataFrame:

   ```python
   data[ data['BoosterVersion'] != 'Falcon 1' ]
   ```

   Pandas **keeps only the rows where the condition is `True`**.
   In this example, it removes rows where `BoosterVersion` equals `"Falcon 1"`.

## 1. Counting with a For Loop

Sometimes you may want to count occurrences without Pandas.  
You can do this with a simple dictionary and a `for` loop.

```python
# Example list of mission outcomes
outcomes = ["True ASDS", "False ASDS", "True RTLS", "True ASDS", "False RTLS"]

# Initialize an empty dictionary
count_dict = {}

# Loop through the outcomes
for outcome in outcomes:
    if outcome in count_dict:
        count_dict[outcome] += 1
    else:
        count_dict[outcome] = 1

print(count_dict)
import pandas as pd

# Example DataFrame
data = {"Outcome": ["True ASDS", "False ASDS", "True RTLS", "True ASDS", "False RTLS"]}
df = pd.DataFrame(data)

# Use .value_counts() on the column
landing_outcomes = df["Outcome"].value_counts()

print(landing_outcomes)

# Data Quality Assessment
Data Wrangling
We can see below that some of the rows are missing values in our dataset.
```python
data_falcon9.isnull().sum()
```
1. Replacing null values with mean

```python
Dealing with Missing Values
Calculate below the mean for the PayloadMass using the .mean(). Then use the mean and the .replace() function to replace np.nan values in the data with the mean you calculated.

# Calculate the mean value of PayloadMass column
PayloadMassMean = df['PayloadMass'].mean()
# Replace the np.nan values with its mean value
df['PayloadMass'] = df['PayloadMass'].replace(np.nan, PayloadMassMean)
```

You should see the number of missing values of the PayLoadMass change to zero.


