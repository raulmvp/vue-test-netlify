<template>
  <table class="w-100">
    <thead>
      <tr class="bg-gray-100 border-b-2 border-gray-400">
        <th></th>
        <th :class="{ up: this.sortOrder === 1, down: this.sortOrder === -1 }">
          <span class="underline cursor-pointer" @click="changeSortOrder">
            Ranking</span
          >
        </th>
        <th>Nombre</th>
        <th>Precio</th>
        <th>Cap. de Mercado</th>
        <th>Variación 24hs</th>
        <td class="hidden sm:block">
          <input
            class="
              bg-gray-100
              focus:outline-none
              border-b border-gray-400
              py-2
              px-4
              block
              w-full
              appearance-none
              leading-normal
            "
            id="filter"
            placeholder="Buscar..."
            type="text"
            v-model="filter"
          />
        </td>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="coin in filteredAssets"
        :key="coin.id"
        class="border-b border-gray-200 hover:bg-gray-100 hover:bg-orange-100"
      >
        <td>
          <img
            class="w-6 h-6"
            :src="`https://static.coincap.io/assets/icons/${coin.symbol.toLowerCase()}@2x.png`"
            alt="coin.name"
          />
        </td>
        <td>#{{ coin.rank }}</td>
        <td>
          <router-link
            class="hover:underline text-green-600"
            :to="{ name: 'coin-detail', params: { id: coin.id } }"
          >
            {{ coin.name }}
          </router-link>
          <small class="ml-1 text-gray-500">
            {{ coin.symbol }}
          </small>
        </td>
        <td>{{ coin.priceUsd | dollar }}</td>
        <td>{{ coin.marketCapUsd | dollar }}</td>
        <td
          :class="{
            'text-red-600': coin.changePercent24Hr < 0,
            'text-green-600': coin.changePercent24Hr > 0,
          }"
        >
          {{ coin.changePercent24Hr | percent }}
        </td>
        <td class="hidden sm:block">
          <px-button @custom-click="goToCoin(coin.id)">
            <span>Detalle</span>
          </px-button>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script>
import PxButton from "@/components/PxButton";
export default {
  name: "PxAssetsTable",
  data() {
    return {
      filter: "",
      sortOrder: 1,
    };
  },
  components: { PxButton },
  props: {
    assets: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    filteredAssets() {
      const altOrder = this.sortOrder === 1 ? -1 : 1;
      return this.assets
        .filter(
          (a) =>
            a.symbol.toLowerCase().includes(this.filter.toLowerCase()) ||
            a.name.toLowerCase().includes(this.filter.toLowerCase())
        )
        .sort((a, b) => {
          if (parseInt(a.rank) > parseInt(b.rank)) {
            return this.sortOrder;
          }
          return altOrder;
        });
    },
  },
  methods: {
    changeSortOrder(){
      this.sortOrder = this.sortOrder === 1 ? -1 : 1;
    },
    goToCoin(id) {
      console.log(id);
      this.$router.push({ name: "coin-detail", params: { id: id } });
    },
  },
};
</script>

<style scoped>
.up::before {
  content: "👆";
}

.down::before {
  content: "👇";
}

td {
  padding: 20px 0px;
  font-size: 0.6rem;
  text-align: center;
}

th {
  padding: 5px;
  font-size: 0.6rem;
}

@media (min-width: 640px) {
  td,
  th {
    padding: 20px;
    font-size: 1rem;
  }

  th {
    padding: 12px;
  }
}
</style>
