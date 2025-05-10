<template>
  <div class="popup-overlay">
    <div class="popup-content">
      <h2>Import Products</h2>
      <!-- Custom file input -->
      <label for="file-input" class="custom-file-input"> Choose File </label>
      <input
        id="file-input"
        type="file"
        @change="handleFileUpload"
        accept=".xlsx, .xls"
      />
      <span v-if="file" class="file-name">{{ file.name }}</span>
      <!-- Display file name -->
      <div v-if="previewData.length > 0">
        <h3>Preview</h3>
        <table>
          <thead>
            <tr>
              <th>Product ID</th>
              <th>Product Name</th>
              <th>Company</th>
              <th>Description</th>
              <th>Quantity</th>
              <th>Department</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in previewData" :key="index">
              <td>{{ item["Product ID"] }}</td>
              <td>{{ item["Product Name"] }}</td>
              <td>{{ item["Company"] }}</td>
              <td>{{ item["Description"] }}</td>
              <td>{{ item["Quantity"] }}</td>
              <td>{{ item["Department"] }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <button @click="importProducts">Import</button>
      <button class="close-btn" @click="close">Close</button>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";

export default {
  data() {
    return {
      file: null,
      previewData: [],
    };
  },
  methods: {
    handleFileUpload(event) {
      this.file = event.target.files[0]; // Store the selected file
      const reader = new FileReader();
      reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: "array" });
        const sheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[sheetName];
        this.previewData = XLSX.utils.sheet_to_json(worksheet);
      };
      reader.readAsArrayBuffer(this.file);
    },
    async importProducts() {
      if (!this.file) {
        alert("No file selected.");
        return;
      }

      const formData = new FormData();
      formData.append("file", this.file);

      try {
        const response = await fetch("http://localhost:3000/import-products", {
          method: "POST",
          body: formData,
        });

        if (response.ok) {
          alert("Products imported successfully");
          this.$emit("close");
          this.$emit("refresh");
        } else {
          const errorData = await response.json();
          alert(`Error importing products: ${errorData.error}`);
        }
      } catch (error) {
        console.error("Error:", error);
        alert("Error importing products");
      }
    },
    close() {
      this.$emit("close");
    },
  },
};
</script>

<style scoped>
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.popup-content {
  background: #f5efff; /* Use the provided background color */
  padding: 20px;
  border-radius: 12px; /* Rounded corners */
  width: 80%;
  max-width: 800px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2); /* Subtle shadow */
}

/* Hide the default file input */
input[type="file"] {
  display: none;
}

/* Custom file input button */
.custom-file-input {
  display: inline-block;
  padding: 10px 20px;
  background-color: #846ea9; /* Matching the table gradient */
  color: white;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.custom-file-input:hover {
  background-color: #b9a7d8; /* Lighter shade on hover */
}

.custom-file-input:active {
  background-color: #6a5a8f; /* Darker shade on click */
}

/* File name display */
.file-name {
  margin-left: 10px;
  color: #846ea9; /* Matching the table gradient */
  font-weight: bold;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
  background: linear-gradient(to bottom, #846ea9, #b9a7d8); /* Gradient for the table */
  border-radius: 8px; /* Rounded corners for the table */
  overflow: hidden; /* Ensure rounded corners are visible */
}

th,
td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
  color: #333; /* Darker text for better readability */
}

th {
  background-color: #846ea9; /* Darker shade for headers */
  color: white; /* White text for headers */
  font-weight: bold;
}

tr:nth-child(even) {
  background-color: rgba(255, 255, 255, 0.1); /* Light background for even rows */
}

tr:hover {
  background-color: rgba(255, 255, 255, 0.2); /* Hover effect for rows */
}

button {
  margin: 5px;
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  background-color: #846ea9; /* Matching the table gradient */
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #b9a7d8; /* Lighter shade on hover */
}

button:active {
  background-color: #6a5a8f; /* Darker shade on click */
}

/* Close button specific styling */
button.close-btn {
  background-color: #ff6b6b; /* Red color for close button */
}

button.close-btn:hover {
  background-color: #ff4c4c; /* Darker red on hover */
}

button.close-btn:active {
  background-color: #e53e3e; /* Even darker red on click */
}
</style>
