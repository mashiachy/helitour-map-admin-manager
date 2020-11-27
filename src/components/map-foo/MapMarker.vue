<script>
import { MAP_MARKER_CONFIG, MAP_ELEMENT_MIXIN } from '@/constants';

export default {
  name: 'MapMarker',

  mixins: [ MAP_ELEMENT_MIXIN ],

  props: {
    latLng: {
      type: Object,
      default: () => null
    },
    pulse: {
      type: Boolean,
      default: false
    },
    size: {
      type: Number,
      default: 10
    }
  },

  watch: {
    pulse (v) {
      v ? this.startAnimate() : this.stopAnimate()
    },
    latLng (v) {
      this.marker.setPosition(v)
    },
    size (v) {
      this.stopAnimate()
      const s = this.marker.getIcon()
      s.scale = v
      this.marker.setIcon(s)
    }
  },

  data () {
    return {
      marker: null,
      animateInterval: null,
      i: 1,
      j: 0
    }
  },

  created () {
    this.marker = new this.google.maps.Marker({
      ...MAP_MARKER_CONFIG(this.google, this.size),
      map: this.map,
      position: this.latLng ? this.latLng : this.map.getCenter()
    })
  },

  beforeDestroy () {
    this.marker.setMap(null)
  },

  methods: {
    startAnimate () {
      this.setDefaultScale()
      this.animateInterval = setInterval(this.animateHandler, 10)
    },
    stopAnimate () {
      this.animateInterval ? clearInterval(this.animateInterval) : null
      this.animateInterval = null
      this.setDefaultScale()
    },
    animateHandler () {
      const s = this.marker.getIcon();
      this.j += this.i
      s.scale = s.scale + this.i * 0.1
      this.marker.setIcon(s)
      if (this.j >= 100) this.i = -1
      if (this.j <= 0) this.i = 1
    },
    setDefaultScale () {
      this.i = 1
      this.j = 0
      const s = this.marker.getIcon()
      s.scale = 10
      this.marker.setIcon(s)
    }
  }
}
</script>