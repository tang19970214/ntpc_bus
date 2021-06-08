<template>
  <div class="busListFormPage">
    <el-tabs type="border-card" class="busListFormPage__tollBar">
      <el-tab-pane label="任務管理">
        <el-button type="warning" plain @click="initMap()"
          >重新載入 google map</el-button
        >
        <div class="busListFormPage__table">
          <el-table
            :data="busList"
            height="300"
            stripe
            style="width: 100%"
            @row-click="getBusInfo"
            @selection-change="selectionChange"
            :show-overflow-tooltip="true"
            :stripe="false"
          >
            <el-table-column type="selection" width="55"></el-table-column>
            <el-table-column
              label="編號"
              prop="id"
              width="100px"
            ></el-table-column>
            <el-table-column
              label="車輛"
              prop="carNo"
              width="130px"
            ></el-table-column>
            <el-table-column
              label="駕駛員"
              prop="driverName"
              width="80px"
            ></el-table-column>
            <el-table-column
              label="所屬"
              prop="companyStation"
              width="180px"
            ></el-table-column>
            <el-table-column
              label="任務備註"
              prop="dutyDesc"
              width="150px"
            ></el-table-column>
            <el-table-column
              label="狀態"
              prop="dutyStatus"
              width="100px"
            ></el-table-column>
            <el-table-column
              label="所需時間"
              prop="time"
              width="100px"
            ></el-table-column>
          </el-table>
        </div>
      </el-tab-pane>
      <el-tab-pane label="地圖監控"></el-tab-pane>
    </el-tabs>
    <!-- <div class="busListFormPage__tollBar">
      <el-button type="warning" plain @click="initMap()"
        >重新載入 google map</el-button
      > -->
    <!-- <el-button type="primary" plain @click="showTollBar = !showTollBar">{{
        showTollBar ? "隱藏" : "顯示"
      }}</el-button> -->
  </div>
</template>

<script>
export default {
  props: {
    busList: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      showTollBar: true,
      busOrderList: [],
    }
  },
  methods: {
    getBusInfo(row) {
      this.$emit("getBusInfo", row)
    },
    initMap() {
      this.$emit("initMap")
    },
    selectionChange(val) {
      this.busOrderList = val
      this.$emit("getBusOrder", this.busOrderList)
    },
  },
}
</script>

<style lang="scss">
.busListFormPage {
  width: 100%;
  max-width: 920px;
  position: fixed;
  top: 4px;
  left: 4px;
  z-index: 10;
  background: white;
  border-radius: 8px;
  .el-tabs__content {
    padding: 0;
  }
  &__table {
    .el-table__header-wrapper .el-checkbox {
      display: none;
    }
    .el-table__row {
      cursor: pointer;
    }
  }
}
</style>
