![](/img/Screenshot 2019-04-22 at 1.23.47 PM.png "Philadelphia's Magic Gardens")
![](/img/Screenshot 2019-04-22 at 1.24.13 PM.png "Philadelphia's Magic Gardens")

# Google Maps Direction

This is a direction vue prop to be used with Google Maps Javascript API. It implements Autocomplete, Route mapping, and drag marker to specify starting point.

## Requirements

To use this vue component, these are the requirements:

1. Google Maps API with Places API
2. Location tracking enable on browser

## Installation

Simply use it as a prop by copying into a new file, and pass required props as stated below.

## Usage

1. Create a new file, and give a name with .vue extension. For example **Direction.vue**
2. Import the newly created file as component into parent file. For example

```
import DirectionPanel from "@/components/maps/Direction"
```

3. Create new DirectionPanel tag and pass the props

```
<DirectionPanel 
    v-if="showPanel"            <!-- Control to show or hide the panel -->
    :map="map"                  <!-- Google Maps instance. Required -->
    :position="myLatLng"        <!-- Current user location. Required -->
    @hidePanel="hidePanel"/>    <!-- Method to cater hide event -->


props should be as follows:

1.  this.map = new google.maps.Map(document.querySelector("#map"), mapOptions)

2.  this.myLatLng = {
        lat: latitude,
        lng: longitude
    }
```