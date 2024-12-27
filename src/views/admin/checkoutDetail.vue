<template>
  <div class="flex bg-gray-300">
    <!-- Populated Cart -->
    <div class="mx-10 my-10 flex bg-white">
      <div class="m-5 w-full">
        <div>
          <h1 class="text-2xl p-5">More info</h1>
        </div>

        <!-- Status Timeline -->
        <div class="p-5">
          <h1 class="text-lg font-semibold mb-4">Status</h1>
          <div class="flex items-center justify-between">
            <div
              v-for="(step, index) in statusSteps"
              :key="index"
              class="flex items-center"
            >
              <!-- Circle -->
              <div
                :class="[
                  'flex items-center justify-center w-8 h-8 rounded-full',
                  step.completed
                    ? 'bg-blue-500 text-white'
                    : 'bg-gray-300 text-gray-500',
                ]"
              >
                <span v-if="step.completed">✔</span>
                <span v-else>{{ index + 1 }}</span>
              </div>

              <!-- Line -->
              <div
                v-if="index < statusSteps.length - 1"
                :class="[
                  'h-1 w-12',
                  statusSteps[index + 1].completed
                    ? 'bg-blue-500'
                    : 'bg-gray-300',
                ]"
              ></div>
            </div>
          </div>
          <!-- Status Labels -->
          <div class="flex justify-between mt-2">
            <div
              v-for="step in statusSteps"
              :key="step.label"
              class="text-center"
            >
              <h2
                :class="[
                  'text-sm font-semibold',
                  step.completed ? 'text-black' : 'text-gray-500',
                ]"
              >
                {{ step.label }}
              </h2>
              <p v-if="step.timestamp" class="text-xs text-gray-600">
                {{ step.timestamp }}
              </p>
              <p v-else class="text-xs text-gray-600">{{ step.status }}</p>
            </div>
          </div>
        </div>
        <div
          v-for="(product, index) in allcart"
          :key="product.id"
          class="flex items-center justify-between p-4 border rounded-lg shadow hover:shadow-lg transition mb-4"
          @click="viewProduct(product.sku)"
        >
          <!-- Product Image -->
          <img
            :src="getImgUrl(product.product_img)"
            :alt="product.name"
            class="w-32 h-32 object-cover rounded-md cursor-pointer"
            @click="viewImg(product.product_img)"
          />

          <!-- Product Details -->
          <div class="flex-1 px-4">
            <h2 class="text-lg font-semibold mb-1">{{ product.name }}</h2>
            <p class="text-gray-600">SKU: {{ product.sku }}</p>

            <p class="text-gray-600">quantity: {{ product.quantity }}</p>
          </div>
          <div class="justify-between items-center px-20 mx-10 flex">
            <h1 class="text-black-600">Price: {{ product.price }}</h1>
            <h1>{{ product.user_id.id }}</h1>
          </div>

          <!-- Quantity Control -->
          <div class="flex items-center gap-4 p-2 rounded">
            <h1 class="text-sm font-medium text-gray-600">Quantity:</h1>
            <div class="flex items-center">
              <span class="px-4 text-lg font-semibold text-gray-800">
                {{ product.quantity }}
              </span>
               
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="w-full my-10">
      <div class="mr-10 bg-white p-3 border rounded-md m-2">
        <div class="justify-between flex">
          <h1>Customer</h1>
          <a href="">Edit</a>
        </div>
        <div class="my-5">
          <img src="" alt="" srcset="" />
          <div>
            <p v-if="allcart.length > 0">
              {{ allcart[0].user_id.fname }} {{ allcart[0].user_id.lname }}
              {{ allcart[0].user_id.mname }}
            </p> <a v-if="allcart.length > 0">
              {{ allcart[0].user_id.email }}  
            </a> <p v-if="allcart.length > 0">
              {{ allcart[0].user_id.contact_num }} 
            </p>
          </div>
        </div>
      </div>
      <div class="mr-10 bg-white p-3 border rounded-md m-2">
        <div class="justify-between flex">
          <h1>Contact person</h1>
          <a href="">Edit</a>
        </div> <div class="my-5">
          <img src="" alt="" srcset="" />
          <div>
            <p v-if="allcart.length > 0">
              {{ allcart[0].user_id.fname }} {{ allcart[0].user_id.lname }}
              {{ allcart[0].user_id.mname }}
            </p>  
          
          </div>
        </div>
      </div>
      <div class="mr-10 bg-white p-3 border rounded-md m-2">
        <div class="justify-between flex">
          <h1>Shipping Address</h1>
          <a href="">Edit</a>
        </div> <div class="my-5">
          <img src="" alt="" srcset="" />
          <div>
            <p v-if="allcart.length > 0">
              {{ allcart[0].user_id.address }} 
            </p> 
          
          </div>
        </div>
      </div>
      <div class="mr-10 bg-white p-3 border rounded-md m-2">
        <div class="justify-between flex">
          <h1>Billing Address</h1>
          <a href="">Edit</a>
        </div> <div class="my-5">
          <img src="" alt="" srcset="" />
          <div>
            <p v-if="allcart.length > 0">
              {{ allcart[0].user_id.address }} 
            </p> 
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";

export default {
  props: ["user_id", "date"],
  data() {
    return {
      allcart: [],
      statusSteps: [
        { label: "Receiving orders", completed: true, timestamp: "05:43 AM" },
        { label: "Order processing", completed: true, timestamp: "01:21 PM" },
        { label: "Being delivered", completed: false, status: "Processing" },
        { label: "Delivered", completed: false, status: "Pending" },
      ],
    };
  },
  computed: {
    totalPrice() {
      return this.allcart.reduce(
        (total, product) => total + product.price * product.quantity,
        0
      );
    },
    totalQty() {
      return this.allcart.reduce(
        (total, product) => total + product.quantity,
        0
      );
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
        .get(`${process.env.VUE_APP_BASE_URL}/user_carts/get_all/1/20`, {})
        .then((res) => {
          if (res.status === 200) {
            const cartData = res.data[0];

            const userIds = parseInt(this.user_id);
            this.allcart = cartData
              .filter(
                (item) =>
                  item.status === 2 &&
                  item.user_id.id === userIds &&
                  item.datetime_added === this.date
              )
              .map((item) => ({
                id: item.id,
                product_id: item.product_id.id,
                name: item.product_id.name,
                sku: item.product_id.sku,
                user_id: item.user_id,
                product_img: item.product_id.product_img,
                price: item.product_id.price || 0,
                quantity: item.quantity,
                date: item.datetime_added,
              }));
          }
        })
        .catch((error) => {
          console.error("Error fetching cart items:", error);
        });
    },
    PurchaseCart() {
      const userId = this.$store.state.user.id;

      axios
        .get(`${process.env.VUE_APP_BASE_URL}/user_carts/get_all/1/20`, {})
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
    viewProduct(id) {
      this.$router.push("/products/" + id);
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
