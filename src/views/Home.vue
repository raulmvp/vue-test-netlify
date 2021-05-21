<template>
  <div>
    <pacman-loader
      :loading="isLoading"
      :color="'#00CC66'"
      :size="100"
    ></pacman-loader>
    <px-assets-table v-if="!isLoading" :assets="assets" />
  </div>
</template>
<script>
import api from "@/api";
import PxAssetsTable from "@/components/PxAssetsTable";

export default {
  name: "Home",
  components: {
    PxAssetsTable,
  },
  data() {
    return {
      isLoading: true,
      assets: [],
    };
  },
  created() {
    console.log(this.assets);
    this.isLoading = true;
    const rep = api
      .getAssets()
      .then((assets) => {
        this.assets = assets;
        console.log(rep, this.assets, assets);
      })
      .finally(() => (this.isLoading = !this.isLoading));
  },
};
</script>
