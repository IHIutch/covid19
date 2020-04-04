<template>
  <Layout>
    <h1>Hey</h1>
  </Layout>
</template>

<script>
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
};
</script>
