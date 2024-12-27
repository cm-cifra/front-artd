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
      <!-- Category Dropdown -->
      <div class="mt-4">
        <label for="category" class="block text-sm font-medium text-gray-700">
          Category
        </label>
        <select
          id="category"
          name="category"
          v-model="selectedCategory"
          class="w-full mt-1 border-gray-300 rounded-md"
        >
          <option value="" disabled>Select a category</option>
          <option
            v-for="category in categories"
            :key="category.id"
            :value="category.id"
          >
            {{ category.description }}
          </option>
        </select>
        <p v-if="categories.length === 0" class="text-sm text-red-600 mt-1">
          No categories available. Please check your connection.
        </p>
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
      uploadInProgress: null,
      selectedCategory: null,
      uploadError: null,
      uploadSuccess: false,
      categories: [],
      csvToDbMap: {
        Item: "Артикул",
        Бренд: brand,
        Конфигурация: configuration,
        "Тип Раковины-Столешницы": countertop_sink_type,
        "Габариты Тумбы (ш*г*в), мм": demension_pedestal,
        "Ширина, мм": width,
        "Цвет тумбы": color_cabinet,
        Монтаж: installation,
        fittings_included,
        fittings_with_closer_mechanism,
        material_cabinet_facade,
        material_cabinet_body,
        pedestal_sku,
        num_pedestals,
        countertop_sink_sku,
        num_countertop,
        mirror_sku,
        num_mirrors,
        mixer_factory_num,
        num_mixers,
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
      usedProductIds: new Set(), // Set to track used product IDs
      currentUniqueId: Math.floor(Math.random() * 999999), // Start from an arbitrary large number to prevent collisions
    };
  },
  methods: {
    handleFileUpload(event) {
      this.file = event.target.files[0];
      this.resetStatus();
    },

    async fetchCategories() {
      try {
        const response = await axios.get(
          process.env.VUE_APP_BASE_URL + "/categories/get_all"
        );
        this.categories = response.data;
      } catch (error) {
        console.error("Error fetching categories:", error);
        this.uploadError = "Failed to fetch categories.";
      }
    },

    resetStatus() {
      this.uploadError = null;
      this.uploadInProgress = null;
      this.uploadSuccess = false;
    },

    generateUniqueId() {
      // Generate a numeric ID using Date.now() and Math.random()
      return Math.floor(Math.random() * 10000) + Date.now(); // Generate a large random number
    },

    async uploadFile() {
      if (!this.file) {
        this.uploadError = "Please select a file to upload.";
        return;
      }

      if (!this.selectedCategory) {
        this.uploadError = "Please select a category.";
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

              // Log generated ID for debugging
              console.log("Generated Unique ID:", uniqueId);

              // Validate the unique ID before adding it to the product and accessory
              if (isNaN(uniqueId) || uniqueId <= 0) {
                console.error("Invalid unique ID generated:", uniqueId);
                this.uploadError = "Failed to generate a valid unique ID.";
                return;
              }

              // Add to products with generated unique ID
              const product = {
                id: uniqueId, // Assign unique ID
                name: mappedRow.name || "Unnamed Product",
                sku: mappedRow.sku,
                product_image: mappedRow.product_image || "",
                category_id: this.selectedCategory,
                datetime_modified: Date.now(),
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
                product_id: uniqueId, // Use the same unique ID for the accessory
                datetime_added: Date.now(),
              };
              accessories.push(accessory);
            }

            // Log the payloads before uploading to check their structure
            console.log("Products Payload:", products);
            console.log("Accessories Payload:", accessories);

            // Upload products
            await axios.post(
              process.env.VUE_APP_BASE_URL + "/products/bulk",
              products
            );

            // Upload accessories
            await axios.post(
              process.env.VUE_APP_BASE_URL + "/info_bathroom_accessories/bulk",
              accessories
            );

            this.uploadSuccess = true;
          } catch (error) {
            console.error("Upload failed:", error.response || error);
            if (error.response && error.response.data) {
              console.log("Error response data:", error.response.data);
            }
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
    this.fetchCategories();
  },
};
</script>
