<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <pacman-loader
        :loading="isLoading"
        :color="'#00CC66'"
        :size="100"
      ></pacman-loader>
    </div>
    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            :src="`https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`"
            class="w-20 h-20 mr-5"
            :alt="asset.name"
          />
          <h1 class="text-5xl text-green-500">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500"> {{ asset.symbol }} </small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span> #{{ asset.rank }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio actual</b>
              <span> {{ asset.priceUsd | dollar }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más bajo</b>
              <span> {{ min | dollar }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más alto</b>
              <span> {{ max | dollar }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio Promedio</b>
              <span> {{ avg | dollar }} </span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variación 24hs</b>
              <span
                :class="
                  asset.changePercent24Hr > 0
                    ? 'text-green-500'
                    : 'text-red-500'
                "
              >
                {{ asset.changePercent24Hr | percent }}
              </span>
            </li>
          </ul>
        </div>

        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            class="
              bg-green-500
              hover:bg-green-700
              text-white
              font-bold
              py-2
              px-4
              rounded
            "
            @click="toggleConverter"
          >
            {{
              dollarToBitcoin
                ? `USD a ${asset.symbol}`
                : `${asset.symbol} a USD`
            }}
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                id="convertValue"
                type="number"
                :placeholder="`Valor en ${
                  dollarToBitcoin ? 'USD' : asset.symbol
                }`"
                class="
                  text-center
                  bg-white
                  focus:outline-none
                  focus:shadow-outline
                  border border-gray-300
                  rounded-lg
                  py-2
                  px-4
                  block
                  w-full
                  appearance-none
                  leading-normal
                "
                v-model="valorConvertido"
              />
            </label>
          </div>

          <span class="text-xl">
            {{ converResult }} {{ dollarToBitcoin ? `${asset.symbol}` : `USD` }}
          </span>
        </div>
      </div>
      <line-chart
        class="my-10"
        :colors="['orange']"
        :min="min"
        :max="max"
        :data="history.map((h) => [h.date, parseFloat(h.priceUsd).toFixed(2)])"
      />

      <h3 class="text-xl my-10">Mejores ofertas de cambio</h3>
      <table>
        <tr
          v-for="market in markets"
          :key="`${market.exchangeId}-${market.priceUsd}`"
          class="border-b"
        >
          <td>
            <b> {{ market.exchangeId }} </b>
          </td>
          <td>{{ market.priceUsd | dollar }}</td>
          <td>{{ market.baseSymbol }} / {{ market.quoteSymbol }}</td>
          <td>
            <px-button
              :is-loading="market.isLoading || false"
              v-if="!market.url"
              @custom-click="getWebsite(market)"
            >
              <slot>Obtener link!</slot>
            </px-button>
            <a v-else class="hover:underline text-green-600" target="_blanck">
              {{ market.url }}
            </a>
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
import api from "@/api";
import PxButton from "@/components/PxButton.vue";
export default {
  components: { PxButton },
  name: "CoinDetail",
  data() {
    return {
      asset: {},
      history: {},
      markets: {},
      isLoading: true,
      dollarToBitcoin: true,
      valorConvertido: null,
    };
  },

  watch: {
    $route() {
      this.getCoin();
    },
  },

  computed: {
    converResult() {
      if (!this.valorConvertido) {
        return 0;
      }
      const result = this.dollarToBitcoin
        ? this.valorConvertido / this.asset.priceUsd
        : this.valorConvertido * this.asset.priceUsd;
      return result.toFixed(4);
    },
    min() {
      return Math.min(
        ...this.history.map((item) => parseFloat(item.priceUsd).toFixed(2))
      );
    },
    max() {
      return Math.max(
        ...this.history.map((item) => parseFloat(item.priceUsd).toFixed(2))
      );
    },
    avg() {
      return (
        this.history.reduce((a, b) => a + parseFloat(b.priceUsd), 0) /
        this.history.length
      );
    },
  },
  created() {
    this.getCoin();
  },

  methods: {
    toggleConverter() {
      this.dollarToBitcoin = !this.dollarToBitcoin;
    },
    getWebsite(exchange) {
      this.$set(exchange, "isLoading", true);
      return api
        .getExchange(exchange.exchangeId)
        .then((res) => {
          this.$set(exchange, "url", res.exchangeUrl);
        })
        .finally(() => this.$set(exchange, "isLoading", false));
    },
    getCoin() {
      this.isLoading = true;
      const id = this.$route.params.id;
      Promise.all([
        api.getAsset(id),
        api.getAssetHistory(id),
        api.getMarkets(id),
      ])
        .then(([asset, history, markets]) => {
          this.asset = asset;
          this.history = history;
          this.markets = markets;
        })
        .finally(() => (this.isLoading = !this.isLoading));
    },
  },
};
</script>

<style scoped>
td {
  padding: 10px;
  text-align: center;
}
</style>
