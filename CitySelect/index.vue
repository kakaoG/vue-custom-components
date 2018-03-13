<template>
  <div class="city-select">
    <select class="provinces" v-model="addressData.province">
      <option value="-1">--请选择--</option>
      <option v-for="item in province" :value="item">{{item.name}}</option>
    </select>
    <select class="city" v-model="addressData.city" v-if="!isSpecialCity">
      <option value="-1">--请选择--</option>
      <option v-for="item in city" :value="item">{{item.name}}</option>
    </select>
    <select class="area" v-model="addressData.area" v-if="!isZxCity && !isSpecialCity">
      <option value="-1">--请选择--</option>
      <option v-for="item in area" :value="item">{{item.name}}</option>
    </select>
  </div>
</template>

<script>
  import locationData from 'regionalism-code';

  const Province = {}; // 省（自治区、直辖市、特别行政区）
  const locationCode = Object.keys(locationData);
  locationCode.forEach((key) => {
    // 省
    if (key.match(/^\d{2}0000$/)) {
      Province[key] = locationData[key];
    }
  });
  // const Area = {}; // 县（市辖区、县级市、旗)
  function getCitys(provinceCode) {
    const citys = {};
    const area = {};
    let isMunicipality = true; // 是否是自治区、直辖市、特别行政区
    locationCode.forEach((key) => {
      const cityCode = key - provinceCode;
      if (cityCode > 0 && cityCode < 1e4) {
        // 存在 xxxx00 的为非直辖市
        if (/00$/.test(cityCode)) {
          isMunicipality = false;
          citys[key] = locationData[key];
        } else if (/^11|12|31|50/.test(key)) { // 4个直辖市
          area[key] = locationData[key];
        } else if (!locationData[Math.floor(key / 100) * 100]) {
          citys[key] = locationData[key];
        }
      }
    });
    return {
      isMunicipality,
      values: isMunicipality ? area : citys,
    };
  }
  function getAreas(cityCode) {
    const area = {};
    const avalidReg = cityCode.substr(0, 4);
    const avalidCode = locationCode.filter(key => new RegExp(`^${avalidReg}`).test(key));
    const cityKey = `${avalidReg}00`;
    const hasValidArea = avalidCode.find(item => item === cityKey);
    avalidCode.forEach((key) => {
      const areaCode = key - cityCode;
      if (areaCode > 0 && areaCode < 1e2 && hasValidArea) {
        area[key] = locationData[key];
      }
    });
    return area;
  }

  export default {
    data () {
      const addressData = {
        province: -1,
        city: -1,
        area: -1,
      };
      let [province, city, area] = [[], [], []];
      Object.entries(Province).forEach((item) => {
        province.push({
          name: item[1],
          code: item[0]
        })
      });
      return {
        province,
        city,
        area,
        isZxCity: false,
        isSpecialCity: false,
        isShowDef: false,
        cityArr: [],
        areaArr: [],
        addressData,
      }
    },
    props: ['defData'],
    methods: {
      dataChange() {
        this.$emit('getAddressData', this.addressData, this.isSpecialCity);
      }
    },
    watch: {
      'addressData.province': {
        handler: function (n) {
          if (n.toString() === '-1') {
            this.addressData.city = -1;
            this.city = [];
          } else {
            const city = getCitys(n.code);
            this.city = [];
            if (city && (Object.keys(city.values).length>0)) {
              Object.entries(city.values).forEach((item) => {
                this.city.push({
                  name: item[1],
                  code: item[0]
                })
              });
              if (!this.isShowDef) this.addressData.city = this.city[0];
            }
            this.isSpecialCity = !(this.city.length>0);
          }
          this.dataChange();
        },
        deep: true
      },
      'addressData.city': {
        handler: function (n) {
          if (n) {
            if (n.toString() === '-1') {
              this.addressData.area = -1;
              this.area = [];
            } else {
              const area = getAreas(n.code);
              this.area = [];
              Object.entries(area).forEach((item) => {
                this.area.push({
                  name: item[1],
                  code: item[0]
                })
              });
              this.isZxCity = !(this.area.length>0);
              if (!this.isShowDef) this.addressData.area = this.area[0];
              this.isShowDef = false;
            }
            this.dataChange();
          }
        },
        deep: true
      },
      'addressData.area': {
        handler: function (n) {
          this.dataChange();
        },
        deep: true,
      },
      defData: function (n) {
        if (Object.getOwnPropertyNames(n).length>0) {
          this.addressData = n;
          if (n.province !== -1) this.isShowDef = true;
        }
      }
    },
  }
</script>

<style type="text/less" lang="less" scoped>
  .city-select {
    select {
      height: 25px;
      border: 1px solid #ddd;
      margin-right: 8px;
    }
    .province {
      min-width: 120px;
    }
    .city, .area {
      min-width: 60px;
    }
  }
</style>