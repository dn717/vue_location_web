# Simple Location Web

## Description
The project is a web application that allows users to search for locations, view them on a map, and store their search history. It provides features like acquiring the user's current location, searching for specific locations, displaying the searched locations on a map with markers, and maintaining a search history table with pagination. Additionally, it shows the time zone and local time of the latest searched location.

## Features
1. **Current Location Button:** Allows users to acquire their current location from their browser.
2. **Search Module:**  Users can input the name of a location and trigger the search feature by clicking a button or pressing the enter key on the keyboard.
3. **Map Display:** Displays the searched location on a map and adds a marker for each searched location whenever the location changes.
4. **Table with Pagination:** Includes a table to show all searched places with pagination functionality:
   - Displays a maximum of 10 records on each page.
   -  Provides checkboxes at the beginning of each row to allow users to select multiple records.
   - Offers a delete button at the bottom to remove all selected records and their respective markers from the map.
5. **Time Zone and Local Time:**Displays the time zone and local time of the latest searched location.

## Usage
To use the application, follow these steps:


1. Enter the name of a location in the search input field.
2. Press the Enter key on the keyboard or click the search button to initiate the search.
3. The map will update to display the searched location with a marker.
4. The search history table will be updated to include the searched location, time zone, and local time.
5. Use the pagination buttons at the bottom of the table to navigate through different pages.
6. To delete selected records, check the checkboxes next to the desired rows and click the delete button.


## Installation
To install and run the project locally, follow these steps:

1. Clone the repository: `git clone https://github.com/dn717/vue_location_web.git`
2. Navigate to the project directory:`cd project-directory`
3. Install the dependencies: `npm install`
4. Compile and start the development server:`npm run serve`
5. Open the application in your browser: `http://localhost:8080`(or the specified port)

## Technologies Used
The project utilizes the following technologies and frameworks:

- Vue.js: A JavaScript framework for building user interfaces
- Leaflet: An open-source JavaScript library for interactive maps
- Axios: A promise-based HTTP client for making API requests
- V-pagination-3: A Vue.js pagination component for displaying paginated data

API used:

- Mapbox Geocoding API
- TimeZoneDB API
