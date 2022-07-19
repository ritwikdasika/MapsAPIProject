<template>
    <div>
<section class="ui two column centered grid" style = "position: relative;z-index: 1;">
        <div class="red column">
            <form class="ui segment large form">
                <div id = "error" class="ui message red" v-show = "error"> {{error}}</div>
                <div class="ui segment">
                    <div class="field">
                        <div class="ui right icon input large" :class = {loading:spinner}>
                            <input type="text" placeholder="Enter your address here" v-model = "address" id = "autocomplete"/>
                            <i class="map marker link icon" @click="locatorButtonPressed"></i>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </section>

    <section id = "map" style="width: 100%; height: 100%">

    </section>
    </div>
    

</template>

<script>

import axios from 'axios';

export default {
    data(){
        return {
            address: "",
            error: "",
            locationAccess: false,
            spinner: false
        }
    },

    mounted(){
        let autocomplete = new google.maps.places.Autocomplete(
            document.getElementById("autocomplete")
        );
        autocomplete.addListener("place_changed",() => {
            let place = autocomplete.getPlace();
            console.log(place);
            this.showUserLocationOnTheMap(place.geometry.location.lat(), place.geometry.location.lng());
        })
    },

    methods: {
        locatorButtonPressed() {
            this.spinner = true;
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(position => {
                    this.getAddressFrom(position.coords.latitude, position.coords.longitude);
                    this.spinner = false;
                    this.showUserLocationOnTheMap(position.coords.latitude, position.coords.longitude);
                },
                    error => {
                        this.error = "Unable to find address. Please allow access to your location.";
                        this.spinner = false;
                        console.log(error.message);
                    });
            } else {
                this.error = error.message;
                this.spinner = false;
                console.log("Browser does not support geolocation API");
            }
        },
        getAddressFrom(lat, long){
            axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng=" 
            + lat 
            + "," 
            + long 
            //+ "&key=AIzaSyA3_ZTgNU6rZw6ARf6Yk65Ydil1QHuoSN4") // old API key
            //+ "&key=AIzaSyDn0lAo0sT4wlY_lxUB1AYP7HN4rJFbeC0")
            + "&key=AIzaSyCLzifHTZfqqvH4ok7UuzrRtfNTXGjtPXA")
            .then(response => {
                if(response.data.error_message){
                    this.error = response.data.error_message;
                    this.spinner = false;
                    console.log(response.data.error_message);
                } else {
                    this.address = response.data.results[0].formatted_address;
                }
            })
            .catch(error =>{
                this.spinner = false;
                this.error = error.message;
            });
        },
        showUserLocationOnTheMap(latitude, longitude){
            //create google maps object
            const map = new google.maps.Map(document.getElementById("map"),{
                zoom:14,
                center: new google.maps.LatLng(latitude, longitude),
                mapTypeId:google.maps.MapTypeId.ROADMAP
                
            })

            //add position marker
            new google.maps.Marker({
                position: new google.maps.LatLng(latitude,longitude),
                map:map
            })
        }
    }
}
</script>

<style>
.ui.button,
.map.marker.alternate.icon {
    background-color: #ff5a5f;
    color: #fff;
}

.pac-icon{
    display:none
}

.pac-item{
    padding:10px;
    font-size:16px;
    cursor: pointer;
}
.pac-item:hover{
    padding:10px;
    font-size:16px;
    cursor: pointer;
    color:#fff
}

.pac-item-query{
    font-size:16px;
}
.pac-item-query:hover{
    font-size:16px;
    color:#fff;
}

.pac-item:hover{
    background-color:lightslategray;
}

#map{
    position:absolute;
    top:0;
    right:0;
    bottom:0;
    left:0;
}

</style>