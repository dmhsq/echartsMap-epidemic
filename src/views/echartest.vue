<template>
  <div>
    <div
      style="background-color: #42b983;width: 1010px;height: 810px;"
      v-loading="isLoading"
    >
      <div
        style="float: left;position: relative;top:50px;left: 250px;z-index: 5"
        v-if="!isChina"
      >
        <el-button type="success" @click="getworld">刷新</el-button>
        <el-button type="info" @click="setEc(0)">现存确诊</el-button>
        <el-button type="info" @click="setEc(1)">总确诊</el-button>
        <el-button type="info" @click="setEc(2)">新增确诊</el-button>
        <el-button type="info" @click="setEcs(check)">中国</el-button>
      </div>
      <div
        style="float: left;position: relative;left: 250px;z-index: 5"
        v-if="isChina"
      >
        <h3>上次更新时间{{ lastTime }}</h3>
        <el-button type="success" @click="getSd">刷新</el-button>
        <el-button type="info" @click="setEcs(0)">现存确诊</el-button>
        <el-button type="info" @click="setEcs(1)">总确诊</el-button>
        <el-button type="info" @click="setEcs(2)">新增确诊</el-button>
        <el-button type="info" @click="setEc(check)">全球</el-button>
      </div>
      <div style="width: 1000px;height: 800px;" ref="chart"></div>
    </div>
  </div>
</template>

<script>
import echarts from "echarts";
import china from "echarts/map/json/china.json";
import world from "echarts/map/json/world.json";
// import lodash from 'lodash'
import axios from "axios";
export default {
  name: "echartest",
  data() {
    return {
      chinaTotal: 0,
      chinaAdd: 0,
      chinaNow: 0,
      datas: [],
      worldData: [],
      lastTime: "",
      check: 0,
      isLoading: false,
      isChina: false,
      worldNames: [],
      worldValue: []
    };
  },
  mounted() {
    this.isLoading = true;
    this.getSd();
    this.getworld();
  },
  methods: {
    test() {
      //翻译
      return axios.get('/fanyi.json');
    },
    getworld() {
      this.isLoading = true;
      axios.post("/aki/ranklist").then(res => {
        let data = res.data.data;
        let names = [];
        data.forEach(item => {
          names.push(item.name);
        });
        this.worldData = data;
        this.test().then(res => {
          console.log(res.data.dst);
          let ss = res.data.dst;
          console.log(ss);
          let namess = ('"' + ss.replace(/, /g, '","') + '"').split(","); //处理翻译的名字
          this.worldNames = namess;

          this.setEc(this.check);
          this.isLoading = false;
        });
      });
    },
    getSd() {
      //全国疫情数据获取
      this.isLoading = true;
      axios.post(`/api/getDisease.html`).then(res => {
        let data = JSON.parse(res.data.data);
        console.log(data);
        let dss = data.areaTree[0].children;
        this.datas = dss;
        this.chinaTotal = data.chinaTotal.confirm;
        this.chinaAdd = data.chinaAdd.confirm;
        this.chinaNow = data.chinaTotal.nowConfirm;
        this.lastTime = data.lastUpdateTime;
        this.isLoading = false;
      });
    },
    setEcs(check) {
      //国内疫情地图绘制
      let min = 0;
      let max = 100;
      let title = " 地图";
      let data = [];
      let names = [];
      let values = [];
      this.check = check;
      if (check === 0) {
        title = "国内现存确诊地图";
        this.datas.forEach((item, index) => {
          names.push(item.name);
          values.push(item.total.confirm - item.total.dead - item.total.heal);
          data.push({ name: names[index], value: values[index] });
        });
        min = 0;
        max = 100;
      }
      if (check === 1) {
        title = "国内总确诊地图";
        this.datas.forEach((item, index) => {
          names.push(item.name);
          values.push(item.total.confirm);
          data.push({ name: names[index], value: values[index] });
        });
        min = 0;
        max = 3000;
      }
      if (check === 2) {
        title = "国内新增确诊地图";
        this.datas.forEach((item, index) => {
          names.push(item.name);
          values.push(item.today.confirm);
          data.push({ name: names[index], value: values[index] });
        });
        min = 0;
        max = 5;
      }
      echarts.registerMap("china", china);
      let myChart = echarts.init(this.$refs.chart);
      myChart.clear();
      let option = {
        title: {
          text: title,
          left: "center"
        },
        tooltip: {
          trigger: "item",
          showDelay: 0,
          transitionDuration: 0.2
        },
        visualMap: {
          left: "right",
          min: min,
          max: max,
          inRange: {
            //颜色梯度
            color: [
              "#f9f9fa",
              // "#bfbfc0",
              // "#74add1",
              // "#abd9e9",
              // "#e0f3f8",
              // "#ffffbf",
              "#fee090",
              "#fdae61",
              "#f46d43",
              "#d73027",
              "#a50026"
            ]
          },
          text: ["High", "Low"], // 文本，默认为数值文本
          calculable: true
        },
        toolbox: {
          show: true,
          //orient: 'vertical',
          left: "left",
          top: "top",
          feature: {
            dataView: { readOnly: false },
            restore: {},
            saveAsImage: {}
          }
        },
        series: [
          {
            name: "数据",
            type: "map",
            roam: true,
            map: "china",
            emphasis: {
              label: {
                show: true
              }
            },
            itemStyle: {
              normal: {
                label: {
                  show: true
                }
              }
            },
            // 文本位置修正

            data: data
          }
        ]
      };
      myChart.setOption(option);
      this.isChina = true;
      myChart.on("click", function(params) {
        console.log(params);
      });
    },
    setEc(check) {
      //世界疫情地图绘制
      let datas = [];
      let title = "";
      this.isChina = false;
      this.check = check;
      let min = 0;
      let max = 100;
      echarts.registerMap("world", world);
      let values = [];
      if (check === 0) {
        this.worldData.forEach((item, index) => {
          values.push(item.nowConfirm);
          let sss = this.worldNames[index];
          sss = sss.substring(1, sss.length - 1);
          if (sss === "USA") {
            sss = "United States";
          }
          datas.push({
            name: sss,
            value: values[index]
          });
        });
        datas.push({ name: "China", value: this.chinaNow });
        title = "世界疫情现存地图";
        min = 1000;
        max = 100000;
      }
      if (check === 1) {
        this.worldData.forEach((item, index) => {
          values.push(item.confirm);
          let sss = this.worldNames[index];
          sss = sss.substring(1, sss.length - 1);
          if (sss === "USA") {
            sss = "United States";
          }
          datas.push({
            name: sss,
            value: values[index]
          });
        });
        datas.push({ name: "China", value: this.chinaTotal });
        title = "世界疫情总确诊地图";
        min = 100000;
        max = 1000000;
      }
      if (check === 2) {
        this.worldData.forEach((item, index) => {
          values.push(item.confirmAdd);
          let sss = this.worldNames[index];
          sss = sss.substring(1, sss.length - 1);
          if (sss === "USA") {
            sss = "United States"; //可以删除，测试百度翻译时使用
          }
          datas.push({
            name: sss,
            value: values[index]
          });
        });
        datas.push({ name: "China", value: this.chinaAdd });
        title = "世界疫情新增确诊地图";
        min = 10;
        max = 1000;
      }

      let myChart = echarts.init(this.$refs.chart);
      myChart.clear();
      let option = {
        title: {
          text: title,
          left: "center"
        },
        tooltip: {
          trigger: "item",
          showDelay: 0,
          transitionDuration: 0.2
        },
        visualMap: {
          left: "right",
          min: min,
          max: max,
          inRange: {
            //颜色梯度
            color: [
              "#f9f9fa",
              // "#bfbfc0",
              // "#74add1",
              // "#abd9e9",
              "#e0f3f8",
              "#ffffbf",
              "#fee090",
              "#fdae61",
              "#f46d43",
              "#d73027",
              "#a50026"
            ]
          },
          text: ["High", "Low"], // 文本，默认为数值文本
          calculable: true
        },
        toolbox: {
          show: true,
          //orient: 'vertical',
          left: "left",
          top: "top",
          feature: {
            dataView: { readOnly: false },
            restore: {},
            saveAsImage: {}
          }
        },
        series: [
          {
            name: "数据",
            type: "map",
            roam: true,
            map: "world",
            emphasis: {
              label: {
                show: true
              }
            },
            itemStyle: {
              normal: {
                label: {
                  show: false
                }
              }
            },
            // 文本位置修正

            data: datas
          }
        ]
      };
      myChart.setOption(option);
      let vm = this;
      myChart.on("click", function(params) {
        console.log(params);
        if (params.name === "China") {
          console.log(1111);
          vm.setEcs(vm.check);
        }
      });
    }
    // getD() {//测试用方法，不用管
    //   axios
    //     .get("https://arena.360.cn/api/service/data/ncov-live-3")
    //     .then(res => {
    //       console.log(res.data.data.detail);
    //       let datass = res.data.data.detail;
    //       let d = [];
    //       let i = 0;
    //       let names = [];
    //       let values = [];
    //       datass.forEach(item => {
    //         names.push(item.city);
    //         values.push(item.diagnosed);
    //         d.push({ name: names[i], value: values[i] });
    //         i++;
    //       });
    //       console.log(d);
    //       var mydata = [
    //         { name: "天津", value: "100" },
    //         { name: "北京", value: "1000" },
    //         { name: "上海", value: "1000" },
    //         { name: "重庆", value: "1000" },
    //         { name: "河北", value: "1000" },
    //         { name: "河南", value: "1000" },
    //         { name: "云南", value: "1000" },
    //         { name: "辽宁", value: "1000" },
    //         { name: "黑龙江", value: "1000" },
    //         { name: "湖南", value: "1000" },
    //         { name: "安徽", value: "1000" },
    //         { name: "山东", value: "1000" },
    //         { name: "新疆", value: "1000" },
    //         { name: "江苏", value: "1000" },
    //         { name: "浙江", value: "1000" },
    //         { name: "江西", value: "1000" },
    //         { name: "湖北", value: "1000" },
    //         { name: "广西", value: "1000" },
    //         { name: "甘肃", value: "1000" },
    //         { name: "山西", value: "1000" },
    //         { name: "内蒙古", value: "1000" },
    //         { name: "陕西", value: "1000" },
    //         { name: "吉林", value: "1000" },
    //         { name: "福建", value: "1000" },
    //         { name: "贵州", value: "1000" },
    //         { name: "广东", value: "1000" },
    //         { name: "青海", value: "1000" },
    //         { name: "西藏", value: "1000" },
    //         { name: "四川", value: "1000" },
    //         { name: "宁夏", value: "1000" },
    //         { name: "海南", value: "1000" },
    //         { name: "台湾", value: "1000" },
    //         { name: "香港", value: "1000" },
    //         { name: "澳门", value: "1000" }
    //       ];
    //       mydata.forEach(item => {
    //         d.push({ name: item.name, value: item.value });
    //       });
    //       console.log(mydata);
    //       this.setEc(d);
    //     });
    // }
  }
};
</script>

<style scoped></style>
