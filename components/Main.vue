<template>
  <div>
    <div class="container">
      <h2>都道府県別、人口推移チェッカー</h2>
      <p>
        総人口、年少人口、生産年齢人口、老年人口の順でグラフが表示されます。
      </p>
      <select v-model="selected">
        <option disabled value="">Please select one</option>
        <option
          v-for="pref in resasResponseResults.prefdata"
          :key="`second-${pref.prefCode}`"
        >
          {{ pref.prefName }}
        </option>
      </select>

      <p>の人口推移</p>
      <div class="chart-container" id="chart-parent">
        <canvas class="graph" id="myChart0"></canvas>
        <canvas class="graph" id="myChart1"></canvas>
        <canvas class="graph" id="myChart2"></canvas>
        <canvas class="graph" id="myChart3"></canvas>
      </div>
      <p>「出典：RESAS（地域経済分析システム）」</p>
      <p>「RESAS（地域経済分析システム）を加工して作成」</p>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Chart from "chart.js";
require("dotenv").config({ debug: true });

export default {
  name: "Main",
  props: {
    msg: String,
  },
  data() {
    return {
      selected: "",
      getPrefectureUrl: process.env.VUE_APP_GET_PREFECTUREURL,
      getPopulationCompositionUrl: process.env.VUE_APP_GET_COMPOSITION_URL,
      headers: {
        "X-API-KEY": process.env.VUE_APP_X_API_KEY,
      },
      resasResponseResults: {
        prefdata: [],
        labels: [],
        data: [],
      },
    };
  },

  methods: {
    displayGraph: function (num, responseLabel) {
      let ctx = document.getElementById("myChart" + num).getContext("2d");
      ctx.canvas.parentNode.style = "";
      let displaySize = window.screen.width;
      if (displaySize < 640) {
        let width = String(displaySize * 0.9);
        ctx.canvas.parentNode.style.width = width + "px";
      } else if (displaySize > 960) {
        ctx.canvas.parentNode.style.width = "50%";
        ctx.canvas.parentNode.style.margin = "auto";
      }
      new Chart(ctx, {
        type: "line",
        data: {
          labels: this.resasResponseResults.labels,
          datasets: [
            {
              label: responseLabel,
              data: this.resasResponseResults.data,
              fill: false,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: true,
        },
      });
    },
    handleResize: function () {
      let self = this;
      let getPrefCode = this.resasResponseResults.prefdata.find((data) => {
        return data.prefName === this.selected;
      });
      let spacifiedPrefCode = getPrefCode["prefCode"];
      axios
        .get(this.getPopulationCompositionUrl + spacifiedPrefCode, {
          headers: { "X-API-KEY": process.env.VUE_APP_X_API_KEY },
        })
        .then(function (response) {
          for (let i = 0; i < response.data.result.data.length; i++) {
            for (let j = 0; j < response.data.result.data[i].data.length; j++) {
              self.resasResponseResults.labels.push(
                response.data.result.data[i].data[j]["year"]
              );
              self.resasResponseResults.data.push(
                response.data.result.data[i].data[j]["value"]
              );
            }
            self.resasResponseResults.responseLabel =
              response.data.result.data[i]["label"];
            self.displayGraph(i, self.resasResponseResults.responseLabel);
            self.resasResponseResults.labels = [];
            self.resasResponseResults.data = [];
            self.resasResponseResults.responseLabel = [];
          }
        })
        .catch(function (error) {
          console.log(error);
        });
    },
  },

  created() {
    let self = this;

    axios
      .get(self.getPrefectureUrl, {
        headers: { "X-API-KEY": process.env.VUE_APP_X_API_KEY },
      })
      .then(function (response) {
        console.log(response.data.result);
        for (var i = 0; i < response.data.result.length; i++) {
          self.resasResponseResults.prefdata.push(response.data.result[i]);
        }
      })
      .catch(function (error) {
        console.log(error);
      });
  },
  mounted: function () {
    window.addEventListener("resize", this.handleResize);
  },
  beforeDestroy: function () {
    window.removeEventListener("resize", this.handleResize);
  },
  watch: {
    selected: function () {
      let self = this;
      let getPrefCode = this.resasResponseResults.prefdata.find((data) => {
        return data.prefName === this.selected;
      });
      let spacifiedPrefCode = getPrefCode["prefCode"];

      axios
        .get(this.getPopulationCompositionUrl + spacifiedPrefCode, {
          headers: { "X-API-KEY": process.env.VUE_APP_X_API_KEY },
        })
        .then(function (response) {
          for (let i = 0; i < response.data.result.data.length; i++) {
            for (let j = 0; j < response.data.result.data[i].data.length; j++) {
              self.resasResponseResults.labels.push(
                response.data.result.data[i].data[j]["year"]
              );
              self.resasResponseResults.data.push(
                response.data.result.data[i].data[j]["value"]
              );
            }
            self.resasResponseResults.responseLabel =
              response.data.result.data[i]["label"];
            self.displayGraph(i, self.resasResponseResults.responseLabel);
            self.resasResponseResults.labels = [];
            self.resasResponseResults.data = [];
            self.resasResponseResults.responseLabel = [];
          }
        })
        .catch(function (error) {
          console.log(error);
        });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container {
  text-align: center;
}
</style>
