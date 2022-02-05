<template>
  <div>
    <h1>{{ title }}</h1>
    <apexchart
      align="center"
      :width="getWidthForChart"
      height="300"
      type="bar"
      :options="options"
      :series="series"
    ></apexchart>

    <p>
      Please enter the stock code below. The Stock codes can be found
      <a href="http://eoddata.com/stocklist/NASDAQ.htm" target="_blank">here</a>
    </p>
    <p v-if="showErrorMessage" style="color: red">
      The Stock has already been added
    </p>
    <div class="inputforstockcode">
      <input
        @keyup.enter="addtoarray"
        type="text"
        v-model="stockToCheck"
        placeholder="Enter the Stock code"
      />
      <button class="addbutton" @click="addtoarray">Add</button>
    </div>
    <h3 v-if="listOfSelectedStocks.length > 0">The Selected Stocks are</h3>
    <div class="stock-list-wrap">
      <ul class="stock-list">
        <li v-for="(obs, index) in listOfSelectedStocks" :key="index">
          <span
            ><span>{{ obs }}</span>
            <span>({{ companyNames[index] }})</span></span
          >
          <button class="button" v-on:click="deleteFunc(index)">Delete</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "HelloWorld",

  data() {
    return {
      title: "Welcome to StockChecker",
      listOfSelectedStocks: [],
      values: [],
      dataOfSelectedStocks: [],
      companyNames: [],
      stockToCheck: "",
      selectedStock: "",
      selectedDelete: "",
      showErrorMessage: false,
      hasElementBeenAdded: false,
      options: {
        chart: {
          id: "vuechart-example",
        },
        xaxis: {
          categories: [],
        },
      },
      series: [
        {
          data: [],
          name: "stocks",
        },
      ],
    };
  },
  computed: {
    getWidthForChart() {
      if (this.listOfSelectedStocks.length < 2) {
        return "300px";
      }
      return `${this.listOfSelectedStocks.length * 150}px`;
    },
  },
  methods: {
    addtoarray() {
      this.stockToCheck = this.stockToCheck.trim(" ").toUpperCase();
      if (!this.listOfSelectedStocks.includes(this.stockToCheck)) {
        this.listOfSelectedStocks.push(this.stockToCheck.toUpperCase());
        this.showErrorMessage = false;
      } else {
        this.showErrorMessage = true;
      }
      this.selectedStock = null;
      this.selectedStock = null;
      this.stockToCheck = null;
      if (this.hasElementBeenAdded) {
        this.fetchDataOfSelectedStocks();
      } else {
        this.hasElementBeenAdded = true;
        this.fetchDataOfSelectedStocks();
        window.setInterval(() => {
          this.fetchDataOfSelectedStocks();
        }, 5000);
      }
    },

    deleteFunc(index) {
      this.showErrorMessage = false;
      this.listOfSelectedStocks.splice(index, 1);
      this.companyNames.splice(index, 1);
      if (!(this.listOfSelectedStocks.length <= 0)) {
        this.fetchDataOfSelectedStocks();
      } else {
        this.updateGraphToEmpty();
      }
    },

    updateGraphToEmpty() {
      this.series = [
        {
          data: [],
        },
      ];
      this.options = {
        xaxis: {
          categories: [],
        },
      };
    },

    async fetchDataOfSelectedStocks() {
      try {
        const response = await axios.get(
          "https://cloud.iexapis.com/stable/stock/market/batch?types=quote&token=pk_044db279039d465eb16ff69f5b0ead45&symbols=" +
            this.listOfSelectedStocks
        );
        this.dataOfSelectedStocks = response.data;
        const stockValues = [];

        for (let eachSymbol of this.listOfSelectedStocks) {
          stockValues.push({
            symbol: eachSymbol,
            value: this.dataOfSelectedStocks[eachSymbol].quote.iexRealtimePrice,
            companyname:
              this.dataOfSelectedStocks[eachSymbol].quote.companyName,
          });
        }
        this.companyNames = stockValues.map((eachItem) => {
          return eachItem.companyname;
        });
        this.values = stockValues.map((eachItem) => {
          return eachItem.value;
        });
        this.series = [
          {
            data: this.values,
          },
        ];
        this.options = {
          xaxis: {
            categories: this.listOfSelectedStocks,
          },
        };
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>

<style scoped>
h1,
h2 {
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
  border-radius: 5%;
}
.stock-list-wrap {
  display: flex;
  justify-content: center;
}
.stock-list {
  width: 30%;
}
.stock-list li {
  display: flex;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid #c0c0c0;
}
.stock-list li span {
  align-self: center;
  margin-right: 1rem;
}
.inputforstockcode input {
  width: 30%;
  height: 40px;
  margin-right: 12px;
}
.inputforstockcode button {
}
.inputforstockcode {
  display: flex;
  justify-content: center;
}
.addbutton {
  background-color: #76d176;
  color: white;
  border: none;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  cursor: pointer;
  border-radius: 5px;
  width: 100px;
}
</style>
