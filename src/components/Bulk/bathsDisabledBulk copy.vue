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
      uploadError: null,
      uploadSuccess: false,
      csvToDbMap: {
        "Dimensions (w*d*h)": "dimensions",
        "Вес, кг": "weight",
        "Вместимость, л": "capacity",
        Описание: "description",
        "Расположение слива": "drain_location",
        "Высота входа в ванну, мм": "entrance_height",
        "Высота проема двери,мм": "opening_height",
        "Ширина открытия, мм": "opening_width",
        "Высота сидения, мм": "seat_height",
        Слив: "drain",
        Оснащение: "equipment",
        Опции: "options",
        Аксессуары: "accessories",
        Гарантия: "guarantee",
        "Вес упаковки, кг": "package_weight",
        Item: "Артикул",
        "Артикул от завода": "article_num",
        Коллекция: "collection",
        Название: "name",
      },
      currentUniqueId: Math.floor(Math.random() * 999999),
    };
  },
  methods: {
    handleFileUpload(event) {
      this.file = event.target.files[0];
      this.resetStatus();
    },
    resetStatus() {
      this.uploadError = null;
      this.uploadInProgress = null;
      this.uploadSuccess = false;
    },
    generateUniqueId() {
      this.currentUniqueId += 1;
      return this.currentUniqueId;
    },
    async uploadFile() {
      if (!this.file) {
        this.uploadError = "Please select a file to upload.";
        return;
      }

      this.uploadInProgress = "Parsing file...";
      this.resetStatus();

      Papa.parse(this.file, {
        header: true,
        skipEmptyLines: true,
        complete: async (results) => {
          const data = results.data.filter((row) =>
            Object.values(row).some((value) => value?.trim())
          );

          if (data.length === 0) {
            this.uploadError = "The file contains no valid data.";
            this.uploadInProgress = null;
            return;
          }

          try {
            this.uploadInProgress = "Uploading data to the server...";
            const products = [];
            const accessories = [];
            const usedProductIds = new Set();

            data.forEach((row) => {
              const mappedRow = this.mapCsvToDb(row);
              let uniqueId = mappedRow.product_id;

              if (!uniqueId || isNaN(uniqueId) || uniqueId <= 0) {
                uniqueId = this.generateUniqueId();
              }

              if (usedProductIds.has(uniqueId)) {
                console.warn(`Duplicate product ID: ${uniqueId}. Skipping.`);
                return;
              }

              usedProductIds.add(uniqueId);

              products.push({
                id: uniqueId,
                name: mappedRow.name || "Unnamed Product",
                article_num: mappedRow.article_num,
                product_image: mappedRow.product_image || "",
                category_id: "1",
              });

              payload.push({
                id: uniqueId,
                product_id: uniqueId,
                dimensions: mappedRow.dimensions || "",
                weight: parseFloat(mappedRow.weight) || null,
                capacity: mappedRow.capacity || "",
                description: mappedRow.description || "",
                drain_location: mappedRow.drain_location || "",
                entrance_height: parseInt(mappedRow.entrance_height) || null,
                opening_height: parseInt(mappedRow.opening_height) || null,
                opening_width: parseInt(mappedRow.opening_width) || null,
                seat_height: parseInt(mappedRow.seat_height) || null,
                drain: mappedRow.drain || "",
                equipment: mappedRow.equipment || "",
                options: mappedRow.options || "",
                accessories: mappedRow.accessories || "",
                guarantee: mappedRow.guarantee || "",
                package_weight: parseFloat(mappedRow.package_weight) || null,
                datetime_modified: new Date().toISOString(),
              });
            });

            await axios.post(
              process.env.VUE_APP_BASE_URL + "/products/bulk",
              products
            );
            await axios.post(
              process.env.VUE_APP_BASE_URL + "/info_baths_disabled/bulk",
              accessories
            );

            this.uploadSuccess = true;
          } catch (error) {
            this.uploadError =
              error.response?.data || "An error occurred during upload.";
            console.error("Upload error:", error);
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
      return Object.fromEntries(
        Object.entries(this.csvToDbMap).map(([csvKey, dbField]) => [
          dbField,
          row[csvKey],
        ])
      );
    },
  },
};
</script>
