<template>
    <div>
        <label for="vectorSelect">Select an option:</label>
        <select v-model="selectedOption" id="vectorSelect">
            <option :value="null" disabled>Select an option</option>
            <option v-for="option in lineOptions" :key="option.id" :value="option">
                {{ option.label }}
            </option>
        </select>

        <div v-if="selectedOption">
            <!-- Display information about the selected option -->
            Selected Option: {{ selectedOption.id }}
        </div>
    </div>
    <div>
        <button @click="getPosts">Load list</button>
        <div v-for="post in  stopInData " :key="post.trip_id">
            <div v-for="out in stopOutData " :key="post.trip_id">

                <template v-if="shouldRenderPost(post, out)">

                    <h3> scheduled_arrival {{ post.scheduled_arrival }} Linha {{ post.line_id }} direção {{ post.headsign }}
                        chegada prevista a estação {{ out.scheduled_arrival }} tempo para chegar {{ this.timeDifference }}
                    </h3>

                </template>


            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    name: 'testList',
    data() {
        return {
            stopInData: [],
            stopinID: 170763,
            stopOutData: [],
            stopOutID: [172478, 172479],
            validLineIds: ['1715', '1228'],
            lineOptions: [
                { id: 0, label: 'linha 1715' },
                { id: 1, label: 'linha 1228' },

            ],
            selectedOption: null  // Initially, no option is selected
        };
    },

    // Call the method when the component is mounted

    methods: {
        getPosts() {
            this.getCurrentTime();
            console.log('getPosts method called');
            const apiUrl = `https://api.carrismetropolitana.pt/stops/${this.stopinID}/realtime`
            axios.get(apiUrl)
                .then((response) => {
                    this.stopInData = response.data;
                    console.log(this.stopInData);
                })
                .catch((error) => {
                    console.error('Error fetching data:', error);
                });
            const apiUrl2 = `https://api.carrismetropolitana.pt/stops/${this.stopOutID[this.selectedOption.id]}/realtime`
            axios.get(apiUrl2)
                .then((response) => {
                    this.stopOutData = response.data;
                    console.log(this.stopOutData);
                })
                .catch((error) => {
                    console.error('Error fetching data:', error);
                });

        },
        shouldRenderPost(post, out) {
            this.calculateTimeDifference(this.currentTime, post.scheduled_arrival)
            return (
                this.validLineIds.includes(post.line_id) &&
                post.scheduled_arrival < this.currentTime &&
                post.trip_id === out.trip_id


            );
        },
        getCurrentTime() {
            const now = new Date();
            const hours = now.getHours();
            const minutes = now.getMinutes();
            const seconds = now.getSeconds();

            // Format the time as HH:mm:ss
            this.currentTime = `${this.padZero(hours)}:${this.padZero(minutes)}:${this.padZero(seconds)}`;

            // Update the time every second (1000 milliseconds)

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
    },
};
</script>

<style lang="postcss" scoped></style>


