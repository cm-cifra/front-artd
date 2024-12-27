<template>
    <div class="justify-center align-middle items-center mx-5 h-[600px] pt-10 w-[600px]">
      <h1 class="text-3xl px-10 pb-5">Update Address</h1>
      <form @submit.prevent="updateAddress">
        <div>
          <label for="address">Street:</label>
          <input v-model="user.address" id="address" type="text" placeholder="Enter address" />
        </div>
        <div>
          <label for="city">City:</label>
          <select v-model="user.city" id="city">
            <option disabled value="">Select City</option>
            <option v-for="city in cities" :key="city" :value="city">
              {{ city }}
            </option>
          </select>
        </div>
        <div>
          <label for="state">State/Region:</label>
          <select v-model="user.state" id="state" class="w-full">
            <option disabled value="">Select State/Region</option>
            <option v-for="state in states" :key="state" :value="state">
              {{ state }}
            </option>
          </select>
        </div>
        <div><label for="">Zip Code:</label><input type="text"></div>
        <button type="submit"  class="bg-yellow-600 hover:bg-yellow-500 ">Update</button>
      </form>
    </div>
  </template>
  
  <script>
  import axios from "axios";
  
  export default {
    props: {},
    data() {
      return {
        user: {
          address: "",
          city: "",
          state: "",
        },
        cities: ["Moscow", "Saint Petersburg", "Novosibirsk", "Yekaterinburg", "Kazan", "Nizhny Novgorod"],  
        states: ["Moscow Oblast", "Leningrad Oblast", "Novosibirsk Oblast", "Sverdlovsk Oblast", "Tatarstan", "Nizhny Novgorod Oblast"],  
      };
    },
    methods: {
      fetchUserData() {
        const userId = this.$store.state.user.id;
        axios
          .get(`${process.env.VUE_APP_BASE_URL}/users/find/${userId}`)
          .then((response) => {
            this.user = response.data;
          })
          .catch((error) => {
            console.error("Error fetching user data:", error);
          });
      },
      updateAddress() {
        const userId = this.$store.state.user.id;
        axios
          .post(`${process.env.VUE_APP_BASE_URL}/users/edit/${userId}`, this.user)
          .then((response) => {
            alert("Address updated successfully!");
          })
          .catch((error) => {
            console.error("Error updating address:", error);
          });
      },
    },
    mounted() {
      this.fetchUserData();
    },
  };
  </script>
  
  <style scoped>
  form {
    max-width: 400px;
    margin: 0 auto;
  }
  
  div {
    margin-bottom: 10px;
  }
  
  label {
    display: block;
    margin-bottom: 5px;
  }
  
  input, select, button {
    width: 100%;
    padding: 8px;
    margin-bottom: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
 
  
   
  </style>
  