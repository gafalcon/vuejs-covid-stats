<template>
    <main v-if="!loading">
        <div class="grid grid-cols-2">
            <DataTitle :text="title" :dataDate="dataDate" />
            <DataBoxes :stats="stats" />
        </div>
    </main>

    <main class="flex flex-col align-center justify-center text-center" v-else>
        <div class="text-gray-500 text-3xl mt-10 mb-6">
            Fetching Data...
        </div>
        <img :src="loadingImage" class="w-24 m-auto" alt="hourglass" />
    </main>

    <CountrySelect @get-country="getCountryData" :countries="countries" />
    <GMap @get-country="getCountryData" />

    <button class="bg-green-700 text-white rounded p-3 mt-10 focus:outline-none hover:bg-green-600" v-if="title !== 'Global'" @click="clearCountryData">Show Global</button>
</template>

<script>
// @ is an alias to /src
import DataTitle from '@/components/DataTitle'
import DataBoxes from '@/components/DataBoxes'
import CountrySelect from '@/components/CountrySelect'
import GMap from '@/components/GMap'

export default {
    name: 'Home',
    components: { DataTitle, DataBoxes, CountrySelect, GMap},
    data() {
        return {
            loading: true,
            title: 'Global',
            dataDate: '',
            stats: {},
            countries: [],
            loadingImage: require('../assets/loading-buffering.gif'),
        }
    },
    methods: {
        async fetchCovidData() {
            const res = await fetch('https://api.covid19api.com/world')
            const data = await res.json()
            return data;
        },
        async fetchCountries() {
            const res = await fetch('https://api.covid19api.com/countries')
            const data = await res.json()
            return data;
        },
        async fetchCountry(slug){
            const res = await fetch(`https://api.covid19api.com/live/country/${slug}`)
            const data = await res.json()
            return data;
        },
        async getCountryData(country){
            this.loading = true
            let countryData
            if (country.Slug)
                countryData = await this.fetchCountry(country.Slug)
            else
                countryData = await this.fetchCountry(country.toLowerCase())
            this.title= country.Country || country;
            this.loading = false
            const len = countryData.length - 1
            this.stats = {
                NewConfirmed: Math.max(countryData[len].Confirmed - countryData[len-1].Confirmed, 0),
                TotalConfirmed: countryData[len].Confirmed,
                NewDeaths: Math.max(countryData[len].Deaths - countryData[len-1].Deaths, 0),
                TotalDeaths: countryData[len].Deaths
            }
        },
        async clearCountryData(){
            this.loading = true
            const data = await this.fetchCovidData()
            this.title = 'Global'
            this.stats = data[0]
            this.loading = false
        }
    },
    async created() {
        this.fetchCountries().then(countries => this.countries = countries)
        const data = await this.fetchCovidData();
        this.dataDate = data[0].Date
        this.stats = data[0]
        this.loading = false
    }
}
</script>
