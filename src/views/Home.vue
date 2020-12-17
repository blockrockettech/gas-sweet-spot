<template>
    <div class="home">
        <trend
                :data="trendData"
                gradientDirection="top"
                :gradient="['#f94144', '#ffe74c', '#42b983']"
                :padding="10.2"
                :radius="12"
                :stroke-width="9.2"
                stroke-linecap="round"
                auto-draw
                smooth>
        </trend>
        <h2>MaxPrice Range</h2>
        <h1>{{ minMaxPrice }} - {{ maxMaxPrice}}</h1>
    </div>
</template>

<script>
  import axios from 'axios';

  export default {
    name: 'Home',
    data() {
      return {
        maxMaxPrice: null,
        minMaxPrice: null,
        trendData: [],
        polling: null
      };
    },
    methods: {
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
              this.trendData.push(data.maxPrice);
              if (!this.maxMaxPrice || data.maxPrice > this.maxMaxPrice) {
                this.maxMaxPrice = data.maxPrice;
              }
              if (!this.minMaxPrice || data.maxPrice < this.minMaxPrice) {
                this.minMaxPrice = data.maxPrice;
              }

              console.log(data.topNDistribution.distribution)
            });
        }, 1500);
      }
    },
    beforeDestroy() {
      clearInterval(this.polling);
    },
    created() {
      this.pollData();
    }
  };
</script>
