<template>
  <div class="bg-teal-600 flex flex-col h-screen justify-center items-center space-y-4">
    <div class="bg-yellow-500 text-center bg-opacity-100 w-10/12 p-4 border-yellow-400 border-4">
      <p class="text-white font-bold text-2xl">お店が見つかるまで</p>
      <p class="text-white font-bold text-2xl">あと{{ this.checkpoint_count }}つ!</p>
    </div>
    <div class="flex flex-col justify-center items-center space-y-1 w-full">
      <p class="text-white text-xl font-bold">＼ ここを見つけてみよう ／</p>
      <div class="h-0 w-0" id="gmap"></div>
      <div class="h-[50vh] w-10/12" id="gpano"></div>
    </div>

    <div class="flex flex-row justify-center items-center space-x-5 w-full">
      <button
        v-if="isGoal"
        class="
          bg-yellow-500
          hover:bg-yellow-400
          text-white
          font-bold
          text-xl
          py-10
          px-4
          rounded-full
          border-yellow-400 border-4
          focus:shadow-outline
        "
        @click="to_goalPage()"
      >
        着いた！
      </button>
      <button
        v-else
        class="
          bg-yellow-500
          hover:bg-yellow-400
          text-white
          font-bold
          text-xl
          py-10
          px-4
          rounded-full
          border-yellow-400 border-4
          focus:shadow-outline
        "
        @click="getNextPosition()"
      >
        見つけた
      </button>
    </div>

    <!-- 確認用　距離は使えそう -->
    <!-- <div class="bg-white txt-margin text-center">
      <p>
        現在の緯度：<span>{{ this.latitude }}</span
        ><span>度</span>
      </p>
      <p>
        現在の経度：<span>{{ this.longitude }}</span
        ><span>度</span>
      </p>
      <p>
        目的地まで：<span>{{ this.distance_to_ckpt }}</span
        ><span>m</span>
      </p>
      <p>
        目的地までの角度：<span>{{ this.angle_to_ckpt }}</span
        ><span>度</span>
      </p>
    </div> -->
  </div>
</template>
<script>
export default {
  head() {},
  layout: 'default',
  components: {},
  middleware: [],
  data() {
    return {
      isGoal: false,
      latitude: 0,
      longitude: 0,
      //（あっきーから）座標を入れるリスト coordinates
      coordinates: [
        { lat: 34.3120297, lng: 135.5948249 },
        { lat: 35.1790435, lng: 136.8768059 },
        { lat: 35.182609, lng: 136.927285 },
      ],
      checkpoint_count: 0,
      distance_to_ckpt: 0,
      angle_to_ckpt: 0,
    }
  },
  watch: {
    latitude: {
      handler: function (newVal, oldVal) {
        console.log('change_position', newVal)
      },
    },
    longitude: {
      handler: function (newVal, oldVal) {
        console.log('change_position', newVal)
      },
    },
  },
  computed: {
    google() {
      return window.google
    },
  },
  created() {},
  beforeMount() {},
  mounted() {
    //TODO:ここであっきーの関数でcoordinatesに値代入する
    this.init()
  },
  beforeDestroy() {},
  methods: {
    init() {
      //初期ストリートビュー準備
      this.checkpoint_count = this.coordinates.length
      this.setStreetView(this.checkpoint_count)
      //現在地習得
      navigator.geolocation.watchPosition(
        (position) => {
          this.updatedCount++
          this.isWatching = true
          const location = position.coords
          this.latitude = location.latitude
          this.longitude = location.longitude
          this.setDistanceToCkpt(
            { lat: location.latitude, lng: location.longitude },
            this.coordinates[this.checkpoint_count - 1]
          )
          this.setAngleToCkpt({ lat: this.latitude, lng: this.longitude }, this.coordinates[this.checkpoint_count - 1])
        },
        (err) => {
          this.isWatching = false
          console.log(err)
        },
        { enableHighAccuracy: true }
      )
    },
    setStreetView(checkpoint_count) {
      //座標リストからstreetviewセットする。
      var coord_index = checkpoint_count - 1
      const fenway = this.coordinates[coord_index]
      const google = this.google
      const map = new google.maps.Map(document.getElementById('gmap'), {
        center: fenway,
        zoom: 14,
      })
      const panorama = new google.maps.StreetViewPanorama(document.getElementById('gpano'), {
        position: fenway,
        pov: {
          heading: 34,
          pitch: 10,
        },
        addressControl: false,
        fullscreenControl: false,
        linksControl: false,
        panControl: false,
        enableCloseButton: false,
      })
      map.setStreetView(panorama)
    },
    getNextPosition() {
      //次の座標に移動
      this.checkpoint_count = this.checkpoint_count - 1
      console.log('あと' + String(this.checkpoint_count) + 'つ')
      if (this.checkpoint_count == 1) {
        this.isGoal = true
      }
      this.setStreetView(this.checkpoint_count)
      this.setDistanceToCkpt({ lat: this.latitude, lng: this.longitude }, this.coordinates[this.checkpoint_count - 1])
      this.setAngleToCkpt({ lat: this.latitude, lng: this.longitude }, this.coordinates[this.checkpoint_count - 1])
    },
    to_goalPage() {
      // 店名＋アニメーション画面に遷移するためのボタン関数
      console.log('次のページへ')
      // TODO：下に最後のページのパス指定
      // this.$router.push('/')
    },
    setDistanceToCkpt(location1, location2) {
      // チェックポイントまでの直線距離を現在地から計算する。(m単位)
      const latitude1 = location1.lat
      const longitude1 = location1.lng
      const latitude2 = location2.lat
      const longitude2 = location2.lng
      const R = 6371 // km
      const diffLatitudeRadian = this.getRadian(latitude2 - latitude1)
      const diffLongitudeRadian = this.getRadian(longitude2 - longitude1)
      const latitudeRadian = this.getRadian(latitude1)
      const longitudeRadian = this.getRadian(latitude2)
      const a =
        Math.sin(diffLatitudeRadian / 2) * Math.sin(diffLatitudeRadian / 2) +
        Math.sin(diffLongitudeRadian / 2) *
          Math.sin(diffLongitudeRadian / 2) *
          Math.cos(latitudeRadian) *
          Math.cos(longitudeRadian)
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a))
      this.distance_to_ckpt = Math.round(R * c * 1000) //m
    },
    getRadian(value) {
      return (value * Math.PI) / 180
    },
    setAngleToCkpt(location_from, location_to) {
      // TODO？
      // ２点間の方角を求める (コンパスできたらいいな)
      // 北:0度、東:90度、南:180度、西:270度
      //　端末の角度とれる↓
      // https://developer.mozilla.org/ja/docs/Web/Events/Detecting_device_orientation
      var x1 = location_from.lat
      var y1 = location_from.lng
      var x2 = location_to.lat
      var y2 = location_to.lng
      var delta_x = x2 - x1
      var r = 6378.137
      var d
      d = Math.sin(y1) * Math.sin(y2) + Math.cos(y1) * Math.cos(y2) * Math.cos(delta_x)
      d = r * Math.acos(d)
      var fai
      var a, b
      a = Math.sin(delta_x)
      b = Math.cos(y1) * Math.tan(y2) - Math.sin(y1) * Math.cos(delta_x)
      fai = Math.atan2(b, a)
      if (fai >= 0) {
        this.angle_to_ckpt = Math.round((fai * 180.0) / Math.PI)
      } else {
        this.angle_to_ckpt = Math.round(((fai + 2 * Math.PI) * 180.0) / Math.PI)
      }
    },
  },
}
</script>
<style scoped>
</style>