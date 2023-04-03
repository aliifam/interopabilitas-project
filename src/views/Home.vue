<template>
    <v-container>
        <h1>Jadwal Sholat</h1>

        <br />

        <p>Your Location : {{ location }}</p>

        <br />

        <v-select
            label="Jenis Jadwal"
            :items="['Per - Hari', 'Per - Bulan']"
        ></v-select>

        <v-autocomplete
            v-model="kota"
            :items="allcityandcode"
            item-title="lokasi"
            item-value="id"
            label="Pilih Kota"
            persistent-hint
            return-object
            single-line
            clearable
        ></v-autocomplete>

        <br />

        <v-btn block size="large" @click="generateJadwal"
            >Generate Jadwal</v-btn
        >
    </v-container>
</template>

<script>
import axios from "axios";
export default {
    data() {
        return {
            location: null,
            jenis: null,
            kota: null,
            lat: null,
            long: null,
            jadwal: null,
            allcity: [],
            allcityandcode: [],
            nowdate: null,
            nowmonth: null,
        };
    },
    methods: {
        async fetchAllCity() {
            const AllCity = await axios.get(
                "https://api.myquran.com/v1/sholat/kota/semua"
            );
            this.allcityandcode = AllCity.data;
            let listofcityname = [];
            AllCity.data.forEach((element) => {
                listofcityname.push(element.lokasi);
            });
            this.allcity = listofcityname;
        },

        async fetchNowCity() {
            //get now location from browser
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition((position) => {
                    this.lat = position.coords.latitude;
                    this.long = position.coords.longitude;
                });
            }

            //get city name from lat and long from free api
            const nowCity = await axios.get(
                "https://api.bigdatacloud.net/data/reverse-geocode-client?latitude=" +
                    this.lat +
                    "&longitude=" +
                    this.long +
                    "&localityLanguage=id"
            );
            this.location = nowCity.data.city;
        },

        generateJadwal() {
            this.nowmonth = new Date().getMonth() + 1;
            this.nowdate = new Date().getDate();
            console.log(this.nowdate);
        },
    },
    async mounted() {
        await this.fetchAllCity();
        await this.fetchNowCity();
    },
};
</script>
