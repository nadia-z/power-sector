<template>
  <div class="secondary-energy" ref="inWrapper">
    <div class="key" :data='logSome'>
      Legend or selectors if any
    </div>
    <div></div>
    <svg :width="innerWidth" :height="innerHeight" :transform="`translate(${margin.left}, 0)`">
      <g v-for="(group, g) in dots" v-bind:key="g + 'group'" :class="labels[g]">
        <circle v-for="(dot, d) in group" v-bind:key="d + 'dot'" :class="labels[g]" :cx="dot.year" cy="150" :r="dot.value"/>
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
      currentScenario: 'NPi2020_1000_v3',
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
    innerWidth () {
      return this.width - (this.margin.left + this.margin.right)
    },
    scenarioFilter () { return _.map(this.energy, (sc, s) => _.filter(sc, d => d.Scenario === this.currentScenario)) },
    regionFilter () { return _.map(this.scenarioFilter, (re, r) => _.filter(re, d => d.Region === this.currentRegion)) },
    scale () {
      return {
        x: d3.scaleLinear()
          .range([50, 800])
          .domain([2010, 2100])
      }
    },
    dots () {
      return _.map(this.regionFilter, (energy, e) => {
        return _.map(energy, (single, s) => {
          console.log(this.scale.x(single.Year), single.Value)
          return {
            year: this.scale.x(single.Year),
            value: single.Value
          }
        })
      })
    },
    logSome () {
      // console.log('dots', this.dots)
      // console.log('grouped', this.energy)
      // console.log(this.SecondaryEnergy)
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
  height: 85vh;

  .key {
    width: 90%;
    height: 100px;
    margin: 0 auto;
    padding: 0 20px;
    border-bottom:0.5px solid blue;
  }

  svg {
    background-color: lightblue;

    circle {
      fill: none;
      stroke: red;
    }
    .Coal {
      fill: grey;
      stroke: red;
    }
  }
}

</style>
