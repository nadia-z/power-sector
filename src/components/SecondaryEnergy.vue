<template>
  <div class="secondary-energy" ref="inWrapper">
    <div class="key">
      <h4>Volume in <span class="dotted">secondary energy</span> production (Ej/year)</h4>
      <p class="highlight">{{ model[0] }}</p>
      Select a scenario and a region:
      <SensesSelect class="scenario_selector" :options="scenarios" v-model="currentScenario"/>
      <SensesSelect class="region_selector" :options="regions" v-model="currentRegion"/>
    </div>
    <div></div>
    <svg :width="innerWidth" :height="innerHeight" :transform="`translate(${margin.left}, 0)`">
      <g v-for="(group, g) in dots" v-bind:key="g + 'group'" :class="`${labels[g]}-group`" :transform="`translate(0, ${groupPosition[g]})`">
        <circle v-for="(dot, d) in group" v-bind:key="d + 'dot'" :class="labels[g]" :cx="dot.year" cy="5" :r="dot.value"/>
        <g class="axis_group">
          <line class="axis" y1="5" y2="5" :x1="scale.x(2010)" :x2="scale.x(2100)"/>
          <circle class="axis-dot" :cx="scale.x(2010)" cy="5" r="2.5"/>
          <circle class="axis-dot" :cx="scale.x(2100)" cy="5" r="2.5"/>
          <text :x="scale.x(2010)" y="-10">{{ labels[g] }}</text>
        </g>
      </g>
      <g v-for="(group, g) in world" v-bind:key="g + 'wgroup'" :class="`${labels[g]}-wgroup`" :transform="`translate(0, ${groupPosition[g]})`">
        <circle v-for="(dot, d) in group" v-bind:key="d + 'wdot'" @mouseover="[active = true, over = d + labels[g]]" @mouseleave="active = false" class="world" :class="labels[g]" :cx="dot.year" cy="5" :r="dot.value"/>
        <g v-for="(text, t) in group" v-bind:key="t + 'text'" :class="active === true & over === t + labels[g] ? 'visible' : 'invisible'">
          <circle class="year-dot" :cx="text.year" cy="5" r="2.5"/>
          <rect class="shadow-label" width="80" height="30" :x="text.year - 40" y="40" rx="15"/>
          <text class="value-label" :x="text.year" y="60">{{ Math.round(text.value) }} Ej/year</text>
          <text class="year-label" :x="text.year" y="-40">{{ years[t] }}</text>
          <line class="line-label" :x1="text.year" :x2="text.year" y1="-35" y2="5"/>
        </g>
      </g>
    </svg>
  </div>
</template>

<script>
import _ from 'lodash'
import * as d3 from 'd3'

import SecondaryEnergy from 'dsv-loader!@/assets/data/SecondaryEnergy.csv' // eslint-disable-line import/no-webpack-loader-syntax
import SensesSelect from 'library/src/components/SensesSelect.vue'

export default {
  name: 'RiskPathway',
  components: {
    SensesSelect
  },
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
      model: [...new Set(SecondaryEnergy.map(r => r.Model))],
      years: [...new Set(SecondaryEnergy.map(r => r.Year))],
      labels: [...new Set(SecondaryEnergy.map(r => r.Variable))],
      scenarios: [...new Set(SecondaryEnergy.map(r => r.Scenario))],
      regions: [...new Set(SecondaryEnergy.map(r => r.Region))],
      allValues: [...new Set(SecondaryEnergy.map(r => r.Value))],
      currentScenario: 'NPi_v3',
      currentRegion: 'World',
      active: false,
      over: '',
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
    worldFilter () { return _.map(this.scenarioFilter, (re, r) => _.filter(re, d => d.Region === 'World')) },
    scale () {
      return {
        x: d3.scaleLinear()
          .range([50, this.innerWidth - (this.margin.right * 10)])
          .domain([2010, 2100]),
        y: d3.scaleLinear()
          .range([0, 500])
          .domain([d3.min(this.allValues), d3.max(this.allValues)])
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
    world () {
      return _.map(this.worldFilter, (energy, e) => {
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
        if (e !== 0) { pos = pos + this.innerHeight / 8 }
        return pos
      })
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
$margin-space: $spacing / 2;

.secondary-energy {
  height: 170vh;

  .key {
    z-index: 9;
    width: 100%;
    height: 100px;
    margin-bottom: 10%;
    padding: 10px 0px;

    position:sticky;
    top: 50px;

    border-bottom:0.5px solid blue;
    background: hsla(0,0%,100%,.90);
    .highlight {
      margin-right: $margin-space;
    }
    .scenario_selector {
      margin-top: $margin-space;
      margin-left: $margin-space;
      margin-right: $margin-space;
    }
  }

  svg {
    .axis {
      stroke: $color-gray;
    }
    circle {
      fill: $color-gray;
      fill-opacity: 0.8;
    }
    .axis-dot {
      fill-opacity: 1;
    }
    .world {
      fill-opacity: 0.1;
      stroke-dasharray: 2 2;
    }
    g {
      .value-label, .year-label {
        text-anchor: middle;
      }
      .shadow-label {
        fill-opacity: 0.6;
        fill: white;
      }
      .year-label {
        fill: $color-gray;
        font-size: 10px;
      }
      .line-label {
        stroke: $color-gray;
        stroke-width: 0.5;
      }
      .visible {
        opacity: 1;
        transition: opacity 0.5s;
      }
      .invisible {
        opacity: 0;
        transition: opacity 0.5s;
      }
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
