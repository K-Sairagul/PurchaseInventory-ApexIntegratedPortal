<template>
  <body>
    <input type="checkbox" id="nav-toggle" />
    <SidebarMain />
    <!-- Include the Sidebar component here -->
    <div class="main-content">
      <HeaderMain
        :userProfile="userProfile"
        :userName="userName"
        :userRole="userRole"
        title="Arrange Products"
      />
      <main>
        <div class="table-responsive">
          <div class="container">
            <div class="request-container">
              <input
                type="text"
                name="search"
                v-model="searchKeyword"
                placeholder="Search..."
                class="search-input"
              />
              <a href="#" class="search-btn">
                <i class="fas fa-search"></i>
              </a>
            </div>
            <div class="table-container">
              <table class="table">
                <thead>
                  <tr>
                    <th><i class="fas fa-list-ol mr-1"></i> Req No</th>
                    <th><i class="fas fa-envelope mr-1"></i> Email</th>
                    <th><i class="fas fa-tag mr-1"></i> Product Name</th>
                    <th><i class="fas fa-sort-amount-up mr-1"></i> Quantity</th>
                    <th><i class="fas fa-link mr-1"></i> Link</th>
                    <th><i class="fas fa-check-circle mr-1"></i> Status</th>
                  </tr>
                </thead>
                <tbody v-if="filteredProducts.length > 0">
                  <template
                    v-for="(request, reqIndex) in filteredProducts"
                    :key="reqIndex"
                  >
                    <tr
                      v-for="(product, prodIndex) in request.products"
                      :key="product._id"
                    >
                      <td
                        v-if="prodIndex === 0"
                        :rowspan="request.products.length"
                        class="req-email"
                      >
                        <div>{{ request.reqNo }}</div>
                      </td>
                      <td
                        v-if="prodIndex === 0"
                        :rowspan="request.products.length"
                        class="req-email"
                        style="width: 25%"
                      >
                        <div>{{ request.email }}</div>
                      </td>
                      <td>{{ product.productName }}</td>
                      <td style="width: 10%">{{ product.quantityNeeded }}</td>

                      <td style="width: 10%">
                        <a :href="product.link" target="_blank" rel="noopener noreferrer">
                          <i class="fas fa-external-link-alt mr-1"></i>
                        </a>
                      </td>
                      <td style="width: 20%">
                        <template v-if="product.status === 0">
                          <div>
                            <Button
                              class="btn-arrange mr-2"
                              @click="arrangeProduct(request.reqNo, product.productName)"
                              >Arrange</Button
                            >
                            <Button
                              class="btn-buy"
                              @click="buyProduct(request.reqNo, product.productName)"
                              >Buy</Button
                            >
                          </div>
                        </template>
                        <template v-else-if="product.status === 1">
                          <div>
                            <i class="fas fa-check-circle text-success"></i>
                            Product Arranged
                          </div>
                        </template>
                        <template v-else-if="product.status === 2">
                          <div>
                            <i class="fas fa-shopping-cart text-primary"></i>
                            Moved to Buy
                          </div>
                        </template>
                        <template v-else>
                          <a
                            :href="product.link"
                            target="_blank"
                            rel="noopener noreferrer"
                          >
                            <i class="fas fa-external-link-alt mr-1"></i>
                          </a>
                        </template>
                      </td>
                    </tr>
                  </template>
                </tbody>
                <tbody v-else>
                  <tr>
                    <td colspan="7">No data available</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </main>
    </div>
  </body>
</template>

<script>
import SidebarMain from "@/views/inventory/components/Sidebar.vue";
import HeaderMain from "@/views/inventory/components/HeaderMain.vue";

export default {
  name: "Arrange_Product_Inventory",
  components: {
    SidebarMain,
    HeaderMain,
  },

  data() {
    return {
      userProfile: {
        profilePhoto:
          "https://i.pinimg.com/736x/6b/aa/98/6baa98cc1c3f4d76e989701746e322dd.jpg",
      },
      userName: "",
      userRole: "",
      requests: [],
      searchKeyword: "",
    };
  },

  async mounted() {
    try {
      const userData = JSON.parse(sessionStorage.getItem("userData"));
      if (userData && userData.email) {
        const response = await fetch("http://localhost:3000/getUserInfo", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ email: userData.email }),
        });
        const data = await response.json();
        const { username, role } = data;
        this.userName = username;
        this.userRole = role;

        // Fetch products
        await this.fetchProducts();
      }
    } catch (error) {
      console.error("Error fetching user info:", error);
    }
  },

  computed: {
    filteredProducts() {
      if (!this.searchKeyword.trim()) {
        return this.requests;
      }

      const keyword = this.searchKeyword.toLowerCase().trim();
      return this.requests.filter((request) => {
        const idString = request.reqno ? request.reqno.toString() : "";
        const name = request.productName ? request.productName.toLowerCase() : "";
        const transferredQuantity = request.transferredQuantity
          ? request.transferredQuantity.toString()
          : "";
        const status = request.status ? request.status.toString() : "";

        // Check if any of the fields contains the keyword
        return (
          idString.includes(keyword) ||
          name.includes(keyword) ||
          transferredQuantity.includes(keyword) ||
          status.includes(keyword)
        );
      });
    },
  },

  methods: {
    async fetchProducts() {
      try {
        const response = await fetch("http://localhost:3000/getarrangeproduct");
        const data = await response.json();
        this.requests = data;
      } catch (error) {
        console.error("Error fetching products:", error);
      }
    },
    filterData() {
      // Assuming 'requests' is an array of objects with 'products' array inside each object
      this.filteredProducts = this.requests.map((item) => ({
        reqNo: item.reqNo,
        email: item.email,
        products: item.products, // Assuming 'products' contains the necessary data
      }));
    },
    async arrangeProduct(reqNo, productName) {
      await this.updateProductStatus(reqNo, productName, 1); // 1 represents "Arranged"
      await this.fetchProducts();
    },
    async buyProduct(reqNo, productName) {
      await this.updateProductStatus(reqNo, productName, 2); // 2 represents "Buy"
      await this.fetchProducts();
    },
    async updateProductStatus(reqNo, productName, status) {
      try {
        const response = await fetch("http://localhost:3000/arrange-update-status", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ reqNo, productName, status }),
        });
        if (!response.ok) {
          throw new Error("Network response was not ok");
        }
        const data = await response.json();
        console.log(data); // Handle response data as needed
      } catch (error) {
        console.error("There was a problem with the fetch operation:", error.message);
        // Handle errors as needed
      }
    },
    toggleDescription(index) {
      // Use native JavaScript assignment to set the property
      this.filteredProducts[index].showFullDescription = !this.filteredProducts[index]
        .showFullDescription;
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

.custom-popup {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.custom-feedback-modal {
  width: 40%;
  max-width: 300px; /* Adjust as needed */
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 8px;
}

.custom-header-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.custom-header-title {
  margin: 0;
}

.custom-close-button {
  background-color: green;
  color: #fff;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
}

.custom-close-icon {
  font-size: 16px;
}

.custom-feedback-textarea {
  width: 100%;
  height: 100px;
  resize: none;
  margin-bottom: 10px;
}

.custom-send-button {
  display: block;
  margin: 0 auto;
  background-color: green;
  color: #fff;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
}

.custom-send-button:hover {
  cursor: pointer;
  opacity: 0.8;
}

.main-content {
  margin-top: 60px;
  padding: 2px 1.5rem;
  min-height: calc(100vh - 60px);
  background: #f5efff;
  border-radius: 5px;
  margin-left: 345px;
  transition: margin-left 300ms;
  width: -100%; /* Adjust to match your sidebar width */
  font-family: "Poppins", sans-serif;
}

.close-button {
  align-items: center;
}
.send-msg {
  background-color: #70ee2c; /* Light red background */
  color: #333; /* Dark text color */
  cursor: pointer;
  width: fit-content;
  height: fit-content;
  font-size: 16px;
  text-align: center;
  transition: background-color 0.3s, color 0.3s;
  box-shadow: 4px 4px 6px 0 rgba(255, 255, 255, 0.3),
    -4px -4px 6px 0 rgba(116, 125, 136, 0.2),
    inset -4px -4px 6px 0 rgba(255, 255, 255, 0.2), inset 4px 4px 6px 0 rgba(0, 0, 0, 0.2);
  border-radius: 30px;
  padding: 10px 20px;
}

.feedback-modal {
  text-align: center;
}

.header-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
}

.close-button {
  color: red; /* Red color for the button */
  background-color: transparent;
  border: none;
  cursor: pointer;
}

.remove-btn:hover {
  background-color: #cc0000; /* Dark red background on hover */
  color: #fff; /* White text color on hover */
}

.remove-btn i {
  margin-right: 5px; /* Add some spacing between icon and text */
}

.table-responsive {
  overflow-x: auto;
  padding: 1rem 1.5rem;
  align-items: center;
  justify-content: center;
}

.edit {
  background-color: #fddb00; /* Light red background */
  color: #333; /* Dark text color */
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
  background-color: #cc0000; /* Dark red background on hover */
  color: #fff; /* White text color on hover */
}
.table-container {
  max-height: 600px; /* Set a maximum height for scroll */
  overflow-y: auto; /* Enable vertical scrolling */
}

.table {
  border-collapse: collapse;
  border-spacing: 0 10px;
  width: 100%;
  box-shadow: 2px 2px 2px 2px rgba(0, 0, 0, 0.1); /* Add a subtle shadow */
  border-radius: 8px; /* Rounded corners for aesthetics */
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
  z-index: 1; /* Ensure header stays above content */
}


th,
td {
  padding: 12px; /* Increased padding for better spacing */
  text-align: center;
  border-bottom: 1px solid #ddd;
  white-space: nowrap; /* Prevent text wrapping in table cells */
  overflow: hidden;
  text-overflow: ellipsis; /* Show ellipsis for overflowing text */
  vertical-align: middle; /* Center align content vertically */
}

/* Center Req No and Email cells */
td.req-email {
  width: 1%;
  white-space: nowrap;
}

td.req-email > div {
  display: inline-block;
  vertical-align: middle;
}

/* Set specific width for columns */
.description-column {
  width: 40%;
  max-width: 150px; /* Adjust max-width as needed */
  text-align: left;
}

.search-bar-request {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.search-bar-request input {
  width: 20%;
  border-radius: 20px;
  background-color: rgb(187, 184, 184);
  height: 40px;
  text-align: center;
  color: #000000;
  -webkit-text-fill-color: #000000;
}

tr td {
  padding: 23px 0; /* Adjust as needed */
}
.btn {
  width: fit-content;
  height: fit-content;
  font-size: 16px;
  font-weight: bolder;
  background: linear-gradient(to bottom, rgb(177, 178, 177), rgb(205, 205, 202));
  color: rgb(0, 0, 0);
  box-shadow: 4px 4px 6px 0 rgba(255, 255, 255, 0.3),
    -4px -4px 6px 0 rgba(116, 125, 136, 0.2),
    inset -4px -4px 6px 0 rgba(255, 255, 255, 0.2), inset 4px 4px 6px 0 rgba(0, 0, 0, 0.2);
  border-radius: 30px;
  padding: 10px 20px;
}

.btn:hover {
  background: linear-gradient(to bottom, rgb(102, 102, 102), rgb(102, 102, 102));
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

#nav-toggle:checked ~ .main-content {
  margin-left: 70px;
}

.table-container::-webkit-scrollbar {
  width: 10px; /* width of the entire scrollbar */
}

.table-container::-webkit-scrollbar-track {
  background: #fff; /* color of the tracking area */
}

.table-container::-webkit-scrollbar-thumb {
  background-color: #000000; /* color of the scroll thumb */
  border-radius: 20px; /* roundness of the scroll thumb */
  border: 3px solid rgb(255, 255, 255); /* creates padding around scroll thumb */
}

/* Popup */
.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); /* semi-transparent overlay */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000; /* ensure it appears above other content */
}

.request-container {
  background: linear-gradient(to bottom,#846ea9, #b9a7d8);
  height: 45px;
  width: fit-content;
  margin-bottom: 20px;
  border-radius: 30px;
  padding: 10px 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition: 0.8s;
}

.request-container:hover > .search-input {
  width: 270px;
}

.request-container .search-input {
  background: transparent;
  border: none;
  outline: none;
  width: 0px;
  font-weight: 500;
  font-size: 16px;
  transition: 0.8s;
}

.request-container .search-btn .fas {
  color: #000000;
}

.rupee-icon {
  font-size: 1.2em; /* Adjust size as needed */
  color: #07a904; /* Example color */
  margin-right: 2px;
}

.reject-btn {
  background: red;
  color: #000000;
  border-radius: 20px 20px;
  padding: 10px;
  font-size: 12px;
  font-weight: 500;
  box-shadow: 2px 2px 2px rgb(45, 43, 43);
}

.btn-arrange {
  background: rgb(75, 243, 75);
  color: #000000;
  border-radius: 20px 20px;
  padding: 10px;
  font-size: 10px;
  font-weight: 500;
  box-shadow: 2px 2px 2px rgb(45, 43, 43);
}
.btn-buy {
  margin-right: -30px;
  background: rgb(208, 239, 70);
  color: #000000;
  border-radius: 20px 20px;
  padding: 10px;
  font-size: 10px;
  font-weight: 500;
  box-shadow: 2px 2px 2px rgb(45, 43, 43);
  width: 50px;
}

.req-class {
  display: flex;
  justify-content: space-around;
  align-items: center;
}

.status.yellow {
  background: yellow;
}
.status.red {
  background: red;
}

.status.green {
  background: green;
}

td .status {
  height: 12px;
  width: 12px;
  border-radius: 50%;
  display: inline-block;
  margin-right: 1rem;
}

.modal {
  display: none;
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
}

.modal-content {
  background-color: #fefefe;
  margin: 15% auto; /* Center modal vertically and horizontally */
  padding: 20px;
  border: 1px solid #888;
  border-radius: 5px;
  max-width: 80%; /* Limit maximum width of the modal */
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
  cursor: pointer;
}

/* Feedback Modal Specific Styles */
.feedback-modal {
  max-width: 300px; /* Adjust the maximum width of the modal content */
}

/* Style the textarea */
#feedback {
  width: 100%;
  height: 100px; /* Adjust the initial height as needed */
  resize: vertical; /* Allow vertical resizing */
}

/* Style the send button */
.send-btn {
  margin-top: 10px;
  padding: 8px 15px;
  font-size: 14px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.send-btn:hover {
  background-color: #45a049;
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

.request-container:hover {
  animation: hoverShake 0.15s linear 3;
}

@media only screen and (max-width: 1200px) {
  .main-content {
    margin-left: 70px;
  }

  .main-content header {
    width: calc(100% - 70px); /* Corrected */
    left: 70px;
  }
}

@media only screen and (max-width: 768px) {
  .main-content {
    width: 100%;
    margin-left: 0rem;
  }

  #nav-toggle:checked ~ .main-content {
    margin-left: 0rem !important;
  }
}

/* Hide scrollbar for the entire component */
</style>
