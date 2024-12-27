<template>
  <div class="p-6 bg-gray-50 min-h-screen">
    <h1 class="text-2xl font-bold mb-4">раковина на столешнице</h1>

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

        "Вес упаковки, кг": "package_weight",
        Артикул: "sku",
        "Артикул от завода": "factory_part_num",
        Коллекция: "collection",
        Название: "name",
        Поверхность: "product_equipment",
        Форма: "color",
        "Габариты (ш*г*в)": "surface",
        Код: "material",
        Гарантия: "guarantee",
        "Вес товара, кг": "product_weight",
        "Тип раковины": "sink_type",
        "Форма раковины": "shell_shape",
        "Количество раковин": "num_shells",
        "Материал раковины": "shell_material",
        "Габариты раковины (ш*г*в)": "sink_dimensions",
        "Материал столешницы": "table_top_material",
        "Габариты столешницы (ш*г*в)": "table_top_dimensions",
        "Толщина столешницы, мм": "table_top_thickness",
        "Расположение столешницы": "table_top_location",
        "Расположение смесителя": "mixer_location",
        "Диаметр слива раковины, мм": "sink_diameter",
        "Для сифона диаметром, мм": "siphon_diameter",
        "Готовые отверстия под смеситель": "ready_made_holes_for_mixer",
        Перелив: "overflow",
        "Донный клапан": "bottom_valve",
        "Возможное основание под раковину": "sink_possible_base",
        "Фото 1": "photo",
        Схема: "scheme",
        Стиль: "style",
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
                style: mappedRow.style || null,
                color: mappedRow.color || null,
                sink_type: mappedRow.sink_type || null,
                shell_shape: mappedRow.shell_shape || null,
                num_shells: parseInt(mappedRow.num_shells, 10) || null,
                shell_material: mappedRow.shell_material || null,
                sink_dimensions: mappedRow.sink_dimensions || null,
                table_top_material: mappedRow.table_top_material || null,
                table_top_dimensions: mappedRow.table_top_dimensions || null,
                table_top_thickness:
                  parseFloat(mappedRow.table_top_thickness) || null,
                table_top_location: mappedRow.table_top_location || null,
                mixer_location: mappedRow.mixer_location || null,
                sink_diameter: parseFloat(mappedRow.sink_diameter) || null,
                siphon_diameter: parseFloat(mappedRow.siphon_diameter) || null,
                ready_made_holes_for_mixer:
                  mappedRow.ready_made_holes_for_mixer || null,
                overflow: mappedRow.overflow || null,
                bottom_valve: mappedRow.bottom_valve || null,
                sink_possible_base: mappedRow.sink_possible_base || null,
                guarantee: mappedRow.guarantee || null,
                product_weight: parseFloat(mappedRow.product_weight) || null,
                package_weight: parseFloat(mappedRow.package_weight) || null,
                datetime_modified: new Date().toISOString(),
                scheme: mappedRow.scheme || null,
              });
            });

            await axios.post(
              process.env.VUE_APP_BASE_URL + "/products/bulk",
              products
            );
            await axios.post(
              process.env.VUE_APP_BASE_URL + "/info_counter_top_sink/bulk",
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
