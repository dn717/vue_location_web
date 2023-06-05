<!-- eslint-disable-next-line vue/multi-word-component-names -->
<template>
  <div class="container mt-4 mb-4">
    <div class="row mb-4">
      <div class="col-6">
        <input v-model="searchInput" class="form-control" placeholder="Enter a location" @keyup.enter="searchLocation" />
      </div>
      <div class="col-2">
        <button class="btn btn-primary" @click="searchLocation">Search</button>
      </div>
      <div class="col-2">
        <button class="btn btn-primary" @click="getCurrentLocation">Get Current Location</button>
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <div id="map"></div>
      </div>
    </div>
    <div class="row mt-4">
      <div class="col-12">
        <h3>Last Searched Location:</h3>
        <p>Location: {{ locationName }}</p>
        <p>Time Zone: {{ timeZone }}</p>
        <p>Local Time: {{ localTime }}</p>
      </div>
    </div>
    <div class="row mt-4">
      <div class="col-12">
        <h3>Search History:</h3>
        <table class="table table-bordered">
          <thead>
            <tr>
              <th>Select</th>
              <th>Location</th>
              <th>Time Zone</th>
              <th>Local Time</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in paginatedSearchHistory" :key="index">
              <td><input type="checkbox" v-model="selectedItems" :value="index"></td>
              <td>{{ item.location }}</td>
              <td>{{ item.timeZone }}</td>
              <td>{{ item.localTime }}</td>
            </tr>
          </tbody>
        </table>
        <div class="text-center">
          <pagination v-model="currentPage" :records="searchHistory.length" :list="searchHistory" :per-page="10" @input="handlePageChange"/>
        </div>
        <div class="text-center">
          <button class="btn btn-danger" @click="deleteSelectedItems">Delete Selected</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import axios from 'axios';
import Pagination from 'v-pagination-3';



export default {
  components:{
    Pagination,
  },
  data() {
    return {
      searchInput: '',
      map: null,
      markers: [],
      locationName: '',
      timeZone: '',
      localTime: '',
      searchHistory: [],
      selectedItems: [],
      currentPage: 1,
      perPage: 10
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = L.map('map').setView([0, 0], 2);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: 'Map data © <a href="https://openstreetmap.org">OpenStreetMap</a> contributors'
      }).addTo(this.map);
    },
    searchLocation() {
      const name = this.searchInput.trim();
      if (name) {
        axios
          .get('https://api.mapbox.com/geocoding/v5/mapbox.places/' + name + '.json', {
            params: {
              access_token: 'pk.eyJ1IjoiY2Fyb2x5bmoyMzMzIiwiYSI6ImNsaTZoN3hyNDBtamwzcm4xbThuNHZyZTEifQ.7CiKhGe9rZRPzttS2mDvVw',
              limit: 1 // Limit the results to one location
            }
          })
          .then(response => {
            const features = response.data.features;
            if (features.length > 0) {
              const coordinates = features[0].center;
              const lat = coordinates[1];
              const lng = coordinates[0];
              this.updateMap(lat, lng);
              this.addMarker(lat, lng);
              this.locationName = features[0].place_name;
              this.getTimeZone(lat, lng).then(() => {
                this.addToSearchHistory(this.locationName, this.timeZone, this.localTime,lat,lng);
              });
            
            } else {
              console.log('No results found');
            }
          })
          .catch(error => {
            console.log('Error:', error);
          });
      }
    },
    updateMap(lat, lng) {
      this.map.setView([lat, lng], 12);
    },
    addMarker(lat, lng) {
      const marker = L.marker([lat, lng]).addTo(this.map);
      this.markers.push(marker);
    },
    getTimeZone(lat, lng) {
      return axios
        .get('https://api.timezonedb.com/v2.1/get-time-zone', {
          params: {
            key: 'U085297ZCJHG',
            format: 'json',
            by: 'position',
            lat: lat,
            lng: lng
          }
        })
        .then(response => {
          //this.timeZone = response.data.zoneName;
          this.timeZone = response.data.nextAbbreviation;
          //const currentTime = new Date(response.data.timestamp * 1000);
          //this.localTime = currentTime.toLocaleString();
          this.localTime = response.data.formatted;
        })
        .catch(error => {
          console.log('Error:', error);
        });
    },
    addToSearchHistory(location, timeZone, localTime, lat, lng) {
      const searchItem = {
        location: location,
        timeZone: timeZone,
        localTime: localTime,
        lat: lat,
        lng: lng

      };
      this.searchHistory.unshift(searchItem);
    },
    getCurrentLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(position => {
          const lat = position.coords.latitude;
          const lng = position.coords.longitude;
          this.updateMap(lat, lng);
          this.addMarker(lat, lng);
          this.searchInput = ''; // Clear the search input
          //this.locationName = 'Current Location';
          //this.getTimeZone(lat, lng);
          //this.addToSearchHistory(this.locationName, this.timeZone, this.localTime);
        }, error => {
          console.log('Error:', error);
        });
      } else {
        console.log('Geolocation is not supported by this browser.');
      }
    },
    deleteSelectedItems() {
      const indexes = this.selectedItems.sort((a, b) => b - a);//通过 selectedItems 数组中的索引来迭代需要删除的项,降序排列
      indexes.forEach(index => {
        const item = this.searchHistory[index];//根据索引从 searchHistory 数组中获取相应的项
        const markerIndex = this.markers.findIndex(marker => marker.getLatLng().lat === item.lat && marker.getLatLng().lng === item.lng);//使用 findIndex 方法和 getLatLng() 函数来查找与 item.location 相匹配的标记
        if (markerIndex !== -1) {
          const marker = this.markers[markerIndex];
          this.map.removeLayer(marker);
          this.markers.splice(markerIndex, 1);
        }
        this.searchHistory.splice(index, 1);
      });
      this.selectedItems = [];
    },
    // removeMarker(location) {
    //   const index = this.searchHistory.findIndex(item => item.location === location);
    //   if (index !== -1 ) {
    //     const marker = this.markers[index];
    //     this.map.removeLayer(marker);  // remove markers from map
    //     this.markers.splice(index, 1);  // remove from markers array
    //   }
    // },
    handlePageChange(page) {
      this.currentPage = page;
    }
  },
  computed: {
    paginatedSearchHistory() {
      const startIndex = (this.currentPage - 1) * this.perPage;
      const endIndex = startIndex + this.perPage;
      return this.searchHistory.slice(startIndex, endIndex);
    }
  }
};
</script>

<style>
#map {
  height: 400px;
}
</style>

     
