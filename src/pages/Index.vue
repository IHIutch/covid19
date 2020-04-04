<template>
  <Layout>
    <div class="container mx-auto flex my-8">
      <div class="flex-shrink-0 px-4">
        <table class="text-left table-auto">
          <thead>
            <tr>
              <th class="border px-2 py-1">Date</th>
              <th class="border px-2 py-1">Count</th>
              <th class="border px-2 py-1">Daily</th>
              <th class="border px-2 py-1">Active</th>
              <th class="border px-2 py-1">Deaths</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(row, idx) in jsonData"
              :key="idx"
              :class="{ 'bg-gray-100': idx % 2 }"
            >
              <td class="border px-2 py-1">
                {{ row.Date | date }}
              </td>
              <td class="border px-2 py-1">
                {{ row.ConfirmedCount_Total }}
              </td>
              <td class="border px-2 py-1">
                {{ row.ConfirmedCount_Daily }}
              </td>
              <td class="border px-2 py-1">
                {{ row.Active_Total }}
              </td>
              <td class="border px-2 py-1">
                {{ row.Deaths_Total }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="flex-grow px-4">
        <div class="shadow border rounded p-4 w-full mb-6">
          <div>
            <h2 class="font-medium text-2xl">Confirmed Daily Count</h2>
          </div>
          <div id="chartOne"></div>
        </div>
        <div class="shadow border rounded p-4 w-full mb-6">
          <div>
            <h2 class="font-medium text-2xl">Daily Count Difference</h2>
          </div>
          <div id="chartTwo"></div>
        </div>
        <div class="shadow border rounded p-4 w-full mb-6">
          <div>
            <h2 class="font-medium text-2xl">Daily Count Difference</h2>
          </div>
          <div id="chartThree"></div>
        </div>
      </div>
    </div>
  </Layout>
</template>

<script>
import dayjs from "dayjs";
import { Chart } from "frappe-charts/dist/frappe-charts.min.esm";

export default {
  metaInfo: {
    title: "Hello, world!",
  },
  data() {
    return {
      apiUrl:
        "https://services1.arcgis.com/CgOSc11uky3egK6O/arcgis/rest/services/ErieCounty_Daily_Totals/FeatureServer/0/query?f=json&where=1%3D1&returnGeometry=false&spatialRel=esriSpatialRelIntersects&outFields=*&orderByFields=Date%20asc&resultOffset=0&resultRecordCount=2000&cacheHint=true",
      jsonData: [],
      formattedData: {},
    };
  },
  mounted() {
    fetch(this.apiUrl)
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        this.jsonData = data.features.map((data) => {
          return data.attributes;
        });
      })
      .then(() => {
        this.initCharts();
      });
  },
  methods: {
    formatDates(value) {
      return dayjs(value).format("MM-DD");
    },
    initCharts() {
      this.initChartOne();
      this.initChartTwo();
      this.initChartThree();
    },
    initChartOne() {
      const data = {
        labels: this.jsonData.map((data) => {
          return this.formatDates(data.Date);
        }),
        datasets: [
          {
            values: this.jsonData.map((data) => {
              return data.ConfirmedCount_Total;
            }),
          },
        ],
      };
      const chart = new Chart("#chartOne", {
        data: data,
        type: "line",
        height: 250,
        colors: ["#7cd6fd"],
      });
    },
    initChartTwo() {
      const data = {
        labels: this.jsonData.map((data) => {
          return this.formatDates(data.Date);
        }),
        datasets: [
          {
            values: this.jsonData.map((data, idx) => {
              if (!this.jsonData[idx - 1]) return 0;
              return (
                data.ConfirmedCount_Total -
                this.jsonData[idx - 1].ConfirmedCount_Total
              );
            }),
          },
        ],
      };
      const chart = new Chart("#chartTwo", {
        data: data,
        type: "line",
        height: 250,
        colors: ["#743ee2"],
      });
    },
    initChartThree() {
      const data = {
        labels: this.jsonData.map((data) => {
          return this.formatDates(data.Date);
        }),
        datasets: [
          {
            values: this.jsonData.map((data, idx) => {
              if (!this.jsonData[idx - 1]) return 0;
              return (
                data.ConfirmedCount_Total -
                this.jsonData[idx - 1].ConfirmedCount_Total
              );
            }),
          },
        ],
      };
      const chart = new Chart("#chartThree", {
        data: data,
        type: "line",
        height: 250,
        colors: ["#ffa00a"],
      });
    },
  },
  filters: {
    date(value) {
      return dayjs(value).format("MM-DD");
    },
  },
};
</script>
