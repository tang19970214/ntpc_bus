<template>
  <div class="homePage">
    <BusListForm :busList="busList" @getBusInfo="getBusInfo" @initMap="initMap()" @getBusOrder="getBusOrder" />
    <div id="map" ref="map"></div>
  </div>
</template>

<script>
import BusListForm from "../components/BusListForm.vue";

export default {
  name: "Home",
  components: { BusListForm },
  data() {
    return {
      busList: [],
      cardList: [],
      busMarkList: [],
      map: document.getElementById("map"),
      test: [],
    };
  },
  methods: {
    //勾選巴士後更新 busMarkList
    getBusOrder(order) {
      if (order.length > 0) {
        // 取得所選車輛id
        let getBusIds = order.map((res) => res.carId);
        let getBusLocal = this.busMarkList.filter((resp) =>
          getBusIds.includes(Number(resp.BusID))
        );
        this.initMap(getBusLocal);
      } else {
        this.initMap(this.busMarkList);
        return;
        this.busMarkList = this.$store.state.busMarkList;
      }
    },

    //定位巴士，顯示 dialog
    getBusInfo(row) {
      let getMark = this.busMarkList.filter((res) => res.BusID == row.carId)[0];
      this.map.panTo({ lat: Number(getMark.Lat), lng: Number(getMark.Lon) });
    },
    initMap(getBusLocal) {
      this.map = new google.maps.Map(document.getElementById("map"), {
        center: {
          //原始中心點
          lat: Number(this.busMarkList[0].Lat),
          lng: Number(this.busMarkList[0].Lon),
        }, // 中心點座標, // 中心點座標
        zoom: 15, // 1-20，數字愈大，地圖愈細：1是世界地圖，20就會到街道
        maxZoom: 20,
        minZoom: 5,
        zoomControl: false,
        mapTypeControl: false,
        scaleControl: false,
        streetViewControl: false,
        rotateControl: false,
        fullscreenControl: false,
        // roadmap 顯示默認道路地圖視圖。
        // satellite 顯示 Google 地球衛星圖像。
        // hybrid 顯示正常和衛星視圖的混合。
        // terrain 顯示基於地形信息的物理地圖。
        mapTypeId: "roadmap",
        // 需要再前往修改，https://mapstyle.withgoogle.com/
      });
      let marker = this.setMaker(getBusLocal);
    },
    setMaker(getBusLocal) {
      // 將巴士及小車合併成一個array
      let allList = [];
      if (!!getBusLocal) {
        allList = getBusLocal.concat(this.cardList);
      } else {
        allList = this.busMarkList.concat(this.cardList);
      }
      console.log(allList);

      // busMarkList & cardList 合併跑回圈
      // let newList = [...this.busMarkList, ...this.cardList];
      // console.log("newList", newList)
      allList.forEach((location) => {
        // const busInfo = [];
        // console.log("location", location)
        // let icon = ""
        // if (location.carId) {
        //   icon = require("@/assets/images/busIcon@1x.png")
        // } else {
        //   icon = require("@/assets/images/carIcon@1x.png")
        // }
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
        });
        // 透過 InfoWindow 物件建構子建立新訊息視窗
      });
      return;
      this.getDialogInfo(busInfo);
      this.openDialogInfo(marker);
    },
    getDialogInfo(busInfo) {
      const infowindow = new google.maps.InfoWindow({
        // 設定想要顯示的內容
        content:
          `
          <div class="markerPopover">
            <p>駕駛員：` +
          busInfo.driverName +
          `</p>
            <p>聯絡電話：` +
          busInfo.driverPhone +
          `</p>
            <p>任務說明：` +
          busInfo.dutyDesc +
          `</p>
            <p>車速：` +
          location.Speed +
          `</p>
            <p>車輛：` +
          busInfo.carNo +
          `</p>
          </div>
        `,
        // 設定訊息視窗最大寬度
        maxWidth: 200,
      });
    },
    openDialogInfo(marker) {
      // 在地標上監聽點擊事件
      marker.addListener("click", () => {
        // 指定在哪個地圖和地標上開啟訊息視窗
        infowindow.open(this.map, marker);
      });
    },
    async getCarInfo() {
      await this.axios
        .get("http://care.1966.org.tw/api/api/DriverInfo/DeviceGpsViewModel")
        .then((res) => {
          this.cardList = res.data.response;
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
  async mounted() {
    this.busList = this.$store.state.busList;
    this.busMarkList = this.$store.state.busMarkList;
    await this.getCarInfo();
    this.initMap();
    // this.setMaker();
  },
};
</script>

<style lang="scss">
.homePage {
  #map {
    width: 100vw;
    height: 100vh;
  }
}
</style>
