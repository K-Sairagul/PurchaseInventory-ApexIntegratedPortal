<template>
  <html lang="en">

  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css" />
  </head>

  <body>
    <input type="checkbox" id="nav-toggle" />
    <SidebarMain />
    <!-- Include the Sidebar component here -->
    <div class="main-content">
      <HeaderMain :userProfile="userProfile" :userName="userName" :userRole="userRole" title="Product Lists" />
      <main>
        <div class="table-responsive">
          <div class="container">
            <div class="search-bar-product">
              <div class="product-container">
                <input type="text" name="search" v-model="searchKeyword" placeholder="Search..." class="search-input" />
                <a href="#" class="search-btn">
                  <i class="fas fa-search"></i>
                </a>
              </div>
              <div class="button-container">
                <button class="btn" @click="openPopup">
                  <i class="las la-paper-plane mr-2"></i> Request Product
                </button>
              </div>
            </div>
            <div class="table-container">
              <table class="table">
                <thead>
                  <tr>
                    <th><i class="fas fa-id-badge mr-1"></i> Product ID</th>
                    <th><i class="fas fa-tag mr-1"></i> Product Name</th>
                    <th><i class="fas fa-image mr-1"></i> Product Image</th>
                    <th><i class="fas fa-building mr-1"></i> Company</th>
                    <th class="description-column">
                      <i class="fas fa-file-alt mr-1"></i> Description
                    </th>
                    <th><i class="fas fa-sort-amount-up mr-1"></i> Quantity</th>
                  </tr>
                </thead>
                <tbody v-if="filteredProducts.length > 0">
                  <tr v-for="(product, index) in filteredProducts" :key="index">
                    <td style="color: rgb(61, 61, 190)">{{ product.productId }}</td>
                    <td>{{ product.productName }}</td>
                    <td>
                      <img :src="getProductImageUrl(product.productId)" :alt="product.productName"
                        style="max-width: 100px" />
                    </td>
                    <td>{{ product.company }}</td>
                    <td class="description-column">
                      <div v-if="product.description.length > 70">
                        <div v-if="!product.showFullDescription">
                          {{ product.description.slice(0, 70) }}...
                          <span @click="toggleDescription(index)">
                            <div style="color: blueviolet">Read more</div>
                          </span>
                        </div>
                        <div v-else>
                          {{ product.description }}
                          <span @click="toggleDescription(index)">
                            <div style="color: blueviolet">Show less</div>
                          </span>
                        </div>
                      </div>
                      <div v-else>
                        {{ product.description }}
                      </div>
                    </td>
                    <td>{{ product.quantity }}</td>
                  </tr>
                </tbody>
                <tbody v-else>
                  <tr>
                    <td colspan="7" style="text-align: center">No data available</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </main>
    </div>
  </body>
  <div class="popup-overlay" v-if="showPopup">
    <ProductRequest @close="closePopup" />
  </div>

  </html>
</template>

<script>
import SidebarMain from "@/views/head/components/Sidebar.vue";
import HeaderMain from "@/views/head/components/HeaderMain.vue";
import ProductRequest from "@/views/head/components/ProductRequest.vue";

export default {
  name: "Product_Head",
  components: {
    SidebarMain,
    HeaderMain,
    ProductRequest,
  },
  data() {
    return {
      userProfile: {
        profilePhoto:
          "https://i.pinimg.com/736x/6b/aa/98/6baa98cc1c3f4d76e989701746e322dd.jpg",
      },
      userName: "",
      userRole: "",
      products: [],
      searchKeyword: "",
      showPopup: false,
      category: "",
    };
  },
  async mounted() {
    const userData = JSON.parse(sessionStorage.getItem("userData"));
    if (userData && userData.email) {
      this.email = userData.email; // Set email if available

      // Fetch user information
      fetch("http://localhost:3000/getUserInfoos", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ email: userData.email }),
      })
        .then((response) => {
          if (!response.ok) {
            throw new Error("Network response was not ok");
          }
          return response.json();
        })
        .then((data) => {
          const { username, role, department, specialLab } = data;
          this.userName = username;
          this.userRole = role;
          this.category = department || specialLab;
          this.fetchProductsByCategory();
        })
        .catch((error) => {
          console.error("Error fetching user info:", error);
        });

      // Update profile photo if available in sessionStorage
      if (userData && userData.profilePhoto) {
        this.userProfile.profilePhoto = userData.profilePhoto;
      }
    } else {
      console.error("User data not found in sessionStorage");
    }
  },

  computed: {
    filteredProducts() {
      return this.products.filter(
        (product) =>
          product.productId
            .toString()
            .toLowerCase()
            .includes(this.searchKeyword.toLowerCase()) ||
          product.productName.toLowerCase().includes(this.searchKeyword.toLowerCase()) ||
          product.company.toLowerCase().includes(this.searchKeyword.toLowerCase()) ||
          product.description.toLowerCase().includes(this.searchKeyword.toLowerCase())
      );
    },
  },

  methods: {
    async fetchProductsByCategory() {
      try {
        const response = await fetch("http://localhost:3000/getProductsByCategory", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ email: this.email }),
        });
        const data = await response.json();
        this.products = data;
      } catch (error) {
        console.error("Error fetching products:", error);
      }
    },
    getProductImageUrl(productId) {
      return `http://localhost:3000/products/images/${productId}`;
    },
    toggleDescription(index) {
      // Use native JavaScript assignment to set the property
      this.filteredProducts[index].showFullDescription = !this.filteredProducts[index]
        .showFullDescription;
    },
    openPopup() {
      this.showPopup = true;
    },
    closePopup() {
      this.showPopup = false;
    },
  },
};
</script>


<style scoped>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  list-style-type: none;
  text-decoration: none;
}

body {
  font-family: "Poppins", sans-serif;
}

.main-content {
  margin-top: 60px;
  padding: 2px 1.5rem;
  min-height: calc(100vh - 60px);
  background: #f5efff;
  border-radius: 5px;
  margin-left: 345px;
  transition: margin-left 300ms;
  width: -100%;
  /* Adjust to match your sidebar width */
  font-family: "Poppins", sans-serif;
}

.remove-btn {
  background-color: #fddb00;
  /* Light red background */
  color: #333;
  /* Dark text color */
  cursor: pointer;
  width: fit-content;
  height: fit-content;
  font-size: 12px;
  text-align: center;
  transition: background-color 0.3s, color 0.3s;
  box-shadow: 4px 4px 6px 0 rgba(255, 255, 255, 0.3),
    -4px -4px 6px 0 rgba(116, 125, 136, 0.2),
    inset -4px -4px 6px 0 rgba(255, 255, 255, 0.2), inset 4px 4px 6px 0 rgba(0, 0, 0, 0.2);
  border-radius: 30px;
  padding: 10px 20px;
}

.remove-btn:hover {
  background-color: #cc0000;
  /* Dark red background on hover */
  color: #fff;
  /* White text color on hover */
}

.remove-btn i {
  margin-right: 5px;
  /* Add some spacing between icon and text */
}

.table-responsive {
  overflow-x: auto;
  padding: 1rem 1.5rem;
  align-items: center;
  justify-content: center;
}

.edit {
  background-color: #fddb00;
  /* Light red background */
  color: #333;
  /* Dark text color */
  cursor: pointer;
  width: fit-content;
  height: fit-content;
  font-size: 12px;
  text-align: center;
  transition: background-color 0.3s, color 0.3s;
  box-shadow: 4px 4px 6px 0 rgba(255, 255, 255, 0.3),
    -4px -4px 6px 0 rgba(116, 125, 136, 0.2),
    inset -4px -4px 6px 0 rgba(255, 255, 255, 0.2), inset 4px 4px 6px 0 rgba(0, 0, 0, 0.2);
  border-radius: 30px;
  padding: 10px 20px;
}

.edit:hover {
  background-color: #cc0000;
  /* Dark red background on hover */
  color: #fff;
  /* White text color on hover */
}

.table-container {
  max-height: 600px;
  /* Set a maximum height for scroll */
  overflow-y: auto;
  /* Enable vertical scrolling */
}

.table {
  border-collapse: collapse;
  border-spacing: 0 10px;
  text-align: center;
  width: 100%;
  box-shadow: 2px 2px 2px 2px rgba(0, 0, 0, 0.1);
  /* Add a subtle shadow */
  border-radius: 8px;
  /* Rounded corners for aesthetics */
}

/* Apply styles to table header cells */
.table thead {
  box-shadow: 4px 4px 6px 0 rgba(255, 255, 255, 0.3),
    -4px -4px 6px 0 rgba(116, 125, 136, 0.2),
    inset -4px -4px 6px 0 rgba(255, 255, 255, 0.2), inset 4px 4px 6px 0 rgba(0, 0, 0, 0.2);
  border-radius: 30px;
  padding: 10px 20px;
}

.table thead th {
  color: #1a1a60;
  position: sticky;
  top: 0;
  background: #c6b3e8;
  z-index: 1;
  /* Ensure header stays above content */
}

th,
td {
  padding: 8px;
  border-bottom: 1px solid #ddd;
  white-space: normal;
  /* Allow text wrapping */
  overflow: hidden;
  /* Hide overflow text */
}

/* Set specific width for columns */
.description-column {
  width: 40%;
  max-width: 150px;
  /* Adjust max-width as needed */
  text-align: left;
}

.search-bar-product {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.search-bar-product input {
  width: 20%;
  border-radius: 20px;
  background-color: rgb(187, 184, 184);
  height: 40px;
  text-align: center;
  color: #000000;
  -webkit-text-fill-color: #000000;
}

tr td {
  padding: 23px 0;
  /* Adjust as needed */
}

.btn {
  width: fit-content;
  height: fit-content;
  font-size: 16px;
  font-weight: bolder;
  background: linear-gradient(to bottom, #846ea9, #b9a7d8);
  color: rgb(0, 0, 0);

  border-radius: 30px;
  padding: 10px 20px;
}

.btn:hover {
  background: linear-gradient(to bottom, #270561, #4f16b1);
  font-weight: bolder;
  color: #fff;
}

.export-btn {
  width: fit-content;
  height: fit-content;
  font-size: 16px;
  font-weight: bolder;
  background: linear-gradient(to bottom, rgb(51, 187, 60), rgb(36, 106, 47));
  color: rgb(0, 0, 0);
  border-radius: 30px;
  padding: 10px 20px;
}

.export-btn:hover {
  background: linear-gradient(to bottom, rgb(0, 80, 14), rgb(0, 78, 8));
  font-weight: bolder;
  color: #fff;
}

#nav-toggle:checked~.main-content {
  margin-left: 70px;
}

.table-container::-webkit-scrollbar {
  width: 10px;
  /* width of the entire scrollbar */
}

.table-container::-webkit-scrollbar-track {
  background: #fff;
  /* color of the tracking area */
}

.table-container::-webkit-scrollbar-thumb {
  background-color: #000000;
  /* color of the scroll thumb */
  border-radius: 20px;
  /* roundness of the scroll thumb */
  border: 3px solid rgb(255, 255, 255);
  /* creates padding around scroll thumb */
}

/* Popup */
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  /* semi-transparent overlay */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  /* ensure it appears above other content */
}

.product-container {
  background: linear-gradient(to bottom, #846ea9, #b9a7d8);
  height: 45px;
  border-radius: 30px;
  padding: 10px 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition: 0.8s;
}

.product-container:hover>.search-input {
  width: 270px;
}

.product-container .search-input {
  background: transparent;
  border: none;
  outline: none;
  width: 0px;
  font-weight: 500;
  font-size: 16px;
  transition: 0.8s;
}

.product-container .search-btn .fas {
  color: #000000;
}

.button-container {
  display: flex;
  gap: 10px;
  /* Adjust spacing between buttons */
}

@keyframes hoverShake {
  0% {
    transform: skew(0deg, 0deg);
  }

  25% {
    transform: skew(5deg, 5deg);
  }

  75% {
    transform: skew(-5deg, -5deg);
  }

  100% {
    transform: skew(0deg, 0deg);
  }
}

.product-container:hover {
  animation: hoverShake 0.15s linear 3;
}

@media only screen and (max-width: 1200px) {
  .main-content {
    margin-left: 70px;
  }

  .main-content header {
    width: calc(100% - 70px);
    /* Corrected */
    left: 70px;
  }
}

@media only screen and (max-width: 768px) {
  .main-content {
    width: 100%;
    margin-left: 0rem;
  }

  #nav-toggle:checked~.main-content {
    margin-left: 0rem !important;
  }
}

/* Hide scrollbar for the entire component */
</style>
