<template>
  <div class="homePage">
    <BusListForm
      :busList="busList"
      @getBusInfo="getBusInfo"
      @initMap="initMap()"
      @getBusOrder="getBusOrder"
    />
    <div id="map" ref="map"></div>
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
    }
  },
  methods: {
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
        lat: Number(getMark.Lat) + 1,
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
    // this.setMaker();
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
