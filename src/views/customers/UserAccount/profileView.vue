<template>
  <div class="flex justify-center items-center p-20 bg-gray-100 min-h-screen">
    <div class="flex mx-10 bg-white p-8 rounded-lg shadow-xl w-full max-w-7xl">
      <div>
        <!-- Header Section -->
        <div class="flex items-center my-10">
          <i class="pi pi-user text-6xl text-yellow-600"></i>
          <div class="ml-4">
            <h1 class="text-3xl font-semibold text-gray-900">
              {{ user.fname }}
            </h1>
            <p class="text-lg text-gray-600">View profile</p>
          </div>
        </div>

        <!-- Navigation Links -->
        <div>
          <ul>
            <!-- My Account -->
            <li
              class="flex items-center text-lg font-medium text-gray-1000 hover:text-yellow-600 cursor-pointer py-2"
              @click="toggleMenu('account')"
            >
              <i class="pi pi-user text-xl mr-2"></i>
              My Account
            </li>
            <ul v-if="menuOpen === 'account'" class="pl-8">
              <li
                class="my-1 text-gray-700 hover:text-yellow-600 cursor-pointer"
                @click="setView('profile')"
              >
                Profile
              </li>
              <li
                class="my-1 text-gray-700 hover:text-yellow-600 cursor-pointer"
                @click="setView('addresses')"
              >
                Addresses
              </li>
              <li
                class="my-1 text-gray-700 hover:text-yellow-600 cursor-pointer"
                @click="setView('changePassword')"
              >
                Change Password
              </li>
              <li
                class="my-1 text-gray-700 hover:text-yellow-600 cursor-pointer"
                @click="setView('privacySettings')"
              >
                Privacy Settings
              </li>
            </ul>

            <!-- My Purchase -->
            <li
              class="flex items-center text-lg font-medium text-gray-1000 hover:text-yellow-600 cursor-pointer py-2"
              @click="setView('purchase')"
            >
              <i class="pi pi-receipt text-xl mr-2"></i>
              My Purchase
            </li>

            <!-- Logout -->
            <li
              class="flex items-center text-lg font-medium text-gray-1000 hover:text-yellow-600 cursor-pointer py-2"
              @click="confirmLogout"
            >
              <i class="pi pi-sign-out text-xl mr-2"></i>
              Logout
            </li>
          </ul>
        </div>
      </div>

      <!-- Profile Form Section -->
      <ProfileForm
        v-if="currentView === 'profile'"
        :user="user"
        @updateProfile="updateProfile"
      />

      <!-- Address Form Section -->
      <Address v-if="currentView === 'addresses'" />

      <!-- Change Password Section -->
      <Password v-if="currentView === 'changePassword'"/>

      <!-- Privacy Settings Section -->
      <div v-if="currentView === 'privacySettings'">Privacy Settings Content Here</div>

      <!-- Purchase Form Section -->
      <PurchaseForm v-if="currentView === 'purchase'" />
 
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Cookies from "js-cookie";
import ProfileForm from "../../../components/CustomerAccount/profile.vue";
import PurchaseForm from "../../../components/CustomerAccount/purchase.vue";
import Address from "../../../components/CustomerAccount/address.vue";
import Password from "../../../components/CustomerAccount/ChangePassword.vue";

import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';

export default {
  components: {
    ProfileForm,
    PurchaseForm,Address,
    Password,
  },
  data() {
    return {
      user: {
        username: "",
        fname: "",
        mname: "",
        lname: "",
        email: "",
        contact_num: "",
        address: "",
        gender: "",
        dob: "",
        imagePreview: null,
        selected_id: 0,
        userId: "",
      },
      currentView: "profile", // Default view
      menuOpen: null, // Tracks which menu is open ('account' or 'purchase')
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
    updateProfile(updatedUser) {
      axios
        .put(
          `${process.env.VUE_APP_BASE_URL}/users/update/${updatedUser.id}`,
          updatedUser
        )
        .then((response) => {
          alert("Profile updated successfully!");
        })
        .catch((error) => {
          console.error("Error updating profile:", error);
          alert("There was an error updating your profile");
        });
    },
    confirmLogout() {
      toast.info("User Logout", {
        
        position: "top-right",
        hideProgressBar: true,
        closeOnClick: true,
        pauseOnHover: true,
        draggable: true,
        progress: undefined,
        theme: "darked",
        onClick: () => {
          this.logout();
  
        }
      });
    },
    logout() {
      this.$store.commit("UPDATE_USER", null);
      this.$router.push("/");
    },
    setView(view) {
      this.currentView = view;
      this.menuOpen = null; // Collapse menu when navigating to a view
    },
    toggleMenu(menu) {
      this.menuOpen = this.menuOpen === menu ? null : menu;
    },
  },
  mounted() {
    this.fetchUserData();
  },
};
</script>
