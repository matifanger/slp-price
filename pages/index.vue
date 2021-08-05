<template>
  <!-- Setting background color to whole app -->
  <v-container
    fluid
    style="background-color: #006466; height: 100%; position: absolute"
  >
    <!-- Github button to repository -->
    <v-card
      align="end"
      color="transparent"
      min-width="100%"
      class="pr-6"
      flat
      style="position: absolute; z-index:10"
    >
      <v-btn to="https://github.com/matifanger/slp-price" icon
        ><v-icon large>mdi-github</v-icon></v-btn
      >
    </v-card>

    <!-- Image-Logo -->
    <v-card
      align="center"
      color="transparent"
      min-width="100%"
      flat
      style="position: absolute; z-index:10"
    >
    <v-img src="slp.png" width="60" height="60"></v-img>
    </v-card>

    <!-- Price -->
    <v-row justify="center" align="center" class="fill-height">
      <v-col xs="12" sm="10" md="8">
        <v-card color="transparent" min-width="100%" flat>
          <div class="parent">
            <h1 :class="slp.change > 0 ? 'price-text-positive' : 'price-text-negative'">${{ slp.price }} {{slp.change > 0 ? arrow.up : arrow.down}}</h1>
            <v-chip
              style="font-size: 3vmin; line-height: 1.1"
              class="ml-3"
              label
              :color="slp.change > 0 ? 'green' : 'red'"
              >{{ slp.change }}%</v-chip
            >
          </div>
        </v-card>
      </v-col>
    </v-row>

    <!-- Chart -->
    <v-footer fluid fixed color="transparent" class="ma-0 pa-0">
      <div>
        <client-only>
          <line-chart
            style="width: 100vw"
            class="pa-1"
            :options="chartOptions"
            :data="chartData"
            :key="chartKey"
          ></line-chart>
        </client-only>
      </div>
    </v-footer>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      chartKey: 0,
      slp: { market: {prices: [], labels: [] }, price: "0.00", change: "0.00", last_updated_at: "0.00" },
      arrow: { down: "↙", up: "↗" },
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        scales: {
          xAxes: [
            {
              display: false,
              gridLines: {
                display: false,
              },
            },
          ],
          yAxes: [
            {
              display: false,
              gridLines: {
                display: false,
              },
            },
          ],
        },
      },
      chartData: {
        labels: [],
        datasets: [
          {
            label: "24H",
            data: [],
            borderColor: this.getGrad,
            pointRadius: 5,
          },
        ],
      },
    };
  },
  methods: {
    setData(labels,data) {
      this.chartData.datasets[0].data = data
      this.chartData.labels = labels
      this.chartKey++
    },
    getGrad(context) {
      const chart = context.chart;
      const { ctx, chartArea } = chart;

      if (!chartArea) {
        // This case happens on initial chart load
        return null;
      }
      return this.getGradient(ctx, chartArea);
    },
    getGradient(ctx, chartArea) {
      let width, height, gradient;
      const chartWidth = chartArea.right - chartArea.left;
      const chartHeight = chartArea.bottom - chartArea.top;
      if (gradient === null || width !== chartWidth || height !== chartHeight) {
        // Create the gradient because this is either the first render
        // or the size of the chart has changed
        width = chartWidth;
        height = chartHeight;
        gradient = ctx.createLinearGradient(
          0,
          chartArea.bottom,
          0,
          chartArea.top
        );
        gradient.addColorStop(0, "blue");
        gradient.addColorStop(0.5, "yellow");
        gradient.addColorStop(1, "red");
      }
      return gradient;
    },
  },
  async created() {
    // * Urls from
    let uriPrice =
      "https://api.coingecko.com/api/v3/simple/price?ids=smooth-love-potion&vs_currencies=usd&include_24hr_change=true&include_last_updated_at=true";
    let uriMarketChart =
      "https://api.coingecko.com/api/v3/coins/smooth-love-potion/market_chart?vs_currency=usd&days=1";

    const reqPrice = await axios.get(uriPrice);
    const reqMarketChart = await axios.get(uriMarketChart);

    // ! console.log(reqPrice, reqMarketChart);

    this.slp.price = reqPrice.data["smooth-love-potion"].usd.toFixed(2);
    this.slp.change = reqPrice.data["smooth-love-potion"].usd_24h_change.toFixed(2);
    this.slp.last_updated_at = reqPrice.data["smooth-love-potion"].last_updated_at;

    let labels = []
    let data = []

    reqMarketChart.data.prices.forEach(element => {
      // this.chartData.datasets.data.push(element[1])
      // this.chartData.labels.push(element[0])
      labels.push(element[0])
      data.push(element[1])
    });
    // ! console.log(labels,data)

    // Converting epoch timestamp to js date
    labels = labels.map(elem => {
      let newElem = new Date(elem).toLocaleString()
      return newElem
    })

    // Updating chart data
    this.setData(labels,data)
  },
};
</script>

<style>
h1 {
  font-size: 15vmin;
  line-height: 1.1;
}

.price-text-positive {
  background: linear-gradient(to right, hsl(98 100% 62%), hsl(204 100% 59%));
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.price-text-negative {
  background: linear-gradient(to right, hsl(9, 68%, 50%), hsl(24, 68%, 50%));
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.parent {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: -150px;
}
</style>