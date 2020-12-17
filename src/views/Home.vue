<template>
    <article class="container content" style="margin-bottom: 200px">
        <h1 class="has-text-centered is-size-1 has-text-primary" style="margin-top: 50px;">Gas Sweet Spot 🎯</h1>
        <hr/>
        <div class="columns">
            <div class="column">
                <line-chart :chart-data="datacollection" :options="options"></line-chart>
            </div>
            <div class="column">
                <h2>Welcome to the knowledge of the crowd</h2>

                <h3 class="is-family-monospace has-text-grey-light">{{ totalTxs }} txs crunched</h3>

                <div class="notification is-warning" v-if="winning">
                    The crowd thinks <span class="has-text-weight-semibold">{{ winning.key }}ish gwei</span> would get your tx the new few blocks...
                </div>

                <b-field label="Modulo" message="Find the sweet spot of gas ranges for your analysis">
                    <b-slider v-model="modulo" :min="5" :max="50" :step="5"></b-slider>
                </b-field>

                <b-field label="Start over?" message="When you hit the site we start processing txs">
                    <button @click="reset" class="button is-primary is-outlined">Reset stats</button>
                </b-field>

                <hr/>

                <blockquote>
                    We are using Blocknative's the Gas Distribution feed. This feed provides the distribution of the top gas prices in the mempool right now.
                    <br/>
                    This enables us to build a dynamic dataset to guess-timate a good gas price to get in the next few blocks visually.
                </blockquote>
            </div>
        </div>


    </article>
</template>

<script>
  import LineChart from '../line-chart';
  import axios from 'axios';
  import _ from 'lodash';

  export default {
    components: {
      LineChart
    },
    data() {
      return {
        rawData: [],
        processedData: {},
        processedDataRounded: {},
        labelSet: [],
        dataSet: [],
        datacollection: {},
        winning: null,
        totalTxs: null,
        modulo: 25,
        options:  {
          scales: {
          xAxes: [ {
            display: true,
            scaleLabel: {
              display: true,
              labelString: 'Gas price in Gwei'
            },
          } ],
            yAxes: [ {
            display: true,
            scaleLabel: {
              display: true,
              labelString: '# of transactions'
            }
          } ]
        }
      },
      };
    },
    methods: {
      reset() {
        this.rawData = [];
        console.log('reset');
      },
      pollData() {
        this.polling = setInterval(() => {
          axios.get(
            'https://api.blocknative.com/gasprices/distribution',
            {
              headers: {
                'Authorization': `6f48615a-3c92-4adb-b593-69f10315f3ab`
              }
            })
            .then(({data}) => {

              this.rawData = this.rawData.concat(data.topNDistribution.distribution);

              this.processedData = _.reduce(this.rawData, (obj, rd) => {
                  obj[`${rd[0]}`] = obj[`${rd[0]}`] ? obj[`${rd[0]}`] + rd[1] : rd[1];
                  return obj;
                },
                {}
              );

              this.processedDataRounded = _.reduce(this.processedData, (obj, value, key) => {
                const newKey = parseInt(parseInt(key) / this.modulo)
                obj[`${newKey * this.modulo}`] = obj[`${newKey * this.modulo}`] ? obj[`${newKey * this.modulo}`] + value : value;
                return obj;
              }, {});

              this.totalTxs = _.reduce(this.processedData, (sum, value, key) => sum + value, 0)

              this.processedDataRounded = _.pickBy(this.processedDataRounded, (value, key) => value > 20);

              this.winning = _.last(_.orderBy(Object.keys(this.processedDataRounded).map(key => ({ key, value: this.processedDataRounded[key] })), 'value'));

              this.labelSet = _.keys(this.processedDataRounded);
              this.dataSet = _.values(this.processedDataRounded);

              this.datacollection = {
                labels: this.labelSet,
                datasets: [
                  {
                    label: `top gas prices in the mempool`,
                    backgroundColor: '#7957d5',
                    data: this.dataSet,
                    pointRadius: 0
                  }
                ]
              };
            });
        }, 2000);
      },
    },
    beforeDestroy() {
      clearInterval(this.polling);
    },
    created() {
      this.pollData();
    }
  };
</script>

<style>
    .small {
        max-width: 600px;
        margin: 150px auto;
    }
</style>