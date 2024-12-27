<template>
  <div class="flex h-screen">
    <Sidebar />
    <div class="flex-1 flex flex-col">
      <Header />

      <AccessoriesList />
    </div>
  </div>
</template>

<script>
import Header from "../component/header.vue";

import Sidebar from "../component/sidebar.vue";
import MainContent from "../component/content.vue";

import AccessoriesList from "./AccessoriesList.vue";
export default {
  components: {
    Header,
    Sidebar,
    MainContent,
    AccessoriesList,
  },
};
</script>
