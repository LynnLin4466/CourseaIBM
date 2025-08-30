How to connect to Data Sources
```python
import requests

url = "https://api.spacexdata.com/v4/launches/past"
response = requests.get(url)   # Send GET request to API
data = response.json()         # Parse response into Python dict/list

print(data[0])   # Example: print first launch record
```
