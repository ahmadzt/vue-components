<template>
    <div class="direction-container">
        <div class="relative">
            <button 
                @click="hidePanel()"
                class="btn btn-default close-btn">
                <span class="fa fa-times"></span>
            </button>
        </div>
        <div class="shadow p-4">
            <div class="mb-2">
                <input 
                    id="start"
                    type="text" 
                    class="w-full border leading-loose outline-none rounded"
                    v-model="start"
                    placeholder="From"
                    @input="directionStart()">
            </div>
            <div>
                <input 
                    id="end"
                    type="text" 
                    class="w-full border leading-loose outline-none rounded"
                    v-model="end"
                    placeholder="To"
                    @input="directionEnd()">
            </div>
            <div 
                v-if="directionInfo"
                class="mt-4">
                <div class="mb-2">via <b>{{ directionInfo.summary }}</b></div>
                <div class="flex justify-between">
                    <div>{{ directionInfo.legs[0].distance.text }}</div>
                    <div>{{ directionInfo.legs[0].duration.text }}</div>
                </div>
            </div>
        </div>
        <div 
            v-if="directionSteps.length > 0"
            class="p-4 overflow-y-auto overflow-x-hidden bg-white">
            <div 
                v-for="(step, index) in directionSteps"
                :key="index"
                class="mb-4 border-b flex">
                <div class="flex items-center justify-center text-xl text-grey-darker pr-4">
                    <span>{{index+1}}</span>
                </div>
                <div class="flex-1">
                    <div 
                        v-html="step.instructions"
                        class="mb-2"></div>
                    <div class="flex justify-between mb-2">
                        <div>{{ step.distance.text }}</div>
                        <div>{{ step.duration.text }}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import { debounce } from "lodash";

export default {
    props: {
        map: {
            type: Object,
            required: true
        },
    },
    data() {
        return {
            start: null,
            end: null,
            pos: null,
            marker: null,
            geocoder: null,
            geoPermission: false,
            direction: null,
            directionInfo: null,
            directionSteps: [],
            directionsService: null,
            directionsDisplay: null,
            originPlaceId: null,
            destinationPlaceId: null,
            clickListener: null,
            dragListener: null
        }
    },
    methods: {
        async init() {
            if (this.position) {
                this.pos = {...this.position};
                this.setMarker();
            } else {
                let result = await navigator.permissions.query({name:'geolocation'});

                if (navigator.geolocation && result.state == 'granted') {
                    this.geoPermission = true;
                    
                    navigator.geolocation.getCurrentPosition(position => {
                        this.pos = {
                            lat: position.coords.latitude,
                            lng: position.coords.longitude
                        }
                        this.setMarker();
                    });
                } else {
                    alert('Please allow location tracking');
                }
            }
        },
        setMarker() {
            var myLatlng = new google.maps.LatLng(this.pos.lat, this.pos.lng);
            this.geocoder = new google.maps.Geocoder();

            var myOptions = {
                zoom: 10,
                center: myLatlng
            }

            this.marker = new google.maps.Marker({
                position: myLatlng,
                map: this.map,
                draggable: true
            });

            this.getCurrentPosition({latLng: myLatlng});
            this.setDragEvent();
            this.setClickEvent();
        },
        setDragEvent() {
            this.dragListener = this.marker.addListener('dragend', (event) =>  {
                this.getCurrentPosition(event);
            });
        },
        setClickEvent() {
            this.clickListener = google.maps.event.addListener(this.map, 'click', (event) => {
                this.getCurrentPosition(event);
            });
        },
        getCurrentPosition(event) {
            this.geocoder.geocode({
                'latLng': event.latLng
            }, (results, status) => {
                if (status == google.maps.GeocoderStatus.OK) {
                    if (results[0]) {
                        this.start = results[0].formatted_address;
                        this.originPlaceId = results[0].place_id;
                        this.directionRoute();
                    }
                }
            });
        },
        getDirection(isStart) {
            if (!this.directionsDisplay || !this.directionsService) {
                this.directionsService = new google.maps.DirectionsService;
                this.directionsDisplay = new google.maps.DirectionsRenderer;
                this.directionsDisplay.setMap(this.map);
            }

            if (isStart) {
                var start = document.getElementById('start');
                var startAutocomplete = new google.maps.places.Autocomplete(start);
                startAutocomplete.setFields(['place_id']);
                this.AutocompleteListener(startAutocomplete, 'ORIG');
            } else {   
                var end = document.getElementById('end');
                var endAutocomplete = new google.maps.places.Autocomplete(end);
                endAutocomplete.setFields(['place_id']);
                this.AutocompleteListener(endAutocomplete, 'END');
            }
        },
        AutocompleteListener(autocomplete, mode) {
            autocomplete.bindTo('bounds', this.map);

            autocomplete.addListener('place_changed', (response) => {
                var start = document.getElementById('start').value;
                var end = document.getElementById('end').value;

                let place = autocomplete.getPlace();

                if (!place.place_id) {
                    window.alert('Please select an option from the dropdown list.');
                    return;
                }
                if (mode === 'ORIG') {
                    this.originPlaceId = place.place_id;
                    this.start = start;
                } else {
                    this.destinationPlaceId = place.place_id;
                    this.end = end;
                }
                this.directionRoute();
            });
        },
        directionRoute() {
            if (!this.originPlaceId || !this.destinationPlaceId) {
                return;
            }

            this.directionsService.route({
                origin: {'placeId': this.originPlaceId},
                destination: {'placeId': this.destinationPlaceId},
                travelMode: 'DRIVING'
            },
            (response, status) => {
                if (status === 'OK') {
                    this.directionsDisplay.setDirections(response);
                    this.directionInfo = response.routes[0];
                    this.directionSteps = response.routes[0].legs[0].steps;
                } else {
                    window.alert('Directions request failed due to ' + status);
                }
            });
        },
        hidePanel() {
            if (this.geoPermission) {
                this.marker.setMap(null);
                google.maps.event.removeListener(this.clickListener);
            }

            if (this.directionsDisplay) {
                this.directionsDisplay.setMap(null);
            }
            
            this.$emit('hidePanel');
        },
        directionStart: debounce(function (val) {
            this.getDirection(true);
        }, 500),
        directionEnd: debounce(function (val) {
            this.getDirection(false);
        }, 500)
    },
    mounted() {
        this.init();
    }
}
</script>
<style lang="sass" scoped>
    .direction-container {
        position: absolute;
        left: 0;
        top: 0;
        bottom: 0;
        width: 350px;
        background: #edf1f4;
        box-shadow: #b5b5b5 5px 5px 10px;
        display: flex;
        flex-direction: column;

        .close-btn {
            position: absolute;
            right: -30px;
            top: 15px;
            height: 40px;
            width: 40px;
            padding: 0;
            box-shadow: none;
            background: #edf1f4;
        }

        input {
            padding-left: 5px;
        }
    }
</style>
