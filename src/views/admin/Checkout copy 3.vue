<template>
  <div class="flex flex-col">
    <!-- Populated Cart -->
    <div class="mx-36 my-10 flex">
      <div class="m-5">
        <!-- Grouped Cart Items -->
        <div
          v-for="group in allcart"
          :key="group.date"
          class="mb-6 border rounded-lg shadow hover:shadow-lg transition p-5"
        >
          <div class="flex justify-between">
            <h2 class="text-lg font-semibold mb-2">{{ group.date }}</h2>

            <div class=" ">
              <button
                @click="toggleExpand(group)"
                class="  text-black  py-2 mt-4 rounded hover:text-yellow-600 mx-2 px-2 text-2xl"
              >
                <i class="pi pi-window-maximize"></i>
              </button>
              <!-- View Button (Redirect to Product Page with UserID and Date) -->
              <button
                @click="viewGroupDetails(group.user_id, group.date)"
                class="text-blue-500 hover:text-blue-700  text-2xl mx-2 "
              >
                <i class="pi pi-eye"></i>
              </button>
            </div>
          </div>

          <!-- Single Item Display with Expand Option -->
          <div
            v-for="(product, index) in group.items"
            :key="product.id"
            v-show="!group.isExpanded && index === 0"
            @click="toggleExpand(group)"
            class="flex items-center justify-between p-4 transition mb-4 cursor-pointer"
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
               
              </div>
              <div class="justify-between items-center px-20 mx-10 flex">
                <h1 class="text-black-600">
                  Price: {{ changeMonetaryFormat(product.price) }}
                </h1>
              </div>
            </div>
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
        return (
          total +
          group.items.reduce(
            (sum, product) => sum + product.price * product.quantity,
            0
          )
        );
      }, 0);
    },
    totalQty() {
      return this.allcart.reduce((total, group) => {
        return (
          total +
          group.items.reduce((sum, product) => sum + product.quantity, 0)
        );
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
        .get(`${process.env.VUE_APP_BASE_URL}/user_carts/get_all/1/50`)
        .then((res) => {
          if (res.status === 200) {
            const cartData = res.data[0];
            const groupedCart = {};

            // Group items by datetime_added
            cartData.forEach((item) => {
              const date = item.datetime_added; // Extract the date part
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
                user_id: item.user_id, // Add user_id to group
                date: item.datetime_added,
              });
            });

            // Convert groupedCart into an array for easier template rendering
            this.allcart = Object.entries(groupedCart).map(([date, items]) => ({
              date,
              items,
              isExpanded: false,
              user_id: items[0].user_id, // Set user_id for each group
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
    viewGroupDetails(userId, date) {
      // Redirect to a new page with user_id and date as query params
      console.log("User ID:", userId.id);
      console.log("Date:", date);

      this.$router.push({
        path: "/check",
        query: { user_id: userId.id, date: date },
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
