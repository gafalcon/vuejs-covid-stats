<template>
    <select id="" name="" v-model="selected" class="form-select mt-10 block w-full border p-3 rounded" @change="onChange()">
        <option value="0">Select Country from list or click on map to view country data</option>
        <option v-for="country in countries" :value="country.Slug">
            {{ country.Country }}
        </option>
    </select>
</template>

<script>
export default {
    name: 'CountrySelect',
    props: ['countries'],
    data(){
        return {
            selected: 0
        }
    },
    methods: {
        onChange(){
            const country = this.countries.find((item) => item.Slug === this.selected)
            console.log(country)
            this.$emit('get-country', country)
        }
    }

}
</script>
