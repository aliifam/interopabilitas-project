<template>
    <v-container>
        <h1>Jadwal Sholat</h1>

        <br />

        <p>Your Location : {{ location }}</p>

        <br />

        <v-select
            label="Jenis Jadwal"
            :items="['Per - Hari', 'Per - Bulan']"
            v-model="jenis"
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

        <br />

        <v-card v-if="jadwal && jenis == 'Per - Hari'">
            <v-card-title>
                <h2>Jadwal Sholat Harian {{ jadwal.data.lokasi }}</h2>
            </v-card-title>
            <v-card-text>
                <v-simple-table>
                    <template v-slot:default>
                        <p>Lokasi : {{ jadwal.data.lokasi }}</p>
                        <p>Tanggal : {{ jadwal.data.jadwal.tanggal }}</p>

                        <v-divider></v-divider>

                        <p>Subuh : {{ jadwal.data.jadwal.subuh }}</p>
                        <p>Dzuhur : {{ jadwal.data.jadwal.dzuhur }}</p>
                        <p>Ashar : {{ jadwal.data.jadwal.ashar }}</p>
                        <p>Maghrib : {{ jadwal.data.jadwal.maghrib }}</p>
                        <p>Isya : {{ jadwal.data.jadwal.isya }}</p>

                        <v-divider></v-divider>

                        <p>Imsak : {{ jadwal.data.jadwal.imsak }}</p>
                        <p>Terbit : {{ jadwal.data.jadwal.terbit }}</p>
                        <p>Dhuha : {{ jadwal.data.jadwal.dhuha }}</p>
                    </template>
                </v-simple-table>
            </v-card-text>
        </v-card>

        <v-card v-if="jadwal && jenis == 'Per - Bulan'">
            <v-card-title>
                <h2>Jadwal Sholat Bulanan {{ jadwal.data.lokasi }}</h2>
            </v-card-title>

            <v-card-text v-for="item in jadwal.data.jadwal" :key="item">
                <v-simple-table>
                    <template v-slot:default>
                        <p>Tanggal : {{ item.tanggal }}</p>

                        <v-divider></v-divider>

                        <p>Subuh : {{ item.subuh }}</p>
                        <p>Dzuhur : {{ item.dzuhur }}</p>
                        <p>Ashar : {{ item.ashar }}</p>
                        <p>Maghrib : {{ item.maghrib }}</p>
                        <p>Isya : {{ item.isya }}</p>

                        <v-divider></v-divider>

                        <p>Imsak : {{ item.imsak }}</p>
                        <p>Terbit : {{ item.terbit }}</p>
                        <p>Dhuha : {{ item.dhuha }}</p>
                    </template>
                </v-simple-table>
            </v-card-text>
        </v-card>

        <br />
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
            nowyear: null,
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
            this.nowdate = new Date().getDate();
            this.nowmonth = new Date().getMonth() + 1;
            this.nowyear = new Date().getFullYear();

            if (this.jenis == "Per - Hari") {
                this.fetchJadwalHarian();
            } else {
                this.fetchJadwalBulanan();
            }
        },

        async fetchJadwalHarian() {
            const Jadwal = await axios.get(
                "https://api.myquran.com/v1/sholat/jadwal/" +
                    this.kota.id +
                    "/" +
                    this.nowyear +
                    "/" +
                    this.nowmonth +
                    "/" +
                    this.nowdate
            );
            this.jadwal = Jadwal.data;
            console.log(this.jadwal.data);
        },

        async fetchJadwalBulanan() {
            const Jadwal = await axios.get(
                "https://api.myquran.com/v1/sholat/jadwal/" +
                    this.kota.id +
                    "/" +
                    this.nowyear +
                    "/" +
                    this.nowmonth
            );
            this.jadwal = Jadwal.data;
            console.log(this.jadwal.data);
        },
    },
    async mounted() {
        await this.fetchAllCity();
        await this.fetchNowCity();
    },
};
</script>
