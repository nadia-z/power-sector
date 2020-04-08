<template>
  <div class="secondary-energy" ref="inWrapper">
    <div class="key" :data='logSome'>
      Legend or selectors if any
    </div>
    <div></div>
    <svg :width="innerWidth" :height="innerHeight" :transform="`translate(${margin.left}, 0)`">
      <g v-for="(group, g) in dots" v-bind:key="g + 'group'" :class="`${labels[g]}-group`" :transform="`translate(0, ${groupPosition[g]})`">
        <line y1="5" y2="5" :x1="scale.x(2010)" :x2="scale.x(2090)" stroke="black"/>
        <circle v-for="(dot, d) in group" v-bind:key="d + 'dot'" :class="labels[g]" :cx="dot.year" cy="5" :r="dot.value"/>
      </g>
    </svg>
  </div>
</template>

<script>
import SecondaryEnergy from 'dsv-loader!@/assets/data/SecondaryEnergy.csv' // eslint-disable-line import/no-webpack-loader-syntax
import _ from 'lodash'
import * as d3 from 'd3'

export default {
  name: 'RiskPathway',
  props: {
    width: {
      type: Number,
      default: 0
    },
    height: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      SecondaryEnergy,
      energy: _.groupBy(SecondaryEnergy, d => d.Variable),
      labels: [...new Set(SecondaryEnergy.map(r => r.Variable))],
      max: [...new Set(SecondaryEnergy.map(r => r.Value))],
      currentScenario: 'NPi2020_400_v3',
      currentRegion: 'World',
      margin: {
        top: 10,
        bottom: 10,
        right: 10,
        left: 10
      },
      innerHeight: 0
    }
  },
  computed: {
    innerWidth () { return this.width - (this.margin.left + this.margin.right) },
    scenarioFilter () { return _.map(this.energy, (sc, s) => _.filter(sc, d => d.Scenario === this.currentScenario)) },
    regionFilter () { return _.map(this.scenarioFilter, (re, r) => _.filter(re, d => d.Region === this.currentRegion)) },
    scale () {
      d3.max(this.max)
      return {
        x: d3.scaleLinear()
          .range([50, this.innerWidth - (this.margin.right * 8)])
          .domain([2010, 2090]),
        y: d3.scaleLinear()
          .range([0, 500])
          .domain([d3.min(this.max), d3.max(this.max)])
      }
    },
    dots () {
      return _.map(this.regionFilter, (energy, e) => {
        return _.map(energy, (single, s) => {
          return {
            year: this.scale.x(single.Year),
            value: this.scale.y(Math.sqrt(single.Value))
          }
        })
      })
    },
    groupPosition () {
      let pos = 50
      return _.map(this.regionFilter, (energy, e, l) => {
        if (e !== 0) { pos = pos + 140 }
        return pos
      })
    },
    logSome () {
      // console.log('dots', this.dots)
      // console.log('grouped', this.energy)
      // console.log(this.SecondaryEnergy)
      console.log(this.groupPosition)
      return 0
    }
  },
  methods: {
    calcSizes () {
      const { inWrapper: el } = this.$refs
      const innerHeight = el.clientHeight || el.parentNode.clientHeight
      this.innerHeight = Math.max(innerHeight, 500)
    }
  },
  mounted () {
    this.calcSizes()
    window.addEventListener('resize', this.calcSizes, false)
  },
  updated () {
    this.calcSizes()
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.calcSizes, false)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
@import "library/src/style/variables.scss";

.secondary-energy {
  height: 170vh;

  .key {
    width: 90%;
    height: 100px;
    margin: 0 auto;
    padding: 0 20px;
    border-bottom:0.5px solid blue;
  }

  svg {
    circle {
      fill: none;
      fill-opacity: 0.8;
      stroke: red;
    }
    .Coal {
      fill: getColor(gray, 80);
      stroke: getColor(gray, 40);
    }
    .Gas {
      fill: getColor(red, 80);
      stroke: getColor(red, 40);
    }
    .Oil {
      fill: getColor(orange, 80);
      stroke: getColor(orange, 40);
    }
    .Nuclear {
      fill: getColor(blue, 80);
      stroke: getColor(blue, 40);
    }
    .Hydro {
      fill: getColor(violet, 80);
      stroke: getColor(violet, 40);
    }
    .Geothermal {
      fill: lighten(#663333, 40);
      stroke: darken(#663333, 30);
    }
    .Solar {
      fill: getColor(yellow, 80);
      stroke: getColor(yellow, 40);
    }
    .Wind {
      fill: lighten(#336666, 40);
      stroke: darken(#336666, 30);
    }
  }
}

</style>
