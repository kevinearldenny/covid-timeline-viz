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

      this.margin = { top: 130, right: 250, bottom: 0, left: 25 };
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



      this.projection = d3.geoAlbersUsa().translate([me.width / 1.8, me.height / 3.5]).scale(me.width * 1.2);
      this.path = d3.geoPath().projection(me.projection);
      this.drawMap()
      this.drawLegend()

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
        drawLegend () {
          var me = this
          this.legend = this.svg
              .append("g")
              .attr("class", "legendQuant")
              .attr("transform", "translate(20,20)");

          var size = 20;
          var legendPosition = {
            x: this.width + 120,
            y: this.height - 240
          };
          var colorDomain = this.color.domain();

          /// Add legend title
          this.legend
              .append("text")
              .attr("x", legendPosition.x - 20)
              .attr("y", function () {
                return legendPosition.y - 20;
              })
              .style("fill", "#505050")
              .text("Cases per 1000 pop.")
              .attr("text-anchor", "left")
              .style("alignment-baseline", "middle");

          /// Add legend dots
          this.legend
              .selectAll("mydots")
              .data(colorDomain)
              .enter()
              .append("rect")
              .attr("x", legendPosition.x)
              .attr("y", function (d, i) {
                return legendPosition.y + i * (size + 5);
              })
              .attr("width", size)
              .attr("height", size)
              .style("fill", function (d) {
                return me.color(d);
              });

          // Add legend labels
          this.legend
              .selectAll("mylabels")
              .data(colorDomain)
              .enter()
              .append("text")
              .attr("x", legendPosition.x + size * 1.2)
              .attr("y", function (d, i) {
                return legendPosition.y + i * (size + 5) + size / 2;
              })
              .style("fill", "#505050")
              .text(function (d, i) {
                if (colorDomain.length > i + 1) {
                  let s = d + 1;
                  let next = colorDomain[i + 1];
                  return s + "-" + next;
                } else {
                  return d + "+";
                }
              })
              .attr("text-anchor", "left")
              .style("alignment-baseline", "middle");

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
        margin-top: 20px;
    }
</style>