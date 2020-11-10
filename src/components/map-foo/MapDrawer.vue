<script>
import { MAP_ELEMENT_MIXIN, DEFAULT_DRAGGABLE_CURSOR, MAP_POLYLINE_CONFIG, MAP_POLYGON_CONFIG, EARTH_RADIUS } from '@/constants';
import { eventBus } from '@/main';
import smooth from 'chaikin-smooth'

export default {
  name: 'MapDrawer',

  mixins: [ MAP_ELEMENT_MIXIN ],

  props: [ 'mapEl' ],

  data () {
    return {
      workMode: null,
      mode: null,
      overlay: null,
      polyLine: null,
      parcelleHeig: null,
      mousePressed: null,
      listenerMouseDown: null,
      listenerMouseUp: null,
      listenerMouseMove: null,
      polygon: null
    }
  },

  created () {
    eventBus.$on('drawerOn', this.modeOn)
    eventBus.$on('drawerOff', this.modeOff)
  },

  beforeDestroy () {
    eventBus.$off('drawerOn', this.modeOn)
    eventBus.$off('drawerOff', this.modeOff)
  },

  methods: {
    processDraw ({ latLng }) {
      if (!this.mousePressed) return
      this.polyLine.getPath().push(latLng)
      this.parcelleHeig.push(new this.google.maps.LatLng(latLng.lat(), latLng.lng()))
    },
    startDraw () {
      this.mousePressed = true;
      this.polyLine.setPath(this.parcelleHeig)
      this.polyLine.setMap(this.map)
    },
    stopDraw () {
      this.mousePressed = false
      let polygon = new this.google.maps.Polygon(MAP_POLYGON_CONFIG)
      polygon.setPath(this.parcelleHeig)
      polygon.douglasPeucker(360.0 / (2.0 * Math.PI * EARTH_RADIUS))
      const path = smooth(smooth(polygon.getPath().i.map(({ lat, lng }) => [ lat(), lng() ])))
        .map(([ lat, lng ]) => ({ lat, lng }))
      eventBus.$emit('drawn', path)
      polygon = null

      eventBus.$emit('drawerOff')
    },
    modeOn (mode) {
      this.mode = true
      this.workMode = mode
      if (this.workMode === 0) {
        this.parcelleHeig = []
        this.mousePressed = false
        this.overlay = new this.google.maps.OverlayView()
        this.overlay.draw = function () {}
        this.overlay.onAdd = function () {}
        this.overlay.onRemove = function () {}
        this.overlay.setMap(this.map)
        this.polyLine = new this.google.maps.Polyline(MAP_POLYLINE_CONFIG)
        this.map.setOptions({
          draggable: false,
          draggableCursor: 'url("pencil-cursor.png"), auto'
        })
        this.listenersOn()
      } else {
        // const { clickable, ...polygonConfig } = MAP_POLYGON_CONFIG
        const [ lat, lng ] = [ this.map.getCenter().lat(), this.map.getCenter().lng() ]
        this.polygon = new this.google.maps.Polygon({
          ...MAP_POLYGON_CONFIG,
          clickable: true,
          editable: true
        })
        this.polygon.setMap(this.map)
        this.polygon.setPath([ { lat, lng }, { lat, lng: lng+0.01 }, { lat: lat+0.01, lng: lng+0.01 } ])
        window.addEventListener('keypress', this.enterListener)
      }
    },
    enterListener ({ key }) {
      if (key === 'Enter') {
        this.modeOff()
      }
    },
    modeOff () {
      if (this.workMode === 0) {
        this.overlay.setMap(null)
        this.overlay = null
        this.map.setOptions({
          draggable: true,
          draggableCursor: DEFAULT_DRAGGABLE_CURSOR
        })
        this.polyLine.setMap(null)
        this.polyLine = null
        this.listenersOff()
        this.mode = false
      } else {
        if (this.polygon) {
          const path = this.polygon.getPath().i
          eventBus.$emit('drawn', path)
          this.polygon.setMap(null)
          this.polygon = null
          window.removeEventListener('keypress', this.enterListener)
        }
      }
    },
    listenersOn () {
      this.listenerMouseDown = this.mapEl.addEventListener('mousedown', this.startDraw)
      this.listenerMouseUp = this.mapEl.addEventListener('mouseup', this.stopDraw)
      this.listenerMouseMove = this.google.maps.event.addListener(this.map, 'mousemove', this.processDraw)
    },
    listenersOff () {
      if (this.mode) {
        this.listenerMouseDown = this.mapEl.removeEventListener('mousedown', this.startDraw)
        this.listenerMouseUp = this.mapEl.removeEventListener('mouseup', this.stopDraw)
        this.google.maps.event.removeListener(this.listenerMouseMove)
        this.listenerMouseMove = null
      }
    }
  }
}
</script>