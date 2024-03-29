<script setup>
import "/node_modules/flag-icons/css/flag-icons.min.css"
import jsonData from './assets/json/data.json'
import moment from 'moment'
</script>

<template>
  <div class="wrapper">
    <select v-model="currentOrder">
        <option value="countrycode">Grouped by Country</option>
        <option value="product">Grouped by Product</option>
    </select> 
    <div v-if="currentOrder == 'countrycode'" class="countryselector">
      <div class="button-container">
        <div v-for="(country, index) in uniqueCountries">
          <input type="radio" :id="index" :value="country.text.toLowerCase()" v-model="picked" :checked="(defaultCountry === country.text) == 1"/>
          <label class="fi" :class="'fi-'+country.text.toLowerCase()" :for="index"></label>
        </div>
      </div>
      <div class="dataswitch">
        <button type="button" :class="buttonState === 'table' ? 'selected' : ''" @click="changeButtonState">Table</button>
        <button type="button" :class="buttonState === 'table' ? '' : 'selected'" @click="changeButtonState">Data breakdown</button>
      </div>
      <h2>{{  ISOconverter }}</h2>
      <table id="tableID" v-show="buttonState === 'table'">
        <tr class="visible">
          <th>Media</th>
          <th>Game</th>
          <th>Headline</th>
          <th>Article date</th>
        </tr>
        <tr v-for="item in Countrysort" :key="item.Id" :class="[item.countrycode, picked == item.countrycode.toLowerCase() ? 'visible' : '']" v-show="picked == item.countrycode.toLowerCase()">
          <td>
            <p>{{ item.name }}</p>
          </td>
          <td>
            <p><b>{{ item.product }}</b></p>
          </td>
          <td class="headline">
            <p>{{ item.headline }}</p>
          </td>
          <td>
            <p>{{ getFormattedDate(item.date) }}</p>
          </td>
        </tr>
      </table>
      <div id="data" v-show="buttonState === 'data'">
        <p>Number of articles: {{ articles }}</p>
        <p>Number of media sources: {{ sources }}</p>
        <p>most frequent media source: {{ frequentSource }}</p>
        <p>Number of games referenced: {{ games }}</p>
        <p>Most common game referenced: {{ frequentGames }}</p>
      </div>
    </div>


    <div v-if="currentOrder == 'product'" >
      <div class="productselector">
        <div class="product" v-for="(product, index) in uniqueGames">
          <input type="radio" :value="product.text" v-model="gamePicked" :id="index"></input>
          <label :for="index">{{ product.text }}</label>
        </div>
      </div>
      <div class="dataswitch">
        <button type="button" :class="buttonState === 'table' ? 'selected' : ''" @click="changeButtonState">Table</button>
        <button type="button" :class="buttonState === 'table' ? '' : 'selected'" @click="changeButtonState">Data breakdown</button>
      </div>
      <h2>{{  gamePicked  }}</h2>
      <p>Articles in {{  ISOconvertergraph }}</p>
      <table id="tableID" v-show="buttonState === 'table'">
        <tr class="visible">
          <th>Country</th>
          <th>Media</th>
          <th>Headline</th>
          <th>Article date</th>
        </tr>
        <tr v-for="item in Productsort" :key="item.Id" :class="[item.product, gamePicked == item.product ? 'visible' : '']" v-show="gamePicked == item.product" >
          <td>
            <label class="fi" :class="'fi-'+item.countrycode.toLowerCase()"></label>
          </td>
          <td>
            <p>{{ item.name }}</p>
          </td>
          <td class="headline">
            <p>{{ item.headline }}</p>
          </td>
          <td>
            <p>{{ getFormattedDate(item.date) }}</p>
          </td>
        </tr>
      </table>
      <div id="data" v-show="buttonState === 'data'">
        <CanvasJSChart :options="options"  :style="styleOptions" @chart-ref="chartInstance"/>
        <div class="button-container">
        <div v-for="(country, index) in uniqueCountries">
          <input type="radio" :id="-index-1" :value="country.text.toLowerCase()" v-model="graphPicked"/>
          <label class="fi" :class="'fi-'+country.text.toLowerCase()" :for="-index-1"></label>
        </div>
        {{ graphData }}
      </div>
      </div>
    </div>
  </div>
</template>

<script>
const regionNamesInEnglish = new Intl.DisplayNames(['en'], { type: 'region' });
export default{
    data(){
        return{
          search: '',
          buttonState: 'table',
          defaultGame: 'AEW: Fight Forever',
          defaultCountry: 'UK',
          picked: 'uk',
          graphPicked: 'uk',
          gamePicked: 'Diablo 4',
          currentOrder: 'countrycode',
          items: jsonData,

          options: {
          animationEnabled: true,
          exportEnabled: true,
          theme: "dark1",
          data: [
      {        
        type: "line",
        dataPoints: [        
        { x: 10, y: 71 },
        { x: 20, y: 55 },
        { x: 30, y: 50 },
        { x: 40, y: 65 },
        { x: 50, y: 95 },
        { x: 60, y: 68 },
        { x: 70, y: 28 },
        { x: 80, y: 34 },
        { x: 90, y: 14 }
        ]
      }
      ],
          axisX: {
            title: "Date",
            gridThickness: 2,
            interval:2, 
            intervalType: "month",        
            valueFormatString: "hh TT K", 
          },
          axisY: {
            logarithmic: true, //change it to false
            title: "Articles",
            titleFontSize: 16,
            valueFormatString: "0.0#E+0"
          }
        },
        styleOptions: {
          width: "100%",
          height: "360px"
        },
        }
    },
    methods: {
      chartInstance(chart) {
        this.chart = chart;
      },
      getFormattedDate(date) {
            moment.locale();  
            return moment(date).format("LL")
      },
      toggleOrder(currentOrder) {
            this.currentOrder = currentOrder;
      },
      orderedData(items) {
          return items.filter(item => {
              if (item[this.currentOrder]) return item;
          });     
      },
      changeButtonState() {
        if (this.buttonState == 'table') {
          this.buttonState = 'data';
        }
        else {
          this.buttonState = 'table';
        }
      }
    },
    computed: {
      graphData() {
        let arr = jsonData.filter((v) => v.countrycode.toLocaleLowerCase() == this.graphPicked)
        arr = arr.filter((v) => v.product == this.gamePicked)
        arr = arr.map((n) => n.date)
        if(arr == false) {arr = "No Articles about "+this.gamePicked+" in "+this.ISOconvertergraph}
        return arr
      },
      articles() {
        let arr = jsonData.filter((v) => v.countrycode.toLocaleLowerCase() == this.picked)
        return arr.length 
      },
      sources() {
        let arr = jsonData.filter((v) => v.countrycode.toLocaleLowerCase() == this.picked)
        arr = arr.map((n) => n.name)
        arr = arr.filter((v, idx) => arr.indexOf(v) == idx)
        return arr.length
      },
      frequentSource() {
        let arr = jsonData.filter((v) => v.countrycode.toLocaleLowerCase() == this.picked)
        arr = arr.map((n) => n.name).sort()
        const map = arr.reduce((acc, e) => acc.set(e, (acc.get(e) || 0) + 1), new Map());
        let media = [...map.values()]
        let count = media.indexOf(Math.max.apply(null, media))
        return [...map.keys()][count]+" ["+[...map.values()][count]+"]"
      },
      games() {
        let arr = jsonData.filter((v) => v.countrycode.toLocaleLowerCase() == this.picked)
        arr = arr.map((n) => n.product)
        arr = arr.filter((v, idx) => arr.indexOf(v) == idx)
        return arr.length
      },
      frequentGames() {
        let arr = jsonData.filter((v) => v.countrycode.toLocaleLowerCase() == this.picked)
        arr = arr.map((n) => n.product).sort()
        const map = arr.reduce((acc, e) => acc.set(e, (acc.get(e) || 0) + 1), new Map());
        let media = [...map.values()]
        let count = media.indexOf(Math.max.apply(null, media))
        return [...map.keys()][count]+" ["+[...map.values()][count]+"]"
      },
      uniqueCountries() {
        let arr = jsonData.map((v) => v.countrycode); // Take only the `Input1` value
        return arr
          .filter((v, idx) => arr.indexOf(v) == idx) // Discard values that are present already (= lower index)
          .map((v) => { return {text: v, value: v}; }); // Transform value into object with property `text` of value
      },
      uniqueGames() {
        let arr = jsonData.map((v) => v.product); // Take only the `Input1` value
        return arr
          .filter((v, idx) => arr.indexOf(v) == idx) // Discard values that are present already (= lower index)
          .map((v) => { return {text: v, value: v}; }); // Transform value into object with property `text` of value
      },
      Countrysort() {
          return this.items
              .filter(item => item.name.match(this.search))
              .sort((a, b) => {
                  if (a.currentOrder < b.currentOrder)
                      return -1;
                  if (a.currentOrder > b.currentOrder)
                      return 1;
                  return 0;
          });
      },
      Productsort() {
          return this.items
              .filter(item => item.name.match(this.search))
              .sort((a, b) => {
                  if (a.product < b.product)
                      return -1;
                  if (a.product > b.product)
                      return 1;
                  return 0;
          });
      },
      ISOconverter() {
        let regionNames = new Intl.DisplayNames(["en"], { type: "region" });
        let replacement = this.picked
        return regionNames.of(replacement.toLocaleUpperCase());
      },
      //Can we pretend we didnt see this
      ISOconvertergraph() {
        let regionNames = new Intl.DisplayNames(["en"], { type: "region" });
        let replacement = this.graphPicked
        return regionNames.of(replacement.toLocaleUpperCase());
      }
    }
}
</script>
<style scoped>
h2 {
  margin: 0;
}
.canvasjs-chart-container {
  width: 100%;
}
.dataswitch button{
  margin: 10px;
}
button.selected {
  background-color: #FFF;
  color: #1a1a1a;
  border: 2px solid #1a1a1a;
}
.fi {
  height: 40px;
    width: 50px;
    border: 5px solid transparent;
}
.game-selector {
  border: 2px solid #FFF;
  border-radius: 50px;
  padding: 5px;
  display: block;
  margin: 5px;
}
label:hover {
  cursor: pointer;
}
.countryselector {
  display: inline-flex;
  flex-direction: column;
  gap: 40px;
  margin-top: 40px;
}
.button-container, .productselector {
  display: inline-flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}
.product {
  border: 2px solid #FFF;
  margin: 10px;
  padding: 5px;
  border-radius: 10px;
}
.countryselector fi {
  border: 5px solid transparent;
}
.headline {
  max-width: 60%;
}
input[type="radio"] {
  display: none;
}
tr:nth-child(even of .visible) {
  background-color: rgba(150, 212, 212, 0.4);
}

th:nth-child(even),td:nth-child(even) {
  background-color: rgba(150, 212, 212, 0.4);
}
td.country {
  display: inline-flex;
}
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
