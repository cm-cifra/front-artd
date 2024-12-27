<template>
  <div class="flex flex-col">
    <div class="w-full">
      <p class="m-4 font-bold text-xl">My Shopping Cart</p>
    </div>

    <!-- Empty Cart -->
    <div v-if="allcart.length === 0" class="border p-8 text-center m-4">
      <img
        src="../../assets/icon_empty_cart.png"
        class="h-20 w-20 mx-auto mb-2"
      />
      <p class="text-gray-700">No items yet</p>
      <p class="text-gray-700">
        Start adding now by selecting products on our website
      </p>
      <button
        class="flex-none bg-primary-600 rounded-md text-white px-8 py-2 hover:bg-primary-700 duration-300 my-4"
        @click="goToUrl('/products')"
      >
        Перейти к продуктам
      </button>
    </div>

    <!-- Populated Cart -->
    <div v-else class="mx-36 my-10 flex">
      <div class="m-5">
        <div><h1 class="text-2xl p-5">My Cart</h1></div>

        <!-- Grouped Cart Items -->
        <div v-for="group in allcart" :key="group.date" class="mb-6">
          <h2 class="text-lg font-semibold mb-2">{{ group.date }}</h2>

          <!-- Single Item Display with Expand Option -->
          <div
            v-for="(product, index) in group.items"
            :key="product.id"
            v-show="!group.isExpanded && index === 0"
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
          </div>

          <!-- Toggle Button -->
          <button
            class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-500"
            @click="toggleExpand(group)"
          >
            {{ group.isExpanded ? "Collapse" : "Show All Items" }}
          </button>
        </div>
      </div>

      <div
        class="text-right mt-4 bg-white text-black fixed right-5 shadow-lg border-2 rounded-lg w-[400px] h-[700px]"
      >
        <h1
          class="text-center p-5 text-3xl font-semibold border-b-2 border-gray-300"
        >
          Order Summary
        </h1>
        <div class="w-full p-5">
          <table class="w-full text-xl text-gray-700">
            <tr class="mt-6">
              <th class="text-left py-8">Total Price</th>
              <td class="font-semibold">
                {{ changeMonetaryFormat(totalPrice) }}
              </td>
            </tr>
            <tr class="mt-6">
              <th class="text-left py-8">Total Quantity</th>
              <td class="font-semibold">{{ totalQty }}</td>
            </tr>
            <tr class="mt-6">
              <th class="text-left py-8">Address</th>
              <td>
                <address class="block text-sm">Address</address>
                <a href="#" class="text-blue-500 hover:underline text-sm"
                  >Update Address</a
                >
              </td>
            </tr>
            <tr class="mt-6 border-b-4">
              <th class="text-left py-8">Estimated Delivery Fee</th>
              <a href="#" class="text-blue-500 hover:underline text-sm"
                >Delivery Fee</a
              >
            </tr>
          </table>

          <button
            class="px-6 py-3 bg-yellow-600 text-white rounded-lg w-full mt-8 shadow-md hover:bg-yellow-500 transition duration-200"
            @click="PurchaseCart()"
          >
            Check Out Item
          </button>
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
        .get(
          `${process.env.VUE_APP_BASE_URL}/user_carts/find_by_type/${this.$store.state.user.id}/1/1/20`
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
              const updatePromise = axios.put(
                `${process.env.VUE_APP_BASE_URL}/user_carts/edit/${product.id}`,
                {
                  type: 2,
                  datetime_added: new Date().toISOString(),

                }
              );

              // Push the promise to the array
              updatePromises.push(updatePromise);
            });

            // Wait for all promises to resolve
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
