<template>
    <el-card class="coinbase-container">
        <div class="box-card">
            <div class="card-header">
                <h2 class="card-text">Currency/Bitcoin Converter</h2>
            </div>
            <div class="control-container">
                <div class="currency-select-container">
                    <h3>Select Currency</h3>
                    <el-select class="currency-selector" 
                        v-model="currencyValue" 
                        placeholder="Select"
                        @change="updateCurrency">
                        <el-option
                            v-for="currency in currencies"
                            :key="currency"
                            :label="currency"
                            :value="currency">
                        </el-option>
                    </el-select>
                </div>
                <div class="currency-select-container">
                    <h3>Bitcoin Buy Price</h3>
                    <h3>{{buyPrice?.amount}}</h3>
                </div>
            </div>
            <div class="control-table">
                <el-table
                    :data="rates"
                    height="350"
                    style="width: 100%">
                    <el-table-column
                        prop="currency"
                        label="Rate"
                        width="300">
                    </el-table-column>
                    <el-table-column
                        prop="amount"
                        label="Conversion Rate"
                        width="300">
                    </el-table-column>
                    <el-table-column
                        prop="address"
                        label="Action">
                        <el-button type="primary">Buy</el-button>
                        <el-button type="danger">Sell</el-button>
                    </el-table-column>
                </el-table>
            </div>
        </div>
    </el-card>
</template>

<script>
import axios from "axios";
import { combineLatest, from, tap } from "rxjs";
export default {
  name: 'CoinBase',
  props: {},
  methods: {
    getApiBaseUrl() {
        return "https://api.coinbase.com/v2"
    },
    formatConverterTable(rates) {
        this.rates = Object.entries(rates).map((rate) => {
            return {
                currency: rate[0],
                amount: rate[1]
            };
        });
    },
    updateData() { 
        const buyPrice$ = from(axios({ method: "GET", "url":  `${this.getApiBaseUrl()}/prices/BTC-${this.currencyValue}/buy`}))
            .pipe(
                tap((result) => this.buyPrice = result.data.data)
            );
        const exchangeRates$ = from(axios({ method: "GET", "url":  `${this.getApiBaseUrl()}/exchange-rates?currency=${this.currencyValue}`}))
            .pipe(
                tap((result) => {
                    this.formatConverterTable(result.data.data.rates);
                    this.currency = result.data.data.currency;
                    this.currencies = Object.keys(result.data.data.rates);
                })
            )

        combineLatest([exchangeRates$, buyPrice$]).subscribe();
    },
    updateCurrency(value) {
        this.currencyValue = value;
        this.updateData();
    }
  },
  data() {
    return {
        rates: [],
        currencies: [],
        currencyValue: 'USD',
        buyPrice: null
    }
  },
  mounted() {
      this.updateData();
  }
}
</script>

<style scoped>
    .coinbase-container {
        width: 50em;
        margin: 0 auto;
    }
    .box-card {
        width: 50em !important;
        margin: 0 auto;
    }
    .card-header {
      display: flex;
      justify-content: space-between;
      text-align: center;
      align-items: center;
    }
    .card-text {
        margin: 1em auto;
    }
    .currency-selector {
        width: 60%;
        margin-bottom: 1em;
    }
    .control-container {
        display: flex;
        align-items: center;
        margin-bottom: 2em;
    }
    .currency-select-container {
        width: 50%;
    }
    .control-table {
        width: 100%;
    }
    .text {
        font-size: 14px;
    }

    .item {
        margin-bottom: 18px;
    }

    .box-card {
        width: 480px;
    }
</style>
