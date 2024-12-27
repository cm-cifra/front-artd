<template>
  <div class="justify-center align-middle">
    <h1 class="text-2xl text-center">Upload Product in bulk</h1>

    <div class="mb-4 p-2">
      <label for="bulk-type" class="block text-gray-700 text-sm font-bold mb-2">
        Select Category:
      </label>
      <select
        id="bulk-type"
        v-model="selectedComponent"
        class="border rounded py-2 px-3 text-gray-700 w-56"
      >
        <option value="" disabled>Select Category</option>
        <option value="BulkAccessories">Accessories</option>
        <option value="BulkCountertopSink">counter top раковина на столешнице</option>
        <option value="BulkDesibledBaths">Desibled Baths Ванны для инвалидов</option>
        <option value="BulkKitchenSink">Kitchen Sink Кухонная мойка</option>
        <option value="BulkMirrors">Mirrors Зеркало</option>
        <option value="BulkMixers">Mixers Смеситель</option>
        <option value="BulkPedestals">Pedestals пьедестал</option>
        <option value="BulkShower">Shower Душевые системы</option>
        <option value="BulkMirrorsCabinet"> Mirrors Cabinet Душевые системы</option>
      </select>
    </div>

    <div v-if="selectedComponent">
      <component :is="selectedComponent" />
    </div>
  </div>
</template>

<script>
import BulkAccessories from "./BulkAccessories.vue";
import BulkCountertopSink from "./BulkCountertopSink.vue";
import BulkDesibledBaths from "./BulkDesibledBaths.vue";
import BulkKitchenSink from "./BulkKitchenSink.vue";
import BulkMirrors from "./BulkMirrors.vue";
import BulkMixers from "./BulkMixers.vue";
import BulkPedestals from "./BulkPedestals.vue";
import BulkShower from "./BulkShower.vue";
import BulkMirrorsCabinet from "./BulkMirrorsCabinet.vue";
export default {
  data() {
    return {
      selectedComponent: "",
    };
  },
  components: {
    BulkAccessories,
    BulkCountertopSink,
    BulkDesibledBaths,
    BulkKitchenSink,
    BulkMirrors,
    BulkMixers,
    BulkPedestals,
    BulkShower,
    BulkMirrorsCabinet,
  },
};
</script>
