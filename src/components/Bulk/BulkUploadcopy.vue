<template>
  <div class="p-6 bg-gray-50 min-h-screen">
    <h1 class="text-2xl font-bold mb-4">Bulk Upload Bathroom Accessories</h1>

    <div class="border-2 border-dashed border-gray-400 rounded-md p-4">
      <!-- File Input -->
      <input
        type="file"
        accept=".csv"
        @change="handleFileUpload"
        class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100"
      />

      <!-- Category Input -->
      <div class="mt-4">
        <label for="category" class="block text-sm font-medium text-gray-700"
          >Category</label
        >
        <select
          id="category"
          name="category"
          v-model="selectedCategory"
          @change="fetchCollectionsByCategory"
        >
          <option
            v-for="category in categories"
            :key="category.id"
            :value="category.id"
          >
            {{ category.description }}
          </option>
        </select>
      </div>

      <!-- Collection Input -->
      <div class="mt-4">
        <label for="collection" class="block text-sm font-medium text-gray-700"
          >Collection</label
        >
        <select id="collection" name="collection" v-model="selectedCollection">
          <option
            v-for="collection in filteredCollections"
            :key="collection.id"
            :value="collection.id"
          >
            {{ collection.description }}
          </option>
        </select>
      </div>

      <!-- Upload Button -->
      <button
        @click="uploadFile"
        :disabled="uploadInProgress"
        class="mt-4 px-4 py-2 bg-indigo-600 text-white rounded disabled:bg-gray-300"
      >
        Upload File
      </button>
    </div>

    <!-- Progress or Error Messages -->
    <div v-if="uploadInProgress" class="mt-4 text-blue-600">
      {{ uploadInProgress }}
    </div>
    <div v-if="uploadError" class="mt-4 text-red-600">{{ uploadError }}</div>

    <!-- Success Message -->
    <div v-if="uploadSuccess" class="mt-4 text-green-600">
      Upload successful!
    </div>
  </div>
</template>

<script>
import Papa from "papaparse";
import axios from "axios";

export default {
  data() {
    return {
      file: null,
      selectedCategory: null,
      selectedCollection: null,
      categories: [],
      collections: [],
      filteredCollections: [],
      uploadInProgress: null,
      uploadError: null,
      uploadSuccess: false,
      currentUniqueId: 1000, // Start ID from a reasonable number
      csvToDbMap: {
        Item: "Артикул",
        "Артикул от завода": "sku",
        Коллекция: "collection",
        Название: "name",
        "Изделие (Оснащение)": "product_equipment",
        Стиль: "style",
        Цвет: "color",
        "Габариты (ш*г*в)": "dimensions",
        Поверхность: "surface",
        Материал: "material",
        "Вес товара, кг": "product_weight",
        "Вес упаковки, кг": "package_weight",
        Гарантия: "guarantee",
        Фото: "product_image",
      },
    };
  },
  methods: {
    async fetchCategories() {
      try {
        const response = await axios.get(
          process.env.VUE_APP_BASE_URL + "/categories/get_all"
        );
        this.categories = response.data;
      } catch (error) {
        console.error("Error fetching categories:", error);
        this.uploadError = "Error fetching categories.";
      }
    },

    async fetchCollectionsByCategory() {
      if (this.selectedCategory) {
        try {
          const response = await axios.get(
            process.env.VUE_APP_BASE_URL + "/collections/get_all",
            {
              params: { category_id: this.selectedCategory },
            }
          );
          this.collections = response.data;
          this.filteredCollections = this.collections.filter(
            (collection) => collection.category_id === this.selectedCategory
          );
        } catch (error) {
          console.error("Error fetching collections:", error);
          this.uploadError = "Error fetching collections.";
        }
      } else {
        this.filteredCollections = [];
      }
    },

    handleFileUpload(event) {
      this.file = event.target.files[0];
      this.resetStatus();
    },

    resetStatus() {
      this.uploadError = null;
      this.uploadInProgress = null;
      this.uploadSuccess = false;
    },

    async uploadFile() {
      if (!this.file) {
        this.uploadError = "Please select a file to upload.";
        return;
      }

      if (!this.selectedCategory || !this.selectedCollection) {
        this.uploadError = "Please select both category and collection.";
        return;
      }

      this.uploadInProgress = "Parsing file...";
      this.uploadError = null;
      this.uploadSuccess = false;

      Papa.parse(this.file, {
        header: true,
        skipEmptyLines: true,
        complete: async (results) => {
          const data = results.data.filter((row) =>
            Object.values(row).some((value) => value && value.trim() !== "")
          );

          // Alert the number of valid rows
          alert(`The file contains ${data.length} valid rows.`);

          if (data.length === 0) {
            this.uploadError = "The file contains no valid data.";
            this.uploadInProgress = null;
            return;
          }

          try {
            this.uploadInProgress = "Uploading data to the server...";

            // Prepare payloads
            const products = [];
            const accessories = [];
            const usedProductIds = new Set(); // Track product IDs to avoid duplicates

            for (const row of data) {
              const mappedRow = this.mapCsvToDb(row);

              let uniqueId = mappedRow.product_id;

              // If product_id is invalid, generate a new numeric ID
              if (!uniqueId || uniqueId === "0" || uniqueId === "null") {
                uniqueId = this.generateUniqueId();
              }

              // If the generated ID already exists, skip adding this product and accessory
              if (usedProductIds.has(uniqueId)) {
                console.log(
                  `Duplicate product ID found: ${uniqueId}. Skipping product.`
                );
                continue; // Skip the current product and accessory
              }

              // Add the unique ID to the set of used product IDs
              usedProductIds.add(uniqueId);

              // Add to products with generated unique ID
              const product = {
                id: uniqueId,
                name: mappedRow.name || "Unnamed Product",
                sku: mappedRow.sku,
                product_image: mappedRow.product_image || "",
                category_id: this.selectedCategory,
                collection_id: this.selectedCollection,
              };
              products.push(product);

              // Add to accessories with the same unique ID
              const accessory = {
                product_equipment: mappedRow.product_equipment || "",
                style: mappedRow.style || "",
                color: mappedRow.color || "",
                dimensions: mappedRow.dimensions || "",
                surface: mappedRow.surface || "",
                material: mappedRow.material || "",
                product_weight: parseFloat(mappedRow.product_weight) || null,
                package_weight: parseFloat(mappedRow.package_weight) || null,
                guarantee: mappedRow.guarantee || "",
                product_id: uniqueId,
              };
              accessories.push(accessory);
            }

            // Upload products and accessories
            await axios.post(
              process.env.VUE_APP_BASE_URL + "/products/bulk",
              products
            );
            await axios.post(
              process.env.VUE_APP_BASE_URL + "/info_bathroom_accessories/bulk",
              accessories
            );

            this.uploadSuccess = true;
          } catch (error) {
            console.error("Upload failed:", error.response || error);
            this.uploadError =
              "An error occurred during upload. Please try again.";
          } finally {
            this.uploadInProgress = null;
            this.file = null;
          }
        },
        error: (error) => {
          console.error("Parsing error:", error);
          this.uploadError = "Error parsing the file. Please check its format.";
          this.uploadInProgress = null;
        },
      });
    },

    mapCsvToDb(row) {
      const mappedRow = {};
      for (const [csvHeader, dbField] of Object.entries(this.csvToDbMap)) {
        mappedRow[dbField] = row[csvHeader];
      }
      return mappedRow;
    },

    generateUniqueId() {
      // Increment the current ID for each new row
      this.currentUniqueId += 1;
      return this.currentUniqueId;
    },
  },

  mounted() {
    this.fetchCategories(); // Fetch categories on component mount
  },
};
</script>
