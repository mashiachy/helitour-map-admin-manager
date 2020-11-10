<script>
import { MAP_ELEMENT_MIXIN, MAP_POLYGON_CONFIG } from '@/constants';
// import smooth from 'chaikin-smooth';

export default {
  name: 'MapPolygon',

  mixins: [ MAP_ELEMENT_MIXIN ],

  props: {
    pathV: {
      required: true
    }
  },

  data () {
    return {
      polygon: null
    }
  },

  watch: {
    pathV (v) {
      if ( !v ) return
      this.polygon.setPath(v)
      // this.polygon.douglasPeucker(360.0 / (2.0 * Math.PI * EARTH_RADIUS))
      // this.polygon.setPath(smooth(smooth(this.polygon.getPath().i.map(({ lat, lng }) => [ lat(), lng() ])))
      //   .map(([ lat, lng ]) => ({ lat, lng }))
      // )
    }
  },

  created () {
    this.polygon = new this.google.maps.Polygon(MAP_POLYGON_CONFIG)
    this.polygon.setMap(this.map)
  },

  beforeDestroy() {
    this.polygon.setMap(null)
    this.polygon = null
  }
}
</script>