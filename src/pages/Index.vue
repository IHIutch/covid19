<template>
  <Layout>
    <div class="container mx-auto">
      <div class="mb-6">
        <h1>Hey</h1>
      </div>
      <table class="text-left">
        <thead>
          <tr>
            <th>Date</th>
            <th>Confirmed Count</th>
            <th>Daily</th>
            <th>Active</th>
            <th>Deaths</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, idx) in jsonData" :key="idx">
            <td>
              {{ row.Date | date }}
            </td>
            <td>
              {{ row.ConfirmedCount_Total }}
            </td>
            <td>
              {{ row.ConfirmedCount_Daily }}
            </td>
            <td>
              {{ row.Active_Total }}
            </td>
            <td>
              {{ row.Deaths_Total }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </Layout>
</template>

<script>
import dayjs from "dayjs";

export default {
  metaInfo: {
    title: "Hello, world!",
  },
  data() {
    return {
      apiUrl:
        "https://services1.arcgis.com/CgOSc11uky3egK6O/arcgis/rest/services/ErieCounty_Daily_Totals/FeatureServer/0/query?f=json&where=1%3D1&returnGeometry=false&spatialRel=esriSpatialRelIntersects&outFields=*&orderByFields=Date%20asc&resultOffset=0&resultRecordCount=2000&cacheHint=true",
      jsonData: {},
    };
  },
  created() {
    fetch(this.apiUrl)
      .then((response) => {
        return response.json();
      })
      .then((data) => {
        this.jsonData = data.features.map((data) => {
          return data.attributes;
        });
      });
  },
  filters: {
    date(value) {
      return dayjs(value).format("MM-DD");
    },
  },
};
</script>
