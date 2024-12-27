<template>
  <div class="px-5 flex">
    <!-- Conditionally Render Tabs -->
    <div>
      <div>
        <div class="rounded-lg bg-white">
          <div class="w-full"><h1 class="p-2 text-gray-500">Search Order ID#</h1></div>
          <div>
            <i class="pi pi-search absolute text-gray-600 mt-3 ml-5"></i>
            <input type="text"  class="w-11/12 rounded-xl  mx-3 bg-gray-300 border-gray-400"/>
          </div>
          <div><h1 class="p-2 text-gray-500">Progress</h1></div>
          <div class="flex justify-between">
            <div class="p-10">
              <i class="pi pi-check p-4 bg-green-600 rounded-full text-white text-3xl"></i>
              <h1>Order Confirmed</h1>
            </div>
            <div class="p-10">
              <i class="pi pi-check p-4 bg-green-600 rounded-full text-white text-3xl"></i>
              <h1>Payment Confirmed</h1>
            </div>
            <div class="p-10">
              <i class="pi pi-check p-4 bg-green-600 rounded-full text-white text-3xl"></i>
              <h1>Processed</h1>
            </div>
            <div class="p-10">
              <i class="pi pi-box p-4 bg-green-600 rounded-full text-white text-3xl"></i>
              <h1>Shipped</h1>
            </div>
            <div class="p-10">
              <i class="pi pi-truck p-4 bg-orange-600 rounded-full text-white text-3xl"></i> 
              <h1>Delivery</h1>
            </div>
          </div>
        </div>
        <div>
          <div class="flex flex-col w-full">
            <div
              class="w-full max-w-full h-[600px] overflow-auto p-5 mx-5 my-2"
            >
              <div
                v-for="product in allpurchase"
                :key="product.id"
                class="flex flex-wrap items-center justify-between p-4 border rounded-lg shadow hover:shadow-lg transition mb-4 bg-white"
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
                <div class="items-center gap-4 p-2 rounded min-w-[100px]">
                  <p>status</p>
                </div>
                <div class="flex items-center gap-4 p-2 rounded min-w-[100px]">
                  <p>{{ product.quantity }}</p>
                </div>
                <div
                  class="flex justify-between items-center px-5 mx-5 min-w-[150px]"
                >
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
          <div>
            <div><h1>Order timeline</h1></div>
          </div>
          <div>
            <i></i>
            <div>
              <h1>title</h1>
              <p>sub title</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="w-1/2 mt-5">
      <div class="bg-white px-5 mx-5 my-2 rounded-md">
        <h1>Order Summary</h1>
        <div>
          <table class="w-full text-gray-700">
            <tr class="border-y-2 mt-10">
              <th class="text-left py-2"><h1>Sub-total</h1></th>
              <td class="text-right font-semibold">$0.00</td>
            </tr>
            <tr class="mt-6 border-y-2">
              <th class="text-left py-2"><h1>Discount</h1></th>
              <td class="text-right font-semibold">$0.00</td>
            </tr>
            <tr class="mt-6 border-y-2">
              <th class="text-left py-2"><h1>Deslivery Charge</h1></th>
              <td class="text-right font-semibold">$0.00</td>
            </tr>
            <tr class="mt-6 border-y-2">
              <th class="text-left py-2"><h1>Tax</h1></th>
              <td class="text-right font-semibold">$0.00</td>
            </tr>
            <tr class="mt-6">
              <th class="text-left py-2"><h1>Total Amount:</h1></th>
              <td class="text-right font-semibold">$0.00</td>
            </tr>
          </table>
        </div>
      </div>
      <div class="bg-white px-5 mx-5 my-2 border rounded-md">
        <div class="justify-between flex">
          <h1>Customer</h1>
          <a href="">Edit</a>
        </div>
        <div class="my-5">
          <img src="" alt="" srcset="" />
          <div>
            <p v-if="allpurchase.length > 0">
              {{ allpurchase[0].user_id.fname }}
              {{ allpurchase[0].user_id.lname }}
              {{ allpurchase[0].user_id.mname }}
            </p>
            <a v-if="allpurchase.length > 0">
              {{ allpurchase[0].user_id.email }}
            </a>
            <p v-if="allpurchase.length > 0">
              {{ allpurchase[0].user_id.contact_num }}
            </p>
          </div>
        </div>
      </div>
      <div class="bg-white p-5 mx-5 my-2 border rounded-md">
        <div class="justify-between flex">
          <h1>Contact person</h1>
          <a href="">Edit</a>
        </div>
        <div class="my-5">
          <img src="" alt="" srcset="" />
          <div>
            <p v-if="allpurchase.length > 0">
              {{ allpurchase[0].user_id.fname }}
              {{ allpurchase[0].user_id.lname }}
              {{ allpurchase[0].user_id.mname }}
            </p>
          </div>
        </div>
      </div>
      <div class="bg-white p-5 mx-5 my-2 border rounded-md">
        <div class="justify-between flex">
          <h1>Shipping Address</h1>
          <a href="">Edit</a>
        </div>
        <div class="my-5">
          <img src="" alt="" srcset="" />
          <div>
            <p v-if="allpurchase.length > 0">
              {{ allpurchase[0].user_id.address }}
            </p>
          </div>
        </div>
      </div>
      <div class="bg-white p-5 mx-5 border rounded-md">
        <div class="justify-between flex">
          <h1>Billing Address</h1>
          <a href="">Edit</a>
        </div>
        <div class="my-5">
          <img src="" alt="" srcset="" />
          <div>
            <p v-if="allpurchase.length > 0">
              {{ allpurchase[0].user_id.address }}
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
  data() {
    return {
      selected_id: 0,
      allpurchase: [],
      selectedTab: "All",
    };
  },
  methods: {
    goToProductDetail(productId) {
      this.$router.push({ name: "order-detail", params: { id: productId } });
    },
    refreshData() {
      this.getParam();
    },
    getParam() {
      this.selected_id = this.$route.params.id;
      this.getItemsFromCart();
    },
    getItemsFromCart() {
      axios
        .get(
          `${process.env.VUE_APP_BASE_URL}/user_carts/find/${this.selected_id}`
        )
        .then((res) => {
          if (res.status === 200 && res.data) {
            const item = res.data;
            this.allpurchase = [
              {
                id: item.id,
                product_id: item.product_id.id,
                name: item.product_id.name,
                sku: item.product_id.sku,
                product_img: item.product_id.product_img,
                price: item.product_id.price || 0,
                quantity: item.quantity,
                user_id: item.user_id,
              },
            ];
          }
        })
        .catch((error) => {
          console.error("Error fetching cart items:", error);
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
    goToProducts() {
      this.$router.push({ name: "products" });
    },
  },
  mounted() {
    this.refreshData();
  },
  watch: {
    $route: {
      handler() {
        this.refreshData();
      },
    },
  },
};
</script>
