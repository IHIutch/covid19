<template>
  <Layout>
    <div class="container mx-auto flex py-8">
      <div class="flex-shrink-0 px-4 relative">
        <div class="sticky top-0 -mt-8 py-8">
          <div class="mb-4">
            <h1 class="">
              <div>
                <span
                  class="uppercase text-gray-600 tracking-wider font-medium text-sm"
                  >Last Updated:
                </span>
              </div>
              <span class="text-2xl font-medium">{{ lastUpdated }}</span>
            </h1>
          </div>
          <table class="text-left table-auto">
            <thead>
              <tr>
                <th class="border px-2 py-1">Date</th>
                <th class="border px-2 py-1">Count</th>
                <th class="border px-2 py-1">Diff</th>
                <th class="border px-2 py-1">Test Rate</th>
                <th class="border px-2 py-1">Weekly Test Rate</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(row, idx) in formattedData.reverse()"
                :key="idx"
                :class="{ 'bg-gray-100': idx % 2 }"
              >
                <td class="border px-2 py-1">
                  {{ row.date | date }}
                </td>
                <td class="border px-2 py-1">
                  {{ row.count }}
                </td>
                <td class="border px-2 py-1">
                  {{ row.diff == 0 ? "-" : row.diff }}
                </td>
                <td class="border px-2 py-1">
                  {{ row.testRate ? `${row.testRate}%` : "-" }}
                </td>
                <td class="border px-2 py-1">
                  {{ row.testWeeklyRate ? `${row.testWeeklyRate}%` : "-" }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <div class="flex-grow px-4">
        <div class="shadow border rounded p-4 w-full mb-6">
          <div>
            <h2 class="font-medium text-2xl">Confirmed Daily Count</h2>
          </div>
          <canvas id="chartOne"></canvas>
        </div>
        <div class="shadow border rounded p-4 w-full mb-6">
          <div>
            <h2 class="font-medium text-2xl">Daily Count Difference</h2>
          </div>
          <canvas id="chartTwo"></canvas>
        </div>
        <div class="shadow border rounded p-4 w-full mb-6">
          <div>
            <h2 class="font-medium text-2xl">Infection Rate</h2>
          </div>
          <canvas id="chartThree"></canvas>
        </div>
        <div class="shadow border rounded p-4 w-full mb-6">
          <div>
            <h2 class="font-medium text-2xl">Prediction Counts</h2>
          </div>
          <canvas id="chartFour"></canvas>
        </div>
      </div>
    </div>
  </Layout>
</template>

<script>
import dayjs from "dayjs";
import ChartJS from "chart.js";

export default {
  metaInfo: {
    title: "Covid 19 Data",
  },
  data() {
    return {
      colors: [
        "#7fdbff",
        "#85144b",
        "#0074d9",
        "#01ff70",
        "#001f3f",
        "#39cccc",
        "#ff4136",
        "#ff851b",
        "#b10dc9",
        "#ffdc00",
        "#2ecc40",
        "#f012be",
        "#aaaaaa",
        "#111111",
        "#3d9970",
        "#dddddd",
      ],
      pctUrl: "https://services1.arcgis.com/CgOSc11uky3egK6O/arcgis/rest/services/AddMoreDates_Trends_VIEW/FeatureServer/0/query?f=json&where=Daily_Pct%20IS%20NOT%20NULL&returnGeometry=false&spatialRel=esriSpatialRelIntersects&outFields=*&orderByFields=Date%20asc&outSR=102100&resultOffset=0&resultRecordCount=32000&resultType=standard&cacheHint=true",
      apiUrl:
        "https://services1.arcgis.com/CgOSc11uky3egK6O/arcgis/rest/services/AddMoreDates_DailyTable_ViewLayer/FeatureServer/0/query?f=json&where=Daily_Count%20IS%20NOT%20NULL&returnGeometry=false&spatialRel=esriSpatialRelIntersects&outFields=*&orderByFields=Date%20asc&resultOffset=0&resultRecordCount=32000&resultType=standard&cacheHint=true",
      dataUrl:
        "https://services1.arcgis.com/CgOSc11uky3egK6O/arcgis/rest/services/COVID_CNTY_Outline_1_VIEW/FeatureServer/0?f=json",
      lastUpdated: "",
      jsonData: [],
      pctData:[]
    };
  },
  mounted() {
    this.fetchData();
    this.fetchPctData();
    this.fetchUpdateTime();
  },
  methods: {
    fetchUpdateTime() {
      fetch(this.dataUrl)
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          this.lastUpdated = dayjs(data.editingInfo.lastEditDate).format(
            "MMMM DD, YYYY hh:mma"
          );
        });
    },
    fetchData() {
      fetch(this.apiUrl)
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          this.jsonData = data.features.map((data) => {
            return data.attributes;
          }).map(data => {
            return {
              ...data, 
              date: this.formatDate(data.Date)
            }
          });
        })
        .then(() => {
          this.initCharts();
        });
    },
    fetchPctData(){
      fetch(this.pctUrl)
        .then((response) => {
          return response.json();
        })
        .then((data) => {
          this.pctData = data.features.map((data) => {
            return data.attributes;
          }).map(data => {
            return {
              ...data, 
              date: this.formatDate(data.Date)
            }
          });
        })
    },
    formatDate(date) {
      return dayjs(date).format("MM/DD");
    },
    getRate(idx) {
      if (!this.jsonData[idx - 1]) return 0;
      const v1 = this.jsonData[idx - 1].Confirmed_Total;
      const v2 = this.jsonData[idx].Confirmed_Total;
      return ((v2 - v1) / v1);
    },
    getWeeklyAvg(idx) {
      if (idx < 7) return 0;
      let arr = [];
      for (let i = idx - 6; i <= idx; i++) {
        var rate = this.getRate(i);
        if (rate == 0) return 0;
        arr.push(rate);
      }
      return (
        arr.reduce((sum, value) => {
          return sum + value;
        }, 0) / arr.length
      );
    },
    initCharts() {
      this.initChartOne();
      this.initChartTwo();
      this.initChartThree();
      this.initChartFour();
    },
    initChartOne() {
      const data = {
        labels: this.formattedData.map((data) => {
          return this.formatDate(data.date);
        }),
        datasets: [
          {
            fill: false,
            backgroundColor: this.colors[0],
            borderColor: this.colors[0],
            data: this.formattedData.map((data) => {
              return data.count;
            }),
          },
        ],
      };
      const ctx = document.getElementById("chartOne");
      const chart = new Chart(ctx, {
        type: "line",
        data: data,
        options: {
          legend: {
            display: false,
          },
          responsive: true,
          tooltips: {
            mode: "index",
            intersect: false,
          },
        },
      });
    },
    initChartTwo() {
      const data = {
        labels: this.formattedData.map((data) => {
          return this.formatDate(data.date);
        }),
        datasets: [
          {
            fill: false,
            backgroundColor: this.colors[0],
            borderColor: this.colors[0],
            data: this.formattedData.map((data, idx) => {
              return this.formattedData[idx - 1] ? data.diff : 0;
            }),
          },
        ],
      };
      const ctx = document.getElementById("chartTwo");
      const chart = new Chart(ctx, {
        type: "line",
        data: data,
        options: {
          legend: {
            display: false,
          },
          responsive: true,
          tooltips: {
            mode: "index",
            intersect: false,
          },
        },
      });
    },
    initChartThree() {
      const data = {
        labels: this.formattedData.map((data) => {
          return this.formatDate(data.date);
        }),
        datasets: [
          {
            fill: false,
            backgroundColor: this.colors[0],
            borderColor: this.colors[0],
            data: this.formattedData.map((data) => {
              return data.rate;
            }),
          },
        ],
      };
      const ctx = document.getElementById("chartThree");
      const chart = new Chart(ctx, {
        type: "line",
        data: data,
        options: {
          legend: {
            display: false,
          },
          responsive: true,
          tooltips: {
            mode: "index",
            intersect: false,
          },
        },
      });
    },
    initChartFour() {
      let labels = this.predictedData.reduce((acc, data) => {
        return [...new Set([...acc, ...data.dates])];
      }, []);
      let datasets = this.predictedData.map((data, dataIdx) => {
        return {
          fill: false,
          backgroundColor: this.colors[dataIdx % this.colors.length],
          borderColor: this.colors[dataIdx % this.colors.length],
          label: data.dates[0] + " - " + data.dates[data.dates.length - 1],
          data: labels.map((label) => {
            let labelIdx = data.dates.indexOf(label);
            return labelIdx == -1 ? null : data.counts[labelIdx];
          }),
        };
      });
      const data = {
        labels: labels,
        datasets: datasets,
      };
      const ctx = document.getElementById("chartFour");
      const chart = new Chart(ctx, {
        type: "line",
        data: data,
        options: {
          responsive: true,
          tooltips: {
            mode: "index",
            intersect: false,
          },
        },
      });
    },
    getPrediction(data) {
      let obj = {
        dates: [data.date],
        counts: [data.count],
      };
      for (let i = 0; i < 6; i++) {
        obj.dates.push(obj.dates[i] + 86400000);
        let multi = obj.counts[i] * (1 + parseFloat(data.weeklyAvgRate));
        obj.counts.push(multi.toFixed(2));
      }
      obj.dates = obj.dates.map((date) => {
        return this.formatDate(date);
      });
      return obj;
    },
    findByDate(key, date){
      if(!this.pctData.length) return null
      var obj = this.pctData.find((data) => {
          return data.date == date
      })
      // console.log(obj ? obj['Daily_Pct'] +  ", " + obj['Week_Pct'] : null)
      return obj ? obj[key] : null
    }
  },
  computed: {
    formattedData() {
      const arrLength = this.jsonData.length;
      return this.jsonData.map((data, idx) => {
          return {
            date: data.Date,
            count: data.Confirmed_Total,
            diff: this.jsonData[idx - 1]
              ? this.jsonData[idx].Confirmed_Total -
                this.jsonData[idx - 1].Confirmed_Total
              : 0,
            rate: this.getRate(idx).toFixed(2),
            weeklyAvgRate: this.getWeeklyAvg(idx).toFixed(2),
            testRate: this.findByDate('Daily_Pct', data.date),
            testWeeklyRate: this.findByDate('Week_Pct', data.date),
            active: data.Active_Total,
            deaths: data.Deaths_Total,
          };
        }).splice(-1* arrLength + 13, arrLength - 13);
        // Remove until Apr 1
    },
    predictedData() {
      const pastDays = 30;
      const filteredArr = this.formattedData.filter((data) => {
        return data.weeklyAvgRate != 0;
      });
      return filteredArr.map((data) => {
        return this.getPrediction(data);
      }).splice(-1* pastDays, pastDays);
    },
  },
  filters: {
    date(value) {
      return dayjs(value).format("MM/DD");
    },
  },
};
</script>
