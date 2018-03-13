<template>
  <Cascader  v-model="cityValue" :data="cityData.provinceArr" :load-data="loadData" @on-change="selectCity">
  </Cascader>
</template>

<script>
  import originData from './cityData';

  export default {
    data() {
      const provinceArr = [];
      const cityArr = [];
      const areaArr = [];
      Object.entries(originData).forEach((item) => {
        const code = item[0];
        const name = item[1];
        if (/0000$/.test(code)) {
          provinceArr.push({
            value: code,
            label: name,
            loading: false,
            children: [],
          });
        } else if (/^11|^12|^31|^50/.test(code)) {
          cityArr.push({
            value: code,
            label: name,
          });
        } else if (/00$/.test(code)) {
          cityArr.push({
            value: code,
            label: name,
            loading: false,
            children: [],
          });
        } else {
          areaArr.push({
            value: code,
            label: name,
          });
        }
      });
      return {
        cityData: {
          provinceArr,
          cityArr,
          areaArr,
        },
        cityValue: [],
      };
    },
    props: {
      cityDefValue: {
        type: Array,
        default: () => [],
      },
    },
    methods: {
      loadData(item, cb) {
        window.console.info(item);
        if (item.children && item.children.length === 0) {
          item.loading = true;
          if (/0000$/.test(item.value)) {
            this.cityData.cityArr.filter(c => item.value.slice(0, 2) === c.value.slice(0, 2))
              .forEach(b => item.children.push(b));
            item.loading = false;
          } else if (/00$/.test(item.value) || /^11|^12|^31|^50/.test(item.value)) {
            this.cityData.areaArr.filter(a => item.value.slice(0, 4) === a.value.slice(0, 4))
              .forEach((b) => {
                if (item.children) item.children.push(b);
              });
            item.loading = false;
          }
        }
        cb();
      },
      selectCity(value, selectedData) {
        this.$emit('on-select-city', selectedData);
      },
    },
    watch: {
      cityDefValue(n) {
        this.cityValue = n;
      },
    },
  };
</script>

<style>

</style>
