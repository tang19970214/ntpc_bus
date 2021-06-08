<template>
  <div class="homePage">
    <BusListForm
      :busList="busList"
      @getBusInfo="getBusInfo"
      @initMap="initMap()"
      @getBusOrder="getBusOrder"
    />
    <div id="map" ref="map"></div>
    <div id="msg"></div>
  </div>
</template>

<script>
import BusListForm from "../components/BusListForm.vue"

export default {
  name: "Home",
  components: { BusListForm },
  data() {
    return {
      busList: [],
      cardList: [],
      busMarkList: [],
      map: document.getElementById("map"),
      // 存放目前開啟的訊息視窗
      infowindow: null,
      latMarker: 23.47316642202988,
      lonMarker: 120.29256820678711,
      Point1: null,
      Point2: null,
      duration: null,
    }
  },
  methods: {
    getDistanceTime() {
      let map
      let directionsService = new google.maps.DirectionsService()
      let directionsRenderer = new google.maps.DirectionsRenderer()

      directionsRenderer.setMap(map)

      const route = {
        origin: this.Point1,
        destination: this.Point2,
        travelMode: "DRIVING",
      }
      directionsService.route(route, (response, status) => {
        this.duration = response.routes[0].legs[0].duration.text
        // console.log("this.duration", this.duration)
        return
        if (status !== "OK") {
          window.alert("Directions request failed due to " + status)
          return
        } else {
          directionsRenderer.setDirections(response) // Add route to the map
          let directionsData = response.routes[0].legs[0] // Get data about the mapped route
          // console.log("directionsData", directionsData)
          if (!directionsData) {
            window.alert("Directions request failed")
            return
          } else {
            document.getElementById("msg").innerHTML +=
              // document.getElementsByClassName("markerPopover").innerHTML +=
              "距離 ： " +
              directionsData.distance.text +
              " (" +
              directionsData.duration.text +
              ")."
          }
        }
      })
    },
    //勾選巴士後更新 busMarkList
    getBusOrder(order) {
      if (order.length > 0) {
        // 取得所選車輛id
        let getBusIds = order.map((res) => res.carId)
        let getBusLocal = this.busMarkList.filter((resp) =>
          getBusIds.includes(Number(resp.BusID))
        )
        this.initMap(getBusLocal)
      } else {
        this.initMap(this.busMarkList)
      }
    },

    //定位巴士，顯示 dialog
    getBusInfo(row) {
      let getMark = this.busMarkList.filter((res) => res.BusID == row.carId)[0]
      this.map.panTo({
        lat: Number(getMark.Lat),
        lng: Number(getMark.Lon),
      })
      const marker = new google.maps.Marker({
        //原始中心點
        position: {
          lat: Number(getMark.Lat),
          lng: Number(getMark.Lon),
        },
        icon: getMark.BusID
          ? require("@/assets/images/busIcon@1x.png")
          : require("@/assets/images/carIcon@1x.png"),
        map: this.map,
      })
      // Point1 自定點位（目前是寫死狀態）
      this.Point1 = { lat: Number(this.latMarker), lng: Number(this.lonMarker) }
      // Point2 點選巴士的點位
      this.Point2 = { lat: Number(getMark.Lat), lng: Number(getMark.Lon) }
      this.getDistanceTime(this.Point1, this.Point2)
      let totalTime = this.duration
      // console.log("totalTime", totalTime)
      // 透過 InfoWindow 物件建構子建立新訊息視窗
      const infowindow = new google.maps.InfoWindow({
        // 設定想要顯示的內容
        content:
          `
          <div class="markerPopover">
            <p>駕駛員：` +
          row.driverName +
          `</p>
            <p>聯絡電話：` +
          row.driverPhone +
          `</p>
            <p>任務說明：` +
          row.dutyDesc +
          `</p>
            <p>車速：` +
          getMark.Speed +
          `</p>
            <p>車輛：` +
          row.carNo +
          `</p>
           <p>預估時間：` +
          totalTime +
          `</p>
          </div>
        `,
        // 設定訊息視窗最大寬度
        maxWidth: 200,
      })
      if (this.infowindow) {
        this.infowindow.close()
        infowindow.open(this.map, marker)
        this.infowindow = infowindow
      } else {
        infowindow.open(this.map, marker)
        this.infowindow = infowindow
      }
    },

    // 建立地圖
    initMap(getBusLocal) {
      this.map = new google.maps.Map(document.getElementById("map"), {
        center: {
          lat: Number(this.busMarkList[0].Lat),
          lng: Number(this.busMarkList[0].Lon),
        },
        zoom: 13,
        maxZoom: 20,
        minZoom: 3,
        streetViewControl: false,
        mapTypeControl: false,
        draggable: true,
      })

      this.setMarker(getBusLocal)
    },
    // 建立地標
    setMarker(getBusLocal) {
      // 將巴士及小車合併成一個array
      let allList = []
      if (!!getBusLocal) {
        allList = getBusLocal.concat(this.cardList)
      } else {
        allList = this.busMarkList.concat(this.cardList)
      }
      // 整理出所有車牌
      let getBusInfos = allList.map((res) => res.BusID)
      allList.forEach((location, idx) => {
        let getInfos = this.busList.filter(
          (res) => getBusInfos.includes(res.carId) || {}
        )
        const marker = new google.maps.Marker({
          //原始中心點
          position: {
            lat: Number(location.Lat),
            lng: Number(location.Lon),
          },
          icon: location.BusID
            ? require("@/assets/images/busIcon@1x.png")
            : require("@/assets/images/carIcon@1x.png"),
          map: this.map,
        })
        // console.log("marker", marker)
        // 透過 InfoWindow 物件建構子建立新訊息視窗
        const infowindow = new google.maps.InfoWindow({
          // 設定想要顯示的內容
          content:
            `
          <div class="markerPopover">
            <p>駕駛員：` +
            getInfos[idx]?.driverName +
            `</p>
            <p>聯絡電話：` +
            getInfos[idx]?.driverPhone +
            `</p>
            <p>任務說明：` +
            getInfos[idx]?.dutyDesc +
            `</p>
            <p>車速：` +
            location.Speed +
            `</p>
            <p>車輛：` +
            getInfos[idx]?.carNo +
            `</p>
            <p>車輛：` +
            getInfos[idx]?.carNo +
            `</p>
          </div>
        `,
          // 設定訊息視窗最大寬度
          maxWidth: 200,
        })

        // 綁定點擊事件監聽
        marker.addListener("click", () => {
          if (this.infowindow) {
            this.infowindow.close()
            infowindow.open(this.map, marker)
            this.infowindow = infowindow
          } else {
            infowindow.open(this.map, marker)
            this.infowindow = infowindow
          }
        })
      })
    },
    async getCarInfo() {
      await this.axios
        .get("http://care.1966.org.tw/api/api/DriverInfo/DeviceGpsViewModel")
        .then((res) => {
          this.cardList = res.data.response
        })
        .catch((error) => {
          console.log(error)
        })
    },
  },
  async mounted() {
    this.busList = this.$store.state.busList
    this.busMarkList = this.$store.state.busMarkList
    await this.getCarInfo()
    this.initMap()
  },
}
</script>

<style lang="scss">
.homePage {
  #map {
    width: 100vw;
    height: 100vh;
  }
}
</style>
