<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
         <apexchart align="center" width="70%" height="300" type="bar" :options="options" :series="series"></apexchart>

       <!-- <select v-model="selectedStock">
             <option disabled value="">Please select one</option>
             <option v-for="(item,index) in listOfAllStocks" v-bind:value="item" v-bind:key="index">{{item.title}}</option>
     </select> -->
     <br/>
     <br/>
     <p>Please enter the stock code below. The Stock codes can be found <a href="http://eoddata.com/stocklist/NASDAQ.htm" target="_blank">here</a></p>
          <p v-if="showErrorMessage" style="color:red;">The Stock has already been added</p>

     <input type="text" v-model="stockToCheck" placeholder="Enter the Stock code">
     <button @click="addtoarray">Add</button>
     <h3>The Selected Stocks are</h3>
     <div class="stock-list-wrap">
      <ul class="stock-list">
          <li v-for="(obs,index) in listOfSelectedStocks" :key="index">
            <span>{{obs}}</span>
            <button class="button" v-on:click="deleteFunc(index)">Delete</button>
          </li>
      </ul>
     </div>
  </div>

</template>

<script>
import axios from 'axios';



export default {
  name: 'HelloWorld',

  data () {
    return {
      msg: 'Welcome to StockChecker',
      listOfSelectedStocks: [],
      values: [],
      dataOfSelectedStocks: [],
      stockToCheck: '',
      selectedStock : '',
      selectedDelete:'',
      showErrorMessage: false,
      options: {
        chart: {
          id: 'vuechart-example'
        },
        xaxis: {
          categories: []
        }
      },
      series: [{
        data: [],
        name : "stocks"
      }]
    }
  },
  methods:{
    addtoarray() {
      console.log("Reached add function")
       this.stockToCheck = this.stockToCheck.trim(" ").toUpperCase()
      if(!this.listOfSelectedStocks.includes(this.stockToCheck)){
        this.listOfSelectedStocks.push(this.stockToCheck.toUpperCase())
        this.showErrorMessage = false
      }else{
        this.showErrorMessage = true
      }
      console.log(this.listOfSelectedStocks+'')
      this.selectedStock = null
            this.selectedStock = null
    this.stockToCheck  = null  
    if(!(this.listOfSelectedStocks.length <= 0)){
      this.fetchDataOfSelectedStocks();
    }
    },


  deleteFunc(index) {
        console.log("Reached delete function")
    this.showErrorMessage = false
    console.log(this.listOfSelectedStocks[index])
    this.listOfSelectedStocks.splice(this.listOfSelectedStocks.indexOf(this.listOfSelectedStocks[index]),1)
    if(!(this.listOfSelectedStocks.length <= 0)){
      this.fetchDataOfSelectedStocks();
    }else{
      this.updateGraphToEmpty()
    }
  },


   updateGraphToEmpty(){
        this.series = [{
          data: []
        }]
        this.options = {
          xaxis:{
            categories: []
          }
        }
  },
  

  async fetchDataOfSelectedStocks() {
      try {
        console.log(this.listOfSelectedStocks)
        const response = await axios.get(
          "https://cloud.iexapis.com/stable/stock/market/batch?types=quote&token=pk_044db279039d465eb16ff69f5b0ead45&symbols="+this.listOfSelectedStocks
        );
        this.dataOfSelectedStocks = response.data;
        const stockValues = []
        for(let eachSymbol of this.listOfSelectedStocks){
          stockValues.push({
            symbol:eachSymbol,
            value:this.dataOfSelectedStocks[eachSymbol].quote.iexRealtimePrice
          })
        }
        this.values= stockValues.map( eachItem => {return eachItem.value})
        console.log("this.stockValues",stockValues);
        this.series = [{
          data: this.values
        }]
        this.options = {
          xaxis:{
            categories: this.listOfSelectedStocks
          }
        }
        console.log("this.options", this.options)
        console.log("this.series", this.series)
      } catch (error) {
        console.log(error);
      }
    }
  },


  mounted() {
    if(!(this.listOfSelectedStocks.length <= 0)){
      this.fetchDataOfSelectedStocks();
    }
  //   window.setInterval(() => {
  //         this.fetchDataOfSelectedStocks()
  // }, 5000)
  }
}
</script>

<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.column_wrapper {
  column-count: 3;
}
.button {
  background-color: #ee8181;
  border: 2px;
  color: white;
  padding: 5px 10px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 14px;
  margin: 2px 1px;
  cursor: pointer;
  border-radius:5%;
}
.stock-list-wrap{
  display: flex;
  justify-content: center;
}
.stock-list{
  width:20%;
}
.stock-list li{
 display: flex;
 justify-content: space-between;
 padding: 12px 0;
 border-bottom: 1px solid #c0c0c0;
}
.stock-list li span{
  align-self:center;
  margin-right: 1rem;
}
</style>
