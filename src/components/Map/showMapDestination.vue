<template>
<div v-if="loaded" class="loadDiv">
    <div id="showMapDestination"></div>
</div>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex';

export default {
  props: ['loaded'],
  name: 'showMapDestination',
  test: null,
  components: {
  },
  data () {
    return {
      openMapPickup: false,
      center: [43.238475, 76.911361],
    }
  },
  computed: {
    ...mapGetters(['initialLocation', 'latlngInitial', 'destinationLocation']),
    ...mapGetters(['destination', 'latlngDestination'])

  },

  watch: {
      destination: function() {
        this.markerLocation.bindLabel(this.destination, 
        { 
            static: true, 
            offset: [-90, -60],
            textDirection: 'auto'
        })
        this.markerLocation.setLatLng(this.latlngDestination)
        this.map.setView(this.latlngDestination)
      },
  },
  mounted() {

    if(document.getElementById("showMapDestination")){
        // document.getElementById('showMapDestination').innerHTML = "<div id='map' style='width: 100%; height: 100%;'></div>";
        const DG = require('2gis-maps');
        
        const map = DG.map('showMapDestination', {
            'center': this.center,
            'zoom': 12,
            'zoomControl': false,
            'fullscreenControl': false,
        })

        DG.control.location({position: 'topright'}).addTo(map);
        
        const lat = map.getCenter().lat
        const lng = map.getCenter().lng
        this.map = map
        const iconLocation = DG.icon({
            iconUrl: require('../../assets/location.png'),
            iconSize: [30, 30],
        });

        const markerLocation = DG.marker(this.latlngDestination, 
            { icon: iconLocation})
            .addTo(this.map)
        this.markerLocation = markerLocation
        let markerLocationCoord = this.markerLocation.getLatLng()

        this.map.on('move', function (e) {
            const newLatLng = this.getCenter()
            markerLocation.setLatLng(newLatLng);
            markerLocationCoord =  markerLocation.getLatLng()
            markerLocation.unbindLabel()
        });

        this.map.on('moveend', DG.bind(onMoveend, this, true))

        // finds a new address when moveend
        function onMoveend(e) {
            markerLocation.bindLabel(this.destination ? this.destination : this.initialLocation, 
            { 
                static: true, 
                offset: [-90, -60],
                textDirection: 'auto'
            })
            this.findName()
        } 
        if (this.destination) {
            this.markerLocation.bindLabel(this.destination, 
            { 
                static: true, 
                offset: [-90, -60],
                textDirection: 'auto'
            })
            this.map.setView(this.latlngDestination, 17, {'animate': true, 'noMoveStart': true})
        } else {
            this.markerLocation.bindLabel(this.initialLocation, 
            { 
                static: true, 
                offset: [-90, -60],
                textDirection: 'auto'
            })
            this.map.setView(this.latlngInitial, 17, {'animate': true, 'noMoveStart': true})
        }
    }
  },
  updated() {
    if (this.destination) {
        this.markerLocation.bindLabel(this.destination, 
        { 
            static: true, 
            offset: [-90, -60],
            textDirection: 'auto'
        });
    }
    
  },
  methods: {
      ...mapMutations(['setDestination']),
      findName: function() {
          const url = `https://nominatim.openstreetmap.org/reverse?format=jsonv2&lat=${this.markerLocation.getLatLng().lat}&lon=${this.markerLocation.getLatLng().lng}`
          const proxyurl = "https://cors-anywhere.herokuapp.com/";
          fetch(url)
              .then(data => data.json())
              .then(location => {
                  const splittedAddress = location.display_name.split(',');
                  const address =
                      splittedAddress[1].trim() +
                      ' ' +
                      splittedAddress[0].trim() +
                      ', ' +
                      splittedAddress[2].trim();

                  const lat = this.markerLocation.getLatLng().lat
                  const lng = this.markerLocation.getLatLng().lng
                    if(this.destinationLocation.lat !== lat && this.destinationLocation.lng !== lng){
                        this.setDestination({address, lat, lng})
                    }
              })
       },   
       
    }
}
</script>

<style lang="scss">
.list {
  height: 370px;
}
.loadDiv{
    width: 100%;
    height: 100%;
}
#showMapDestination {
    width: 100%;
    height: 95%;
    position: absolute !important; 
    z-index: 9; 
}
.leaflet-marker-icon {
    position: absolute;
}

</style>
