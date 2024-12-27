<template>
  <div class="p-6 bg-gray-50 min-h-screen">
    <h1 class="text-2xl font-bold mb-4">Bulk Upload Душевые системы</h1>

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
        Артикул: "sku",
        "Артикул от завода": "factory_part_num",
        Коллекция: "collection",
        Название: "name",
        Поверхность: "product_equipment",
        Цвет: "color",
        "Габариты (ш*г*в)": "surface",
        Код: "material",
        Поверхность: "surface",
        Материал: "material",
        Излив: "spout",
        "Высота смесителя, мм": "height",
        "Высота излива, мм": "spout_height",
        "Длина излива, мм": "spout_length",
        Монтаж: "installation",
        Управление: "control",
        "Тип подводки": "eyeliner_type",
        "Стандарт подводки": "eyeliner_standard",
        "Механизм управления": "control_mechanism",
        "Подходит для раковины-чаши": "suitable_for_bowl_sink",
        "Функция экономии расхода": "consumption_saving",
        "Вес упаковки, кг": "package_weight",
        Стиль: "style",
        "Габариты (ш*в*г)": "dimensions",
        Форма: "form",
        Рама: "frame",
        Материал: "material",
        "Расположение подсветки": "backlight_placement",
        "Цвет подсветки": "backlight_color",
        "Тип выключателя": "switch_type",
        "Покрытие «анти-туман»": "anti_fog_coating",
        Особенности: "peculiarities",
        "Вес товара, кг": "product_weight",
        "Вес упаковки, кг": "product_height",
        Гарантия: "guarantee",
        Схема: "scheme",
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
        header: false, // Do not treat the first row as headers automatically.
        skipEmptyLines: false, // Include all rows.
        complete: async (results) => {
          const rawData = results.data;

          if (rawData.length === 0) {
            this.uploadError = "The file contains no data.";
            this.uploadInProgress = null;
            return;
          }

          // Manually extract headers and rows
          const headers = rawData[0]; // First row as headers
          const dataRows = rawData
            .slice(1)
            .filter((row) => row.some((cell) => cell.trim() !== "")); // Remove rows with only empty cells.

          if (dataRows.length === 0) {
            this.uploadError = "The file contains no data rows.";
            this.uploadInProgress = null;
            return;
          }

          // Filter out empty rows
          dataRows = dataRows.filter((row) =>
            row.some((cell) => cell && cell.trim() !== "")
          );

          if (dataRows.length === 0) {
            this.uploadError = "The file contains only empty rows.";
            this.uploadInProgress = null;
            return;
          }

          try {
            this.uploadInProgress = "Uploading data to the server...";
            const products = [];
            const payload = [];
            const usedProductIds = new Set();

            dataRows.forEach((row, index) => {
              const rowObject = headers.reduce((acc, header, i) => {
                acc[header] = row[i] || ""; // Map header to value
                return acc;
              }, {});

              const mappedRow = this.mapCsvToDb(rowObject);
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
                name: mappedRow.name || `Unnamed Product ${index}`, // Handle missing names.
                sku: mappedRow.sku,
                product_image: mappedRow.product_image || "",
                factory_part_num: mappedRow.factory_part_num || "",
                category_id: "1",
              });

              payload.push({
                product_id: uniqueId,
                style: mappedRow.style || "",
                dimensions: mappedRow.dimensions || "",
                form: mappedRow.form || "",
                frame: mappedRow.frame || "",
                material: mappedRow.material || "",
                backlight_placement: mappedRow.backlight_placement || "",
                backlight_color: mappedRow.backlight_color || "",
                switch_type: mappedRow.switch_type || "",
                anti_fog_coating: mappedRow.anti_fog_coating || "",
                peculiarities: mappedRow.peculiarities || "",
                product_weight: parseFloat(mappedRow.product_weight) || "",
                product_height: parseFloat(mappedRow.product_height) || "",
                guarantee: mappedRow.guarantee || "",
                scheme: mappedRow.scheme || "",
                datetime_modified: new Date().toISOString(),
              });
            });

            await axios.post(
              process.env.VUE_APP_BASE_URL + "/products/bulk",
              products
            );
            await axios.post(
              process.env.VUE_APP_BASE_URL + "/info_mirror_cabinets/bulk",
              payload
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
