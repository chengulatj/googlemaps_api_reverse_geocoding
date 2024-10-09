# Google Maps API Reverse Geocoding

## Description
This repository demonstrates how to use the **Google Maps Geocoding API** to reverse geocode latitude and longitude coordinates, extracting **county information**. The tool also converts **Degrees-Minutes-Seconds (DMS)** coordinates to **decimal format** before using the API for geocoding.

## Features
- Convert latitude and longitude from **DMS** to **decimal format**.
- Use the **Google Maps API** to reverse geocode the coordinates.
- Extract the **county** from the geocoded location data.
- Handle API errors such as invalid coordinates or API rate limits.
- Save the enriched data to an **Excel file**.

## Requirements
- **Python 3.x**
- **Google Maps API Key**: You'll need an API key from Google Cloud Platform to access the **Google Maps Geocoding API**.
  
### Python Libraries:
- **pandas**: For data handling.
- **googlemaps**: For accessing the Google Maps Geocoding API.
- **openpyxl**: For saving results to Excel.

## Set up and Installations
### 1. Open Google Colab

You can access Google Colab through this link: [Google Colab](https://colab.research.google.com/).

### 2. Install Required Libraries

In your Colab notebook, you'll need to install the required Python libraries by running the following command:

```python
!pip install pandas googlemaps openpyxl
```

- `pandas`: For handling data and reading/writing Excel files.
- `googlemaps`: To access the **Google Maps Geocoding API**.
- `openpyxl`: To write the output to Excel files.

### 3. Set Up Your Google Maps API Key
You’ll need an API key from **Google Cloud** to use the **Google Maps Geocoding API**. Follow these steps:

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Enable the **Geocoding API** and generate an API key.
3. Copy the key and use it in the Colab code:
   
   ```python
   API_KEY = 'your-google-maps-api-key'
   ```
### API Key Protection

To protect the Google Maps API key, it was not directly pasted into the Colab notebook. Instead, the API key is securely stored in a file called `KEY.txt`, which is uploaded to **Google Drive**. The key is then read from the file and used in the notebook.:

The API key was saved in a text file called `KEY.txt` in your **Google Drive** with the following format:
   ```txt
   API_KEY=your-google-maps-api-key
   ```
## Code Usage (Google Colab)

Here's a step-by-step explanation of the code you'll use in your Colab notebook:

### 1. Import Required Libraries

```python
import pandas as pd
import re
import googlemaps
import time
```
### 2. Read the Excel File

Upload your file to Colab and read it using **pandas**:

```python
locations = pd.read_excel('/content/SL0202_Replacement_Locations.xlsx', skiprows=1)
```
- The `skiprows=1` ensures that any extra header row is skipped.
  
### 3. Convert DMS to Decimal

Define the `dms_to_dd` function to convert **DMS (Degrees, Minutes, Seconds)** to **decimal** format:

```python
def dms_to_dd(dms_str):
    dms_str = str(dms_str)
    parts = re.split('[°\'"]+', dms_str)

    if len(parts) != 4:
        return float('nan')  # Return NaN for invalid inputs

    degrees = float(parts[0])
    minutes = float(parts[1])
    seconds = float(parts[2])
    direction = parts[3].strip()

    dd = degrees + minutes / 60 + seconds / 3600
    if direction in ['S', 'W']:
        dd *= -1  # South and West are negative
    return dd
```

### 4. Apply DMS to Decimal Conversion

Convert the **Latitude** and **Longitude** columns from **DMS** to **decimal** format:

```python
latlongs = locations[['Latitude', 'Longitude']]
latlongs['Latitude_dec'] = latlongs['Latitude'].apply(dms_to_dd)
latlongs['Longitude_dec'] = latlongs['Longitude'].apply(dms_to_dd)
```
### 5. Initialize Google Maps Client

```python
gmaps = googlemaps.Client(key=API_KEY)
```
### 6. Define the Reverse Geocoding Function

```python
def get_county_google(lat, lon):
    try:
        if pd.isna(lat) or pd.isna(lon):
            return 'Invalid Coordinates'
        if not (-90 <= lat <= 90) or not (-180 <= lon <= 180):
            return 'Invalid Lat/Long Range'

        # Use Google Maps API to reverse geocode
        result = gmaps.reverse_geocode((lat, lon))

        # Extract the county from the address components
        for component in result[0]['address_components']:
            if 'administrative_area_level_2' in component['types']:
                return component['long_name']

        return 'County not found'
    except Exception as e:
        print(f"Error: {e}")
        return 'Error'
```

### 7. Apply Reverse Geocoding to Get County Information

Use the **Google Maps API** to get the county for each location:

```python
latlongs['County'] = latlongs.apply(lambda row: get_county_google(row['Latitude_dec'], row['Longitude_dec']), axis=1)
```
### 8. Add the County Information Back to the Original DataFrame

```python
locations['County'] = latlongs['County']
```
### 9. Save the Results to Excel (Or a desired format)

Save the output as an Excel file with the county information included:

```python
locations.to_excel('/content/SL0202_Replacement_Locations_with_County.xlsx', index=False)
```
## Error Handling

- **Invalid Coordinates**: The script checks for invalid latitude/longitude values and handles these gracefully.
- **API Errors**: If the Google Maps API call fails due to timeouts or quota issues, the script will print the error and continue processing.

## Limitations

- **Google Maps API Quota**: The free tier of the Google Maps Geocoding API has usage limits, so you may encounter quota restrictions if running the script on a large dataset.
- **DMS Format**: The input coordinates must be in the proper DMS format. Ensure that the latitude and longitude data follow the structure shown in the example.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

