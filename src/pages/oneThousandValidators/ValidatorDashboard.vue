<template>
  <div id="validatorStatus">
    <md-progress-bar md-mode="query" v-if="showProgressBar"></md-progress-bar>
    <p v-if="showProgressBar">Loading Nominator Info...</p>
    
    <div class="info-panel md-layout">
      <div class="info-text md-layout-item">
        <div class=md-title>{{this.displayName || this.stash}}</div>
        <md-divider/>
        <md-table>
          <md-table-row>
            <md-table-cell>Stash ID</md-table-cell>
            <md-table-cell>
            <Identicon class="ident-icon" @click.native="copy(stash)"
              :size="32"
              :theme="'polkadot'"
              :value="stash"
            />
            <div id="validator-id">{{stash}}</div>
            </md-table-cell>
          </md-table-row>
          <md-table-row>
            <md-table-cell class="validator-info">Total Nomination Amount </md-table-cell> 
            <md-table-cell style="color: #5f6368">{{inactiveKSM}} {{coinName}}</md-table-cell>
          </md-table-row>
          <md-table-row>
            <md-table-cell class="validator-info">Commission </md-table-cell> 
            <md-table-cell style="color: #5f6368">{{eraCommission.toFixed(1)}}%</md-table-cell>
          </md-table-row>
        </md-table>
      </div>
      <apexchart id="nomination-trend" ref="nomination-trend" width="500" type="line" :options="options" :series="series" class="md-layout-item"></apexchart>
      <apexchart id="apy-trend" ref="apy-trend" width="500" type="line" :options="apyOptions" :series="apySeries" class="md-layout-item"></apexchart>
    </div>
    <div class="nominator-list">
      <div class="nominator" v-for="(nominator, index) in nominators" :key="index">
        <Identicon class="ident-icon" @click.native="copy(nominator.address)"
            :size="32"
            :theme="'polkadot'"
            :value="nominator.address"
        />
        <div class="detail">
          <div class="stash-id">{{nominator.address.substr(0, 5)}}......{{nominator.address.substr(nominator.address.length - 5)}}</div>
          <div class="bonding" v-if="balances[index] !== null && balances[index] !== undefined">
            {{(balances[index] / decimal).toFixed(3)}} {{coinName}}
          </div>
          <div class="bonding" v-if="balances[index] === null || balances[index] === undefined">
            ? {{coinName}}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const Yaohsin = require('../../scripts/yaohsin');
const constants = require('../../scripts/constants');
import Identicon from '@polkadot/vue-identicon';
export default {
  name: 'validatorStatus',
  data: function() {
    return {
      nominators: [],
      balances: [],
      displayName: '',
      showProgressBar: false,
      inactiveKSM: 0,
      stash: "",
      apyTrend: [],
      eraCommission: 0,
      options: {
        chart: {
          id: 'vuechart-example'
        },
        xaxis: {
          categories: this.xaxisCatagory,
          trim: true,
          hideOverlappingLabels: true,
        },
        yaxis: [
          {
            min: 0,
            labels: {
              formatter: function(val) {
                return val.toFixed(0);
              }
            },
            title: {
              text: "Nominator Count",
              style: {
                color: '#008FFB',
              }
            },
            seriesName: 'Nominator Count'
          },
          {
            min: 0,
            show: false,
            seriesName: 'Nominator Count'
          },
          {
            opposite: true,
            min: 0,
            max: 100,
            title: {
              text: "Commission (%)",
              style: {
                color: '#FEB019',
              }
            },
            seriesName: 'Commission'
          }
        ]
      },
      series: [{
        name: 'Nominator Count',
        data: this.nominatorCounts
      }, {
        name: 'Nominated',
        data: this.exposures,
      }, {
        name: 'Commission',
        data: this.commisions,
      }],
      apyOptions: {
        chart: {
          id: 'vuechart-example'
        },
        xaxis: {
          categories: this.xaxisCatagory,
          trim: true,
          hideOverlappingLabels: true,
        },
        yaxis: [
          {
            min: 0,
            labels: {
              formatter: function(val) {
                return val.toFixed(1);
              }
            },
            title: {
              text: "APY(%)",
              style: {
                color: '#008FFB',
              }
            },
            seriesName: 'APY'
          },
        ]
      },
      apySeries: [{
        name: 'APY',
        data: this.apyTrend
      }]
    }
  },
  mounted: async function() {
    this.showProgressBar = true;
    this.yaohsin = new Yaohsin();
    this.stash = this.$route.query.stash;
    this.coinName = this.$route.query.coin;
    console.log(this.$route);
    const validatorHistory = await this.yaohsin.getValidatorStatus(this.stash, {coin: this.coinName});
    this.xaxisCatagory= [];
    this.exposures = [];
    this.nominatorCounts = [];
    this.commissions = [];
    this.apyTrend = [];
    const data = validatorHistory.data[0];
    if(data.identity !== undefined) {
      console.log(data.identity);
      if(data.identity.display !== null) {
        this.displayName = data.identity.display;
      }
    }
    data.info.forEach((eraData)=>{
      this.xaxisCatagory.push(eraData.era.toString());
      this.nominatorCounts.push(eraData.nominators.length);
      this.exposures.push(eraData.exposure.others.length);
      this.commissions.push(eraData.commission);
      this.nominators = eraData.nominators.sort((a, b)=>{
        if(a.balance.lockedBalance > b.balance.lockedBalance) {
          return -1;
        } else if(a.balance.lockedBalance < b.balance.lockedBalance) {
          return 1;
        }
        return 0;
      });
      this.apyTrend.push(eraData.apy * 100);
    });
    this.eraCommission = this.commissions[this.commissions.length - 1];
    if(this.eraCommission < 0.0001) {
      this.eraCommission = 0;
    }
    this.updateSeriesLine();
    this.balances = this.nominators.map((nominator) => {
      return nominator.balance.lockedBalance;
    });

    this.inactiveKSM = this.balances.reduce((acc, b)=>{
      if(b === null) {
        return acc;
      }
      acc += parseFloat((parseFloat(b / constants.KUSAMA_DECIMAL)).toFixed(3));
      return acc;
    }, 0);
    this.inactiveKSM = this.inactiveKSM.toFixed(3);
    this.showProgressBar = false;
  },
  methods: {
    updateSeriesLine() {
      this.$refs['nomination-trend'].updateSeries([{
        data: this.nominatorCounts
      }, {
        data: this.exposures
      }, {
        data: this.commissions
      }], false, true);
      this.options = {
        xaxis: {
          categories: this.xaxisCatagory
        }
      };
      this.$refs['apy-trend'].updateSeries([{
        data: this.apyTrend,
      }], false, true);
      this.apyOptions = {
        xaxis: {
          categories: this.xaxisCatagory,
          title: {
            text: 'Era'
          }
        }
      };
    },
    copy: function(nominator) {
      this.$copyText(nominator);
    },
  },
  computed: {
    decimal: function() {
      if(this.coinName === 'KSM') {
        return constants.KUSAMA_DECIMAL;
      } else if(this.coinName === 'DOT') {
        return constants.POLKADOT_DECIMAL;
      }
      return constants.KUSAMA_DECIMAL;
    }
  },
  components: {
    Identicon
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
#validatorStatus {
}
.nominator-list {
  text-align: left;
  padding-top: 20px;
  padding-left: 20px;
}
.info-text {
  vertical-align: top;
  display: inline-block;
}
.validator-info {
  vertical-align: top;
}
.info-panel {
  padding-top: 30px;
  text-align: left;
  vertical-align: top;
}
#nomination-trend {
  display: inline-block;
  padding-right: 30px;
}
.nominator {
  display: inline-block;
  text-align: left;
  overflow:auto;
  min-width: 150px;
}
.ident-icon {
  display: inline-block;
  cursor: copy;
}
.bonding {
  color: #5f6368;
  font-size: 8px;
}
.detail {
  display: inline-block;
  font-size: 6px;
}
#validator-id {
  display: inline-block;
  font-size: 10px;
  vertical-align: super;
}
</style>