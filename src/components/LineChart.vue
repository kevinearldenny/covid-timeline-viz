<template>
    <div id="line-box">
        <div class="mini-title">New cases per day</div>
    </div>
</template>

<script>
  import * as d3 from "d3";

  export default {
    name: 'LineChart',
    props: ['covidcurve', 'xscale', 'width'],
    components: {

    },
    data () {
      return {

      }
    },
    computed: {

    },
    watch: {
        covidcurve () {
            this.updateMap();
        }
    },
    mounted: function () {
      var me = this

      this.margin = { top: 10, right: 180, bottom: 10, left: 160 };
      // this.width = 1300 - this.margin.left - this.margin.right;
      this.height = 150 - this.margin.top - this.margin.bottom;


      this.svg = d3
          .select("#line-box")
          .append("svg")
          .attr("width", me.width + me.margin.left + me.margin.right)
          .attr("height", me.height + me.margin.top + me.margin.bottom)
          .attr("display", "block");

      this.drawChart()
    },
    methods: {
        drawChart () {
          var me = this
          this.chart = this.svg.append("g").attr("transform", "translate(" + me.margin.left + "," + me.margin.top + ")");
          var timeParse = d3.timeParse("%Y-%m-%d")
          this.yScale = d3
              .scaleLinear()
              .range([me.height, 0])
              .domain([0, 50000]);


          var line = d3.line()
              .x(function(d) { return me.xscale(timeParse(d.key)); }) // set the x values for the line generator
              .y(function(d) { return me.yScale(d.newCases); }) // set the y values for the line generator
              .curve(d3.curveMonotoneX) // apply smoothing to the line

          this.chart.append("path")
              .datum(me.covidcurve)
              .attr("class", "line")
              .attr("d", line);

          this.chart
              .append("g")
              .attr("transform", `translate(${me.width+10}, 0)`)
              .call(d3.axisRight(me.yScale));
        },
        updateMap () {
          this.svg.selectAll('g').remove();
          this.drawChart();
        }
    }
  }
</script>
<style lang="scss">
    #line-box {
        margin-top: -50px;
        margin-bottom: 20px;
    }
</style>