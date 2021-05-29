<template>
  <div class="busListFormPage">
    <div class="busListFormPage__tollBar">
      <el-button type="warning" plain @click="initMap()">重新載入 google map</el-button>
      <el-button type="primary" plain @click="showTollBar = !showTollBar">{{showTollBar ? "隱藏" : "顯示"}}</el-button>
    </div>
    <div class="busListFormPage__table" v-if="showTollBar">
      <el-table :data="busList" height="300" stripe style="width: 100%" @row-click="getBusInfo">
        <el-table-column label="編號" prop="id" width="100px"></el-table-column>
        <el-table-column label="車輛" prop="carNo" width="130px"></el-table-column>
        <el-table-column label="駕員" prop="driverName" width="60px"></el-table-column>
        <el-table-column label="所屬" prop="companyStation" width="180px"></el-table-column>
        <el-table-column label="任務備註" prop="dutyDesc" width="150px"></el-table-column>
        <el-table-column label="狀態" prop="dutyStatus" width="100px"></el-table-column>
      </el-table>
    </div>
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
    };
  },
  methods: {
    getBusInfo(row) {
      this.$emit("getBusInfo", row);
    },
    initMap() {
      this.$emit("initMap");
    },
  },
};
</script>

<style lang="scss">
.busListFormPage {
  position: fixed;
  top: 4px;
  left: 4px;
  z-index: 10;
  background: white;
  border-radius: 8px;

  &__table {
    .el-table__row {
      cursor: pointer;
    }
  }
}
</style>