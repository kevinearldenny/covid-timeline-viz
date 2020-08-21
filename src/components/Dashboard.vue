<template>
  <div id="app">
    <div class="title" v-if="formattedDate">
      <span>COVID-19 analysis, {{ formattedDate.start }} to {{ formattedDate.end }}</span>
      <span><button id="play-button" v-on:click="playButton">{{ buttonText }}</button></span>
    </div>
    <div id="vis">
      <div class="mini-title">
        Timeline (drag to change map + curve)
      </div>

    </div>
    <div v-if="x && width">
      <line_chart :covidcurve="covidCurve" :xscale="x" :width="width" :currentstate="currentState"></line_chart>
    </div>
    <div v-if="geojson && width">
      <states_map :geojson="geojson" :width="width" :currentday="currentDay" :currentstate="currentState" v-on:stateselect="selectState"></states_map>
    </div>
  </div>
</template>

<script>
  import covid from "../data/covid.js";
  import states from "../data/us-states.json";
  import population from "../data/states-population.json";
  import * as d3 from "d3";
  import StatesMap from './Map'
  import LineChart from './LineChart'

  export default {
  name: 'Dashboard',
  components: {
    states_map: StatesMap,
    line_chart: LineChart
  },
  data () {
    return {
      currentValue: 0,
      todayData: null,
      currentState: null,
      currentDay: null,
      x: null,
      startDate: null,
      endDate: null,
      buttonText: 'Play'
    }
  },
  computed: {
    geojson () {
      var me = this
      if (states && me.todayData) {
        return states.features.map(d => {
          var stateCases = me.todayData.filter((l) => l.state === d.properties.name)
          if (stateCases.length > 0) {
            d.properties.cases = stateCases[0].cases
            d.properties.casesPerCapita = stateCases[0].casesPerCapita
            d.properties.deaths = stateCases[0].deaths
          } else {
            d.properties.cases = 0
            d.properties.casesPerCapita = 0
            d.properties.deaths = 0
          }

          return d
        })
      } else {
        return null
      }
    },
    covidData () {
      // var parseDate2 = d3.timeParse("%Y-%m-%d");
      if (covid) {
        return covid.map(d => {
          d.cases = +d.cases;
          d.population = population[d.state];
          d.casesPerCapita = (d.cases / d.population) * 100000;
          return d
        })
      } else {
        return null
      }
    },
    formattedDate () {
      var formatDate = d3.timeFormat("%m/%d/%Y")
      if (this.startDate && this.endDate) {

        return {
          start: formatDate(this.startDate),
          end: formatDate(this.endDate)
        }
      } else {
        return null
      }
    },
    covidCurve () {
      var me = this
      var days;
      var parseDate = d3.timeParse("%Y-%m-%d")
      if (this.covidData && this.currentDay) {
        if (!this.currentState) {
          days = d3.nest()
              .key(function(d) { return d.date; })
              .entries(me.covidData)

        } else {
          var stateData = this.covidData.filter(d => d.state === me.currentState)
          days = d3.nest()
              .key(function(d) { return d.date; })
              .entries(stateData)
        }

        return days.filter(d => parseDate(d.key) <= parseDate(me.currentDay)).map((d, i) => {
          d.daySum = d3.sum(d.values, v => v.cases)
          if (i > 0) {
            d.newCases = d.daySum - days[i-1].daySum
          } else {
            d.newCases = d.daySum
          }
          return d
        })

      } else {
        return null
      }
    }
  },
  watch: {

  },
  mounted: function () {
    var me = this
    this.formatDateIntoYear = d3.timeFormat("%b %Y");
    this.formatDate = d3.timeFormat("%b %d %Y");
    this.parseDate2 = d3.timeParse("%Y-%m-%d");


      this.endDate = new Date("2020-05-08");

    this.currentDay = "2020-01-21"
    this.startDate = this.parseDate2(this.currentDay)


    if (screen.width > 800) {
      this.margin = { top: 50, right: 50, bottom: 0, left: 75 };
      this.width = 1100 - this.margin.left - this.margin.right;
    } else {
      this.margin = { top: 50, right: 25, bottom: 0, left: 110 };
      this.width = 600 - this.margin.left - this.margin.right
    }
    this.height = 250 - this.margin.top - this.margin.bottom;
    // var timeFormat = d3.timeFormat("%Y-%m-%d");

    this.todayData = this.covidData.filter((d) => d.date === me.currentDay);

    this.svg = d3
        .select("#vis")
        .append("svg")
        .attr("width", me.width + me.margin.left + me.margin.right)
        .attr("height", me.height)
        .attr("transform", "translate(" + me.margin.left + "," + me.height / 8 + ")");

    this.renderSlider();

  },
  methods: {
    renderSlider () {
      var me = this
      this.moving = false;
      this.targetValue = this.width;

      this.timer;

      this.x = d3
          .scaleTime()
          .domain([me.startDate, me.endDate])
          .range([0, me.targetValue])
          .clamp(true);

      var slider = me.svg
          .append("g")
          .attr("class", "slider")
          .attr("transform", "translate(" + me.margin.left + "," + me.margin.top + ")");

      slider
          .append("line")
          .attr("class", "track")
          .attr("x1", me.x.range()[0])
          .attr("x2", me.x.range()[1])
          .select(function () {
            return this.parentNode.appendChild(this.cloneNode(true));
          })
          .attr("class", "track-inset")
          .select(function () {
            return this.parentNode.appendChild(this.cloneNode(true));
          })
          .attr("class", "track-overlay")
          .call(
              d3
                  .drag()
                  .on("start.interrupt", function () {
                    slider.interrupt();
                  })
                  .on("start drag", function () {
                    me.currentValue = d3.event.x;
                    me.update(me.x.invert(me.currentValue));
                  })
          );

      slider
          .insert("g", ".track-overlay")
          .attr("class", "ticks")
          .attr("transform", "translate(0," + 18 + ")")
          .selectAll("text")
          .data(me.x.ticks(5))
          .enter()
          .append("text")
          .attr("x", me.x)
          .attr("y", 10)
          .attr("text-anchor", "middle")
          .text(function (d) {
            return me.formatDateIntoYear(d);
          });

      this.handle = slider
          .insert("circle", ".track-overlay")
          .attr("class", "handle")
          .attr("r", 9);

      this.label = slider
          .append("text")
          .attr("class", "label")
          .attr("text-anchor", "middle")
          .text(me.formatDate(me.startDate))
          .attr("transform", "translate(0," + -25 + ")");

    },
    playButton () {
      var me = this
      if (this.buttonText === "Pause") {
        me.moving = false;
        clearInterval(me.timer);
        // timer = 0;
        this.buttonText = 'Play';
      } else {
        me.moving = true;
        me.timer = setInterval(me.step, 100);
        this.buttonText = 'Pause';
      }
      console.log("Slider moving: " + this.moving);
    },
    step () {
      var me = this
      this.update(me.x.invert(me.currentValue));
      this.currentValue = this.currentValue + this.targetValue / 151;
      if (this.currentValue > this.targetValue) {
        this.moving = false;
        this.currentValue = 0;
        clearInterval(me.timer);
        // timer = 0;
        this.playButton.text("Play");
        console.log("Slider moving: " + this.moving);
      }
    },
    update (h) {
      var timeFormat = d3.timeFormat("%Y-%m-%d");
      this.currentDay = timeFormat(h)
      var me = this
      this.handle.attr("cx", me.x(h));
      this.label.attr("x", me.x(h)).text(me.formatDate(h));
      this.todayData = this.covidData.filter((d) => d.date === timeFormat(h));
    },
    selectState (d) {
      this.currentState = d;
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
  body {
    font-family: "avenir next", Arial, sans-serif;
    font-size: 12px;
    color: #696969;
  }
  @media (min-width: 800px) {
    /* CSS that should be displayed if width is equal to or less than 800px goes here */
    #app {
      margin-left: 200px;
    }
  }
  #vis {
    margin-top: 30px;
  }
  #play-button {
    /*position: absolute;*/
    /*top: 110px;*/
    /*left: 110px;*/
    background: #f08080;
    padding-right: 26px;
    border-radius: 3px;
    margin-left: 20px;
    /*margin-top: 2px;*/
    border: none;
    color: white;
    /*margin: 0;*/
    padding: 0 12px;
    width: 60px;
    cursor: pointer;
    height: 30px;
    vertical-align: top;
  }

  #play-button:hover {
    background-color: #696969;
  }

  .ticks {
    font-size: 10px;
  }

  .track,
  .track-inset,
  .track-overlay {
    stroke-linecap: round;
  }

  .track {
    stroke: #000;
    stroke-opacity: 0.3;
    stroke-width: 10px;
  }

  .track-inset {
    stroke: #dcdcdc;
    stroke-width: 8px;
  }

  .track-overlay {
    pointer-events: stroke;
    stroke-width: 50px;
    stroke: transparent;
    cursor: crosshair;
  }

  .handle {
    fill: #fff;
    stroke: #000;
    stroke-opacity: 0.5;
    stroke-width: 1.25px;
  }
  div.tooltip {
    position: absolute;
    text-align: center;
    width: 80px;
    height: 60px;
    padding: 2px;
    font: 12px sans-serif;
    background-color: #b6cdc2;
    color: #848484;
    border-radius: 5px;
    border: 0px;
    pointer-events: none;
  }
  .title {
    margin-left: 80px;
    font-size: 24px;
    vertical-align: center;
  }
  #line-viz {
    /*height: 200px;*/
  }
  .line {
    fill: none;
    stroke: #ffab00;
    stroke-width: 3;
  }
  .mini-title {
    font-size: 18px;
    margin-left: 75px;
  }
</style>
