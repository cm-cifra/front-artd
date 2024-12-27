<template>
    <div class="p-5">
      <ul class="flex flex-wrap mx-5 p-2 shadow justify-between max-w-full">
        <li
          v-for="tab in tabs"
          :key="tab"
          :class="['p-5 cursor-pointer', selectedTab === tab ? 'text-yellow-600 border-b-4 border-yellow-600' : 'hover:text-yellow-500']"
          @click="selectedTab = tab"
        >
          {{ tab }}
        </li>
      </ul>
  
      <!-- Conditionally Render Tabs -->
      <div v-if="selectedTab === 'All'" class="flex flex-col">
        <div v-if="allpurchase.length === 0" class="border p-8 text-center m-4 z-20">
          <img
            src="../../assets/icon_empty_cart.png"
            class="h-20 w-20 mx-auto mb-2"
          />
          <p class="text-gray-700">No items yet</p>
          <p class="text-gray-700">My purchased</p>
          <button
            class="flex-none bg-primary-600 rounded-md text-white px-8 py-2 hover:bg-primary-700 duration-300 my-4"
          >
            Перейти к продуктам
          </button>
        </div>
  
        <div class="w-full max-w-full h-[600px] overflow-auto p-5 m-5">
          <div
            v-for="(product, index) in allpurchase"
            :key="product.id"
            class="flex flex-wrap items-center justify-between p-4 border rounded-lg shadow hover:shadow-lg transition mb-4"
            @click="viewProduct(product.id)"
          >
          
            <img
              :src="getImgUrl(product.product_img)"
              :alt="product.name"
              class="w-20 h-20 object-cover rounded-md cursor-pointer"
              @click="viewImg(product.product_img)"
            />
            <!-- Product Details -->
            <div class="flex-1 px-4 min-w-[200px]">
              <h2 class="text-lg font-semibold mb-1">{{ product.name }}</h2>
              <p class="text-gray-600">SKU: {{ product.sku }}</p>
            </div>
            <div class="flex justify-between items-center px-5 mx-5 min-w-[150px]">
              <h1 class="text-black-600">
                Price: {{ changeMonetaryFormat(product.price) }}
              </h1>
            </div>
            <!-- Quantity -->
            <div class="flex items-center gap-4 p-2 rounded min-w-[100px]">
              <h1 class="text-sm font-medium text-gray-600">Quantity:</h1>
              <p>{{ product.quantity }}</p>
            </div>
          </div>
        </div>
      </div>
  
      <!-- Other Tabs -->
      <div v-if="selectedTab === 'ToPay'" id="ToPay">To Pay</div>
      <div v-if="selectedTab === 'ToShip'" id="ToShip">To Ship</div>
      <div v-if="selectedTab === 'ToRecieve'" id="ToRecieve">To Recieve</div>
      <div v-if="selectedTab === 'Complete'" id="Complete">Complete</div>
      <div v-if="selectedTab === 'Cancelled'" id="Cancelled">Cancelled</div>
      <div v-if="selectedTab === 'Return'" id="Return">Return</div>
    </div>
  </template>
  
  <script>
import axios from 'axios';
  export default {
    data() {
      return {
        tabs: ["All", "ToPay", "ToShip", "ToRecieve", "Complete", "Cancelled", "Return"],
        allpurchase: [],
        selectedTab: "All",
      };
    },
    methods: {
      goToProductDetail(productId) {
        this.$router.push({ name: 'order-detail', params: { id: productId } });
      },
      refreshData() {
        this.getItemsFromCart();
      },
      getItemsFromCart() {
        axios
          .get(
            `${process.env.VUE_APP_BASE_URL}/user_carts/get_all/1/50`
          )
          .then((res) => {
            if (res.status === 200) {
              const cartData = res.data[0];
              this.allpurchase = cartData.map((item) => ({
                id: item.id,
                product_id: item.product_id.id,
                name: item.product_id.name,
                sku: item.product_id.sku,
                product_img: item.product_id.product_img,
                price: item.product_id.price || 0,
                quantity: item.quantity,
              }));
            }
          })
          .catch((error) => {
            console.error("Error fetching cart items:", error);
          });
      },  viewProduct(id) {
      this.$router.push("/order-details/" + id);
    },
      getImgUrl(imgPath) {
        return `${process.env.VUE_APP_BASE_URL}/products/get_img/${imgPath}`;
      },
      viewImg(imgPath) {
        window.open(this.getImgUrl(imgPath), "_blank");
      },
      changeMonetaryFormat(val) {
        return `${val}₽`;
      },
    },
    mounted() {
      this.refreshData();
    },
  };
  </script>
  