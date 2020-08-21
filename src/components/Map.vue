<template>
    <div id="map-box">
        <div class="mini-title">Cases per 100,000 pop.</div>
    </div>
</template>

<script>
  import * as d3 from "d3";

  export default {
    name: 'GeoMap',
    props: ['geojson', 'width'],
    components: {

    },
    data () {
      return {

      }
    },
    computed: {

    },
    watch: {
        geojson () {
          this.updateMap();
        }
    },
    mounted: function () {
      var me = this

      this.margin = { top: 125, right: 50, bottom: 0, left: 25 };
      // this.width = 1100 - this.margin.left - this.margin.right;

      this.height = 700 - this.margin.top - this.margin.bottom;


      this.svg = d3
          .select("#map-box")
          .append("svg")
          .attr("width", me.width + me.margin.left + me.margin.right)
          .attr("height", me.height + me.margin.top + me.margin.bottom)
          .attr("display", "block")
          .append("g")
          .attr("transform", "translate(" + me.margin.left + "," + me.margin.top + ")");

      this.color = d3
          .scaleThreshold()
          .domain([0, 50, 100, 500, 1000])
          .range(["#F8F8F8", "#fdd49e", "#fdbb84", "#fc8d59", "#e34a33", "#b30000"]);

      this.projection = d3.geoAlbersUsa().translate([me.width / 1.8, me.height / 3.5]).scale(1225);
      this.path = d3.geoPath().projection(me.projection);
      this.drawMap()

      this.tooltip = d3.select("#map-box").append("div")
          .attr("class", "tooltip")
          .style("opacity", 0);

    },
    methods: {
        drawMap () {
          var me = this
          this.svg.selectAll("path")
              .data(me.geojson)
              .enter()
              .append("path")
              .attr("d", me.path)
              .style("stroke", "#808080")
              .style("fill", function (d) {
                var raw = d.properties.casesPerCapita;
                if (raw) {
                  // if value exists
                  return me.color(raw);
                } else {
                  // if value is undefined...
                  return "#F8F8F8";
                }
              })
        },
        updateMap () {
          this.svg.selectAll("path").remove();
          this.drawMap();
        },
        selectState () {

        }
    }
  }
</script>
<style lang="scss">
    path {
        cursor: pointer;
    }
    #map-box {
    }
</style>