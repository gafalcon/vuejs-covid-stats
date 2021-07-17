<template>
    <div id="map-canvas" class="mt-2" style="width: 100%;height: 800px;"></div>
</template>

<script>
export default {
    name: 'GMap',
    data() {
        return {
            countries: []
        }
    },
    methods: {
        async loadCountries(){
            const res = await fetch('data.json')
            const data = await res.json()
            return data
        },

        createCountry(coords, country){
            var currentCountry = new google.maps.Polygon({
                paths: coords,
                //strokeColor: 'white',
                title: country.country,
                code: country.iso,
                strokeOpacity: 0,
                //strokeWeight: 1,
                //fillColor: country['color'], // can be used as default color
                fillOpacity: 0
            });

            this.countries.push(currentCountry);
        },
        showCountries(map) {
            for (var i=0; i<this.countries.length; i++) {
                let country = this.countries[i]
                this.countries[i].setMap(map);

                google.maps.event.addListener(this.countries[i],"mouseover",function(){
                    this.setOptions({fillColor: "#f5c879", 'fillOpacity': 0.5});
                });

                google.maps.event.addListener(this.countries[i],"mouseout",function(){
                    this.setOptions({fillColor: "#f5c879", 'fillOpacity': 0});
                });

                google.maps.event.addListener(this.countries[i], 'click', function(event) {
                    this.$emit('get-country', country.title)
                }.bind(this));
            }
        }
    },
    async mounted() {
        let countries = [];

        let mapOptions = {
            zoom: 3,
            minZoom: 1,
            center: new google.maps.LatLng(50.7244893,3.2668189),
            mapTypeId: google.maps.MapTypeId.ROADMAP,
            backgroundColor: 'none'
        };

        let map = new google.maps.Map(document.getElementById('map-canvas'), mapOptions);

        let countryData = await this.loadCountries()

        if (countryData){
            countryData.forEach((country, id) => {
                let countryCoords
                let ca;
                let co;
                if ('multi' in country){
                    let ccArray = [];
                    for (var t in country['xml']['Polygon']) {

                        countryCoords = [];

                        co = country['xml']['Polygon'][t]['outerBoundaryIs']['LinearRing']['coordinates'].split(' ');

                        for (var i in co) {

                            ca = co[i].split(',');

                            countryCoords.push(new google.maps.LatLng(ca[1], ca[0]));
                        }

                        ccArray.push(countryCoords);
                    }
                    this.createCountry(ccArray,country);
                }else{
                    countryCoords = [];

                    co = country['xml']['outerBoundaryIs']['LinearRing']['coordinates'].split(' ');

                    for (var j in co) {

                        ca = co[j].split(',');

                        countryCoords.push(new google.maps.LatLng(ca[1], ca[0]));
                    }

                    this.createCountry(countryCoords,country);
                }

            })

            this.showCountries(map);
        }
    }
}
</script>
