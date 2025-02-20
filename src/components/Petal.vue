<template>
  <g :style="styleObject">
    <path :d="pathData"/>
    
    <template v-for="(personObject, person, personIndex) of peopleData">
      <StamenFillament :personObject="personObject" :person="person" :index="index" :personIndex="personIndex" :angleSize="angleSize" :halfWidth="halfWidth" :numPeople="numPeople" :angleShift="angleShift" :petalCords="petalCords" :county="county" :state="state" :isTexasMobile="isTexasMobile" :filter="filter"></StamenFillament>
    </template>

    <template v-for="(songObject, songIndex) of this.obj">
      <StyleFillament :songObject="songObject" :index="index" :songIndex="songIndex" :angleSize="angleSize" :halfWidth="halfWidth" :totalSongs="totalSongs" :angleShift="angleShift" :county="county" :state="state" :isTexasMobile="isTexasMobile" :filter="filter">
      </StyleFillament>
    </template>

    <template v-for="(songObject, songIndex) of this.obj">
      <Style :songObject="songObject" :county="county" :state="state" :isTexasMobile="isTexasMobile" :filter="filter" v-on:filterChange="emitFilter" v-on:resetFilter="resetFilter">
      </Style>
    </template>

    <template v-for="(personObject, person, personIndex) of peopleData">
      <Stamen :person="person" :personObject="personObject" :county="county" :state="state" :isTexasMobile="isTexasMobile" :filter="filter" v-on:filterChange="emitFilter" v-on:resetFilter="resetFilter">
      </Stamen>
    </template>

    <circle :cx="xPosition" :cy="yPosition" :r="radiusSize" :class="county" v-on:mouseenter="setPetalFilter($event)" v-on:mouseleave="resetFilter"></circle>
  </g>
</template>

<script>
import { store } from '../store'
import StamenFillament from './StamenFillament'
import Stamen from './Stamen'
import StyleFillament from './StyleFillament'
import Style from './Style'
export default {
  name: 'Petal',
  store: store,
  components: {
    StamenFillament,
    StyleFillament,
    Stamen,
    Style
  },
  props: ['obj', 'county', 'index', 'angleSize', 'halfWidth', 'angleShift', 'state', 'isTexasMobile', 'filter'],
  methods: {
    getPersonAttributes: function (personName) {
      return this.personAttributes.filter((d) => { return d.name === personName })[0]
    },
    setPetalFilter: function (event) {
      this.$store.commit('setDisplayTooltip', true)
      this.$store.commit('setTooltip', {mouseX: event.clientX, mouseY: event.clientY, text: `${this.county}`})

      let allSongs = this.obj.map(d => { return d['Digital Id'] })
      let newFilter = {county: this.county, person: Object.keys(this.peopleData), id: allSongs}
      this.emitFilter(newFilter)
    },
    resetFilter: function () {
      this.$store.commit('setDisplayTooltip', false)
      this.$emit('filterChange', {})
    },
    emitFilter: function (newFilter) {
      this.$emit('filterChange', newFilter)
    }
  },
  computed: {
    shouldRender: function () {
      if (Object.keys(this.filter).length === 0) { return true }
      return this.filter.county === this.county
    },
    styleObject: function () {
      return {
        display: this.shouldRender ? 'block' : 'none'
      }
    },
    pathData: function () {
      return `M0 0 C ${this.firstControlPointX} ${this.firstControlPointY}, ${this.secondControlPointX} ${this.secondControlPointY}, ${this.xPosition} ${this.yPosition}`
    },
    firstControlPointX: function () {
      return 1 / 3 * this.xPosition
    },
    firstControlPointY: function () {
      return this.yPosition + 10
    },
    secondControlPointX: function () {
      return 2 / 3 * this.xPosition
    },
    secondControlPointY: function () {
      return this.yPosition + 10
    },
    petalCords: function () {
      return {x: this.xPosition, y: this.yPosition}
    },
    radiusSize: function () {
      if (this.isTexasMobile) {
        return 5
      }
      return 8
    },
    xPosition: function () {
      if (this.isTexasMobile) {
        return this.halfWidth / 3.75 * Math.cos(this.angle)
      }
      return this.halfWidth / 4 * Math.cos(this.angle)
    },
    angle: function () {
      return ((this.index + 0.5) * this.angleSize) + this.angleShift
    },
    yPosition: function () {
      if (this.isTexasMobile) {
        return -this.halfWidth / 3.75 * Math.sin(this.angle)
      }
      return -this.halfWidth / 4 * Math.sin(this.angle)
    },
    numPeople: function () {
      return Object.keys(this.peopleData).length
    },
    peopleData: function () {
      let people = {}
      let peopleSet = new Set()
      for (let i = 0; i < this.obj.length; i++) {
        for (let j = 0; j < this.obj[i]['Contributor Names'].length; j++) {
          if (!this.obj[i]['Contributor Names'][j].endsWith(' (Collector)') && !peopleSet.has(this.obj[i]['Contributor Names'][j])) {
            peopleSet.add(this.obj[i]['Contributor Names'][j])
            people[this.obj[i]['Contributor Names'][j]] = [this.obj[i]]
          } else if (!this.obj[i]['Contributor Names'][j].endsWith(' (Collector)') && peopleSet.has(this.obj[i]['Contributor Names'][j])) {
            people[this.obj[i]['Contributor Names'][j]] = [...people[this.obj[i]['Contributor Names'][j]], this.obj[i]]
          }
        }
      }
      return people
    },
    totalSongs: function () {
      return this.obj.length
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
circle{
  fill: url(#RadialGradient1);
}
circle:hover{
  cursor: pointer;
}
path{
  stroke: white; 
  fill: transparent;
}
</style>