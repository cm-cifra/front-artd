<template>
  <div class="flex flex-col">
    
    <!-- Populated Cart -->
    <div  class="mx-36 my-10 flex">
      <div class="m-5">
       

        <!-- Grouped Cart Items -->
        <div v-for="group in allcart" :key="group.date" class="mb-6">
          <h2 class="text-lg font-semibold mb-2">{{ group.date }}</h2>

          <!-- Single Item Display with Expand Option -->
          <div
            v-for="(product, index) in group.items"
            :key="product.id"
            v-show="!group.isExpanded && index === 0"
            @click="toggleExpand(group)"
            class="flex items-center justify-between p-4 border rounded-lg shadow hover:shadow-lg transition mb-4 cursor-pointer"
          >
            <img
              :src="getImgUrl(product.product_img)"
              :alt="product.name"
              class="w-32 h-32 object-cover rounded-md"
            />
            <div class="flex-1 px-4">
              <h2 class="text-lg font-semibold mb-1">{{ product.name }}</h2>
              <p class="text-gray-600">SKU: {{ product.sku }}</p>
              <p class="text-gray-600">Quantity: {{ product.quantity }}</p>
            </div>
            <div class="justify-between items-center px-20 mx-10 flex">
              <h1 class="text-black-600">
                Price: {{ changeMonetaryFormat(product.price) }}
              </h1>
            </div>
          </div>

          <!-- Expandable Section -->
          <div v-if="group.isExpanded">
            <div
              v-for="product in group.items"
              :key="product.id"
              class="flex items-center justify-between p-4 border rounded-lg shadow hover:shadow-lg transition mb-4"
            >
              <img
                :src="getImgUrl(product.product_img)"
                :alt="product.name"
                class="w-32 h-32 object-cover rounded-md cursor-pointer"
                @click="viewImg(product.product_img)"
              />
              <div class="flex-1 px-4">
                <h2 class="text-lg font-semibold mb-1">{{ product.name }}</h2>
                <p class="text-gray-600">SKU: {{ product.sku }}</p>
                <p class="text-gray-600">Quantity: {{ product.quantity }}</p>
                <button
                  @click="removeProduct(product.id)"
                  class="px-4 py-2 mt-3 bg-yellow-600 text-white rounded hover:bg-yellow-500"
                >
                  Remove
                </button>
              </div>
              <div class="justify-between items-center px-20 mx-10 flex">
                <h1 class="text-black-600">
                  Price: {{ changeMonetaryFormat(product.price) }}
                </h1>
              </div>
            </div>

            <!-- Collapse Button -->
            <button
              @click="toggleExpand(group)"
              class="bg-gray-600 text-white px-8 py-2 mt-4 rounded hover:bg-gray-700"
            >
              Collapse
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";

export default {
  data() {
    return {
      allcart: [],
    };
  },
  computed: {
    totalPrice() {
      return this.allcart.reduce((total, group) => {
        return total + group.items.reduce((sum, product) => sum + product.price * product.quantity, 0);
      }, 0);
    },
    totalQty() {
      return this.allcart.reduce((total, group) => {
        return total + group.items.reduce((sum, product) => sum + product.quantity, 0);
      }, 0);
    },
  },
  methods: {
    goToUrl(val) {
      this.$router.push(val);
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
            const groupedCart = {};

            // Group items by datetime_added
            cartData.forEach((item) => {
              const date = item.datetime_added.split("T")[0]; // Extract the date part
              if (!groupedCart[date]) {
                groupedCart[date] = [];
              }
              groupedCart[date].push({
                id: item.id,
                product_id: item.product_id.id,
                name: item.product_id.name,
                sku: item.product_id.sku,
                product_img: item.product_id.product_img,
                price: item.product_id.price || 0,
                quantity: item.quantity,
              });
            });

            // Convert groupedCart into an array for easier template rendering
            this.allcart = Object.entries(groupedCart).map(([date, items]) => ({
              date,
              items,
              isExpanded: false,
            }));
          }
        })
        .catch((error) => {
          console.error("Error fetching cart items:", error);
        });
    },
    toggleExpand(group) {
      group.isExpanded = !group.isExpanded;
    },
    PurchaseCart() {
      const userId = this.$store.state.user.id;

      axios
        .get(
          `${process.env.VUE_APP_BASE_URL}/user_carts/find_by_type/${userId}/1/1/20`
        )
        .then((res) => {
          if (res.status === 200) {
            const products = res.data[0];
            const updatePromises = [];

            products.forEach((product) => {
              axios.put(
                `${process.env.VUE_APP_BASE_URL}/user_carts/edit/${product.id}`,
                {
                  type: 2,
                }
              );
            });

            Promise.all(updatePromises)
              .then(() => {
                console.log("Cart items updated successfully.");
                this.refreshData();
              })
              .catch((error) => {
                console.error("Error updating cart items:", error);
              });
          } else {
            console.log("No items to update.");
          }
        })
        .catch((error) => {
          console.error("Error fetching cart items:", error);
        });
    },
    updateQuantity(index, delta) {
      const product = this.allcart[index];
      const newQuantity = product.quantity + delta;
      if (newQuantity > 0) {
        product.quantity = newQuantity;

        axios
          .put(
            `${process.env.VUE_APP_BASE_URL}/user_carts/edit/${product.id}`,
            {
              quantity: newQuantity,
            }
          )

          .then(() => {
            console.log("Quantity updated successfully");
          })
          .catch((error) => {
            console.error("Error updating quantity:", error);
          });
      }
    },
    removeProduct(product) {
      axios
        .post(`${process.env.VUE_APP_BASE_URL}/user_carts/delete`, {
          id: product,
        })
        .then(() => {
          console.log("Product removed successfully");
        })
        .catch((error) => {
          console.error("Error removing product:", error);
        });
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
