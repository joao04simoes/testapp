<template>
    <div>
        <div id="map"></div>
        <button @click="putVehicles">put cars</button>
        <label for="vectorSelect" class="label">Select an option:</label>
        <select v-model="selectedOption" class="selectLabel" id="vectorSelect">
            <option :value="null" disabled>Select an option</option>
            <option v-for="option in lineOptions" :key="option.id" :value="option">
                {{ option.label }}
            </option>
        </select>

        <div v-if="selectedOption">
            <div v-for="post in stopInData" :key="post.trip_id">
                <div v-for="out in stopOutData" :key="post.trip_id">

                    <div v-if="shouldRenderPost(post, out)">

                        <h3 class="dataLines">chegada a estação as {{ out.scheduled_arrival }} tempo para chegar {{
                            this.timeDifference }}
                        </h3>

                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

import L from 'leaflet';
import 'leaflet/dist/leaflet.css';


export default {

    name: 'testList',
    data() {
        return {
            vehicleMarkers: [],
            vehicles: [],
            stopInData: [],
            stopinID: 170763,
            stopOutData: [],
            stopOutID: [172478, 172479],
            validLineIds: ['1715', '1228'],
            tripID: [2, 3, 3, 4, 454, 54],
            lineOptions: [
                { id: 0, label: 'linha 1715' },
                { id: 1, label: 'linha 1228' },
            ],
            selectedOption: null  // Initially, no option is selected
        };
    },

    computed: {
        triggerDataFetch() {
            return this.selectedOption ? this.selectedOption.id : null;
        },
    },

    watch: {
        triggerDataFetch: 'getPosts'
    },

    methods: {
        getPosts() {
            if (this.triggerDataFetch !== null) {
                this.getCurrentTime();
                console.log('getPosts method called');
                const apiUrl = `https://api.carrismetropolitana.pt/stops/${this.stopinID}/realtime`;
                axios.get(apiUrl)
                    .then((response) => {
                        this.stopInData = response.data;
                        console.log(this.stopInData);
                    })
                    .catch((error) => {
                        console.error('Error fetching data:', error);
                    });

                const apiUrl2 = `https://api.carrismetropolitana.pt/stops/${this.stopOutID[this.triggerDataFetch]}/realtime`;
                axios.get(apiUrl2)
                    .then((response) => {
                        this.stopOutData = response.data;
                        console.log(this.stopOutData);
                    })
                    .catch((error) => {
                        console.error('Error fetching data:', error);
                    });
            }
        },

        async getVehicles() {
            const apiUrl = 'https://api.carrismetropolitana.pt/vehicles';

            return axios.get(apiUrl)
                .then((response) => {
                    this.vehicles = response.data;
                    return response.data; // Optionally, you can return the data if needed
                })
                .catch((error) => {
                    console.error('Error fetching data:', error);
                    throw error; // Re-throw the error to propagate it to the caller
                });
        },
        async putVehicles() {
            await this.getVehicles();

            // Clear existing markers by removing the previous layer group
            if (this.markerLayerGroup) {
                this.map.removeLayer(this.markerLayerGroup);
            }

            // Create a new layer group for markers
            this.markerLayerGroup = L.layerGroup().addTo(this.map);

            for (const vehicle of this.vehicles) {
                if (vehicle.lat !== undefined && vehicle.lon !== undefined && this.tripID.includes(vehicle.trip_id)) {
                    L.marker([vehicle.lat, vehicle.lon])
                        .addTo(this.markerLayerGroup)
                        .bindPopup(`Vehicle ID: ${vehicle.id}<br>Status: ${vehicle.current_status}`);
                }
            }
        },




        shouldRenderPost(post, out) {
            this.calculateTimeDifference(this.currentTime, post.scheduled_arrival)
            if (this.validLineIds.includes(post.line_id) &&
                post.scheduled_arrival > this.currentTime &&
                post.trip_id === out.trip_id) {
                this.tripID.push(post.trip_id);
                return (1);
            }
            return (0);
        },

        getCurrentTime() {
            const now = new Date();
            const hours = now.getHours();
            const minutes = now.getMinutes();
            const seconds = now.getSeconds();

            // Format the time as HH:mm:ss
            this.currentTime = `${this.padZero(hours)}:${this.padZero(minutes)}:${this.padZero(seconds)}`;
        },

        calculateTimeDifference(start, end) {
            const startTime = new Date(`2022-01-01 ${start}`);
            const endTime = new Date(`2022-01-01 ${end}`);

            const timeDifferenceInMilliseconds = endTime - startTime;
            const hours = Math.floor(timeDifferenceInMilliseconds / (1000 * 60 * 60));
            const minutes = Math.floor((timeDifferenceInMilliseconds % (1000 * 60 * 60)) / (1000 * 60));
            const seconds = Math.floor((timeDifferenceInMilliseconds % (1000 * 60)) / 1000);

            this.timeDifference = `${this.padZero(hours)}:${this.padZero(minutes)}:${this.padZero(seconds)}`;
        },

        padZero(value) {
            // Add leading zero if the value is less than 10
            return value < 10 && value > -1 ? `0${value}` : value;
        }
    },

    mounted() {
        console.log('testList component mounted');
        this.map = L.map('map').setView([38.771491, -9.281025], 10);

        // Add a tile layer (you can use other tile layers as well)
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
        }).addTo(this.map);
    },
};
</script>

<style lang="postcss" scoped>
.selectLabel {
    /* Your styles go here */
    font-weight: bold;
    color: #c6c861;
    background-color: rgb(35, 35, 33);

    /* Add any other styles you want */
}

.label {
    font-weight: bold;
    color: #e7ba72;

}

.dataLines {
    font-weight: bold;
    color: rgb(0, 0, 0);
    //background-image: url('https://images.freeimages.com/images/large-previews/d0a/american-flag-1192239.jpg?fmt=webp&w=500');
}

#map {
    height: 600px;
    width: 600px;
}
</style>



