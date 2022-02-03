# World Weather Analysis

## Overview

For thos project a PlanMyTrip app is being developed. Management has recommended adding the weather description to the weather data, so beta testers can input statements to filter the data for their weather preferences, which will be used to identify potential travel destinations and nearby hotels. From the list of potential travel destinations, the beta tester will choose four cities to create a travel itinerary. Finally, using the Google Maps Directions API, a travel route is created between the four cities as well as a marker layer map.

### Resources
- Data source: The audit was performed on the file "election_results.cvs" provided by the Colorado Board of Election. The following image shows the structure of the file.

- Software use to perform the analysis: Python 3.7.11, Anacoda 4.11, Jupyter Notebook 6.4.6

- APIs from OpenWeatherMap and Google Maps

## Results

### Retrieve Weather Data

1. Generate a set of 2,000 random latitudes and longitudes.

    - Using numpy random function we created combinatios for 2,000 latitudes (values from -90 to 90) and longitudes (values from -180 to 180)

    ![random coordinates](/Resources/random_coords.png)

2. Using the *citipy* module we retreived the name of the cities for each location. 

    - From the 2,000 coordinates, there were 729 cities located.

3. We used the OpenWeatherMap API to retreive data weather from the cities located and created a DataFrame.  Data include: city name, country code, latitude, longitude, maximum temperature (°F), humidity (%), cloudiness (%), wind speed (MPH) and the current weather description.

    - If the city was not located it was not added to the final DataFrame.  The resulting DataFrame contained 671 cities.

        ![cities datafram](/Resources/cities_df.png)

    - Finally the DataFrame was exported to the csv file: "WeatherPy_Database.csv"

        |Input example for minimum temp |![input for min temp](/Resources/min_temp.png) |


### Create a Customer Travel Destinations Map

1. Import the weather database created in the previous step.

2. Ask the user for a minimum and a maximum temperature to filter the cities.

    - We established some conditions in order to avoid entering strings instead of values and checking if maximum temperature was equal or higher than the minimum temperature entered.

         ```python
        try:
            min_temp = float(min_temp)
            min_ok = True
        except ValueError:
            print('Input is not a number')
        ```
         ```python
        try:
            max_temp = float(max_temp)
            if max_temp >= min_temp:
                max_ok = True
            else:
                print('Maximum temperature is not >= than minimum temperature')
        except ValueError:
            print('Input is not a number')        
        ```

3. Using the temperature preferences from the user we filtered the DataFrame, using the *loc* method, and then drop rows if they have any missing data.  We then create a new DataFrame and add a column to place hotel names.

        ![hotel dataframe](/Resources/hotel_df.png)

4. Using Google Places Nearby Search we make a "lodging" request using the coordinates ('Lat','Lng') for each row in our DataFrame.

    - We obtain a *json* object and try to grab the first hotel name from the response.

        ```python
        try:
            hotel_df.loc[index, "Hotel Name"] = hotels["results"][0]["name"]
        except (IndexError):
            print("Hotel not found... skipping.")
        ```
    - We get a DataFrame in which we only keep the rows with information about the hotel name, and the save it to the file "WeatherPy_vacation.csv".

        ![final hotel dataframe](/Resources/clean_hotel_df.png)

    - Finally we plot our cities in a Google Map with markers and pop-up information boxes containing the name of the hotel, city name, country code, and weather conditions.

        ![hotel map](/Vacation_Search/WeatherPy_vacation_map.png)

### Create a Travel Itinerary Map

1. Use the Google Directions API to create a travel itinerary that shows the route between four cities chosen from the customer’s possible travel destinations. 

    - From the "WeatherPy_vacation.csv", we specify four cities within a region for a possible itinerary. In our case, we choose in the US pacific coast:
        - Pacific Grove
        - Fremont
        - Ukiah
        - Laguna
    
    - For each city a DataFrame is created with the *loc* method. Then we get the coordinates ('Lat','Lng') as tuples, using the *to_numpy* function. Starting and ending cities is the same.

        Example:
        ```python
        start = vacation_start["Lat"].to_numpy()[0], vacation_start["Lng"].to_numpy()[0]
        end = vacation_end["Lat"].to_numpy()[0], vacation_end["Lng"].to_numpy()[0]
        ```
    
    - We create a gmaps direction layer with the route: Start -> Stop1 -> Stop2 -> Stop3 -> End. 

       ![route map](/Vacation_Itinerary/WeatherPy_travel_map.png)


2. Create a marker layer map with a pop-up marker for each city on the itinerary.

    - Using the pandas *concat()* function we combine the cities individual DataFrames into a single one, from which we'll create the markers locations and information boxes to map them.

       ![markers map](/Vacation_Itinerary/WeatherPy_travel_map_markers.png)