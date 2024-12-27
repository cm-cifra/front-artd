<template>
  <div class="p-5">
    <ul class="flex flex-wrap mx-5 p-2 shadow justify-between max-w-full">
      <li
        v-for="tab in tabs"
        :key="tab"
        :class="[
          'p-5 cursor-pointer',
          selectedTab === tab
            ? 'text-yellow-600 border-b-4 border-yellow-600'
            : 'hover:text-yellow-500',
        ]"
        @click="selectedTab = tab"
      >
        {{ tab }}
      </li>
    </ul>

    <!-- Conditionally Render Tabs -->
    <div v-if="selectedTab === 'All'" class="flex flex-col">
      <div
        v-if="allpurchase.length === 0"
        class="border p-8 text-center m-4 z-20"
      >
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

      <div class="w-full max-w-full h-[600px]  p-5 m-5">
        <div class="w-full max-w-full h-[600px]   p-5 m-5">
  <table class="min-w-full table-auto border-collapse bg-white  ">
    <thead class="bg-gray-700 text-white  ">
      <tr class="border-b">
        <th class="px-4 py-2 text-left">Product Image</th>
 
        <th class="px-4 py-2 text-left">SKU</th>
        <th class="px-4 py-2 text-left">User</th>
        <th class="px-4 py-2 text-left">Date</th>
        <th class="px-4 py-2 text-left">Priority</th>
        <th class="px-4 py-2 text-left">Price</th>
        <th class="px-4 py-2 text-left">Status</th>
        <th class="px-4 py-2 text-left">Quantity</th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="(product, index) in allpurchase"
        :key="product.id"
        @click="viewProduct(product.id)"
        class="border-b hover:bg-gray-100 cursor-pointer"
      >
        <td class="px-4 py-2">
          <img
            :src="getImgUrl(product.product_img)"
            :alt="product.name"
            class="w-20 h-20 object-cover rounded-md"
            @click="viewImg(product.product_img)"
          />
        </td>
 
        <td class="px-4 py-2">{{ product.sku }}</td>
        <td class="px-4 py-2">{{ product.user.fname }} {{ product.user.lname }}</td>
        <td class="px-4 py-2">{{ product.date }}</td>
        <td class="px-4 py-2">Priority</td>
        <td class="px-4 py-2">{{ product.price }}</td>
        <td class="px-4 py-2">Status</td>
        <td class="px-4 py-2">{{ product.quantity }}</td>
      </tr>
    </tbody>
  </table>
</div></div>

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
import axios from "axios";
export default {
  data() {
    return {
      tabs: [
        "All",
        "ToPay",
        "ToShip",
        "ToRecieve",
        "Complete",
        "Cancelled",
        "Return",
      ],
      allpurchase: [],
      selectedTab: "All",
    };
  },
  methods: {
    goToProductDetail(productId) {
      this.$router.push({ name: "order-detail", params: { id: productId } });
    },
    formatDate(dateStr) {
      const date = new Date(dateStr);
      const month = date.getMonth() + 1; // Months are zero-indexed
      const day = date.getDate();
      const year = date.getFullYear();
      const hours = date.getHours();
      const minutes = String(date.getMinutes()).padStart(2, "0");
      const seconds = String(date.getSeconds()).padStart(2, "0");

      return `${month}/${day}/${year}, ${hours}:${minutes}:${seconds}`;
    },
    refreshData() {
      this.getItemsFromCart();
    },
    getItemsFromCart() {
      axios
        .get(`${process.env.VUE_APP_BASE_URL}/user_carts/get_all/1/50`)
        .then((res) => {
          if (res.status === 200) {
            const cartData = res.data[0];
            this.allpurchase = cartData.map((item) => ({
              id: item.id,
              product_id: item.product_id.id,
              name: item.product_id.name,
              sku: item.product_id.sku,
              user:item.user_id,
              product_img: item.product_id.product_img,
              price: item.product_id.price || 0,
              quantity: item.quantity,
              date: this.formatDate(item.datetime_added),
            }));
          }
        })
        .catch((error) => {
          console.error("Error fetching cart items:", error);
        });
    },
    viewProduct(id) {
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
