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
        title="Dashboard"
      />
      <!-- Include the Header component here -->
      <main>
        <div class="cards">
          <div class="card-single">
            <div>
              <h1>54</h1>
              <span>Customers</span>
            </div>
            <div>
              <span class="las la-users"></span>
            </div>
          </div>
          <div class="card-single">
            <div>
              <h1>79</h1>
              <span>Project</span>
            </div>
            <div>
              <span class="las la-clipboard-list"></span>
            </div>
          </div>
          <div class="card-single">
            <div>
              <h1>124</h1>
              <span>Orders</span>
            </div>
            <div>
              <span class="las la-shopping-cart"></span>
            </div>
          </div>
          <div class="card-single">
            <div>
              <h1><span>&#8377;</span>60000</h1>
              <span>Income</span>
            </div>
            <div>
              <span class="las la-wallet" style="color: white"></span>
            </div>
          </div>
        </div>

        <div class="recent-grid">
          <div class="projects">
            <div class="card">
              <div class="card-header">
                <h2>Recent Activities</h2>
                <button>See all<span class="las la-arrow-right"></span></button>
              </div>
              <div class="card-body">
                <div class="table-responsive">
                  <table width="100%">
                    <thead>
                      <tr>
                        <td>Product Title</td>
                        <td>DepartMent</td>
                        <td>Status</td>
                      </tr>
                    </thead>
                    <tbody>
                      <tr>
                        <td>UI/UX Design</td>
                        <td>UI Team</td>
                        <td><span class="status purple"></span>inprogress</td>
                      </tr>
                      <tr>
                        <td>Web development</td>
                        <td>Ushop app</td>
                        <td><span class="status pink"></span>pending</td>
                      </tr>
                      <tr>
                        <td>UI/UX Design</td>
                        <td>UI Team</td>
                        <td><span class="status orange"></span>inprogress</td>
                      </tr>
                      <tr>
                        <td>Web development</td>
                        <td>Ushop app</td>
                        <td><span class="status orange"></span>pending</td>
                      </tr>
                      <tr>
                        <td>UI/UX Design</td>
                        <td>UI Team</td>
                        <td><span class="status orange"></span>inprogress</td>
                      </tr>
                      <tr>
                        <td>Web development</td>
                        <td>Ushop app</td>
                        <td><span class="status orange"></span>pending</td>
                      </tr>
                      <tr>
                        <td>UI/UX Design</td>
                        <td>UI Team</td>
                        <td><span class="status pink"></span>inprogress</td>
                      </tr>
                      <tr>
                        <td>UI/UX Design</td>
                        <td>UI Team</td>
                        <td><span class="status purple"></span>inprogress</td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </div>
          <div class="task-container">
            <div class="task-header">
              <h3>Today's Tasks</h3>
            </div>
            <div class="task-body">
              <!-- Todo List -->
              <div class="todo-list">
                <input
                  type="text"
                  placeholder="📑 Add a task"
                  v-model="newTaskName"
                  class="add-input"
                />
                <button @click="validateAndAddTask" class="add-button">
                  <span class="las la-plus-circle plus-icon"></span>
                </button>
              </div>
              <table class="task-table">
                <thead>
                  <tr>
                    <th class="task-name-cell">Task Name</th>
                    <th class="action-cell">Action</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="task in tasks" :key="task._id">
                    <td class="task-name-cell">{{ task.taskName }}</td>
                    <td class="action-cell">
                      <button @click="removeTask(task._id)" class="remove-task-button">
                        <span class="las la-minus-circle remove-btn"></span>
                      </button>
                    </td>
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
  name: "DashboardInventory",
  components: {
    SidebarMain,
    HeaderMain, // Register the Sidebar component
  },
  data() {
    return {
      userProfile: {
        profilePhoto:
          "https://i.pinimg.com/736x/6b/aa/98/6baa98cc1c3f4d76e989701746e322dd.jpg",
      },
      userName: "",
      userRole: "",
      title: "",
      tasks: [],
      newTaskName: "",
      email: "", // Add email to data
    };
  },
  mounted() {
    const userData = JSON.parse(sessionStorage.getItem("userData"));
    if (userData && userData.email) {
      this.email = userData.email; // Set email if available

      fetch("http://localhost:3000/getUserInfo", {
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
          const { username, role } = data;
          this.userName = username;
          this.userRole = role;
          if (role !== "Manager") {
            // Redirect to unauthorized page if the user is not a manager
            this.$router.push({ name: "UnauthorizedPage" });
          }
        })
        .catch((error) => {
          console.error("Error fetching user info:", error);
        });
    } else {
      console.error("User data not found in sessionStorage");
    }

    // Update profile photo if available in sessionStorage
    if (userData && userData.profilePhoto) {
      this.userProfile.profilePhoto = userData.profilePhoto;
    }

    // Fetch tasks on component mount
    this.fetchTasks();
  },
  methods: {
    validateAndAddTask() {
      if (this.newTaskName.trim() !== "") {
        this.addTask(); // Call addTask if input is not empty
      }
    },
    async fetchTasks() {
      try {
        const userData = JSON.parse(sessionStorage.getItem("userData"));
        const email = userData.email;

        const response = await fetch(`http://localhost:3000/getTasks?email=${email}`, {
          method: "GET",
          headers: {
            "Content-Type": "application/json",
          },
        });
        const data = await response.json();
        if (response.ok) {
          this.tasks = data.tasks;
        } else {
          throw new Error(data.message || "Error fetching tasks");
        }
      } catch (error) {
        console.error("Error fetching tasks:", error);
      }
    },

    async addTask() {
      try {
        const userData = JSON.parse(sessionStorage.getItem("userData"));
        const email = userData.email;

        const response = await fetch("http://localhost:3000/addTask", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ taskName: this.newTaskName, email }),
        });
        const data = await response.json();
        if (response.ok) {
          this.tasks.push(data.task); // Add the new task to the UI
          this.newTaskName = ""; // Clear input
        } else {
          throw new Error(data.message || "Error adding task");
        }
      } catch (error) {
        console.error("Error adding task:", error);
      }
    },

    async removeTask(taskId) {
      try {
        const userData = JSON.parse(sessionStorage.getItem("userData"));
        const email = userData.email;

        const response = await fetch("http://localhost:3000/removeTask", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ taskId, email }),
        });
        if (response.ok) {
          this.tasks = this.tasks.filter((task) => task._id !== taskId); // Update UI after removing task
        } else {
          throw new Error("Error removing task");
        }
      } catch (error) {
        console.error("Error removing task:", error);
      }
    },
  },
};
</script>

<style scoped>
.img-inventory {
  width: 40px;
  height: 40px; /* Adjust height as needed */
  margin-right: 10px; /* Add spacing between image and text */
}

* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  list-style-type: none;
  text-decoration: none;
}

body {
  font-family: "Poppins", sans-serif;
  background: #f5efff;
}

.slidebar {
  width: 345px;
  position: fixed;
  height: 100%;
  left: 0;
  top: 0;
  background: #e5d9f2;
  z-index: 100;
  transition: width 300ms;
}

.slidebar-brand {
  height: 90px;
  padding-left: 1rem 0rem 1rem 2rem;
  color: #000000;
  align-content: center;
}

.slidebar-brand span {
  display: inline-block;
  padding-right: 1rem;
}

.slidebar-brand h2 {
  font-family: "Kalam", sans-serif; /* Use Kalam font */
  font-weight: 700; /* Bold weight */
  display: flex;
  font-size: 30px;
  align-items: center;
}

.sliderbar-menu {
  margin-top: 1rem;
}

.sliderbar-menu li {
  width: 100%;
  margin-bottom: 1.7rem;
  padding-left: 1rem;
}

.sliderbar-menu a {
  padding-left: 1rem;
  display: block;
  color: #000000;
  font-size: 1.1rem;
}

.sliderbar-menu a span:first-child {
  font-size: 1.5rem;
  padding-right: 1rem;
}

.sliderbar-menu a.active {
  padding-top: 1rem;
  padding-bottom: 1rem;
  background: #cdc1ff;
  border-radius: 30px 0px 0px 30px;
}

#nav-toggle:checked + .slidebar {
  width: 70px;
}

#nav-toggle:checked + .slidebar .slidebar-brand,
#nav-toggle:checked + .slidebar li {
  padding-left: 1rem;
}

#nav-toggle:checked + .slidebar li a {
  padding-left: 0rem;
}

#nav-toggle:checked + .slidebar .slidebar-brand h2 span:last-child,
#nav-toggle:checked + .slidebar li a span:last-child {
  display: none;
}

#nav-toggle:checked ~ .main-content {
  margin-left: 70px;
}

#nav-toggle:checked ~ .main-content header {
  width: calc(100% - 70px); /* Corrected */
  left: 70px;
}
.main-content {
  transition: margin-left 300ms;
  margin-left: 345px;
}

header {
  display: flex;
  padding: 1rem 1.5rem;
  justify-content: space-between;
  box-shadow: 2px 2px 4px #d5bfed;
  position: fixed;
  left: 345px;
  width: calc(100% - 345px);
  top: 0;
  z-index: 100;
  transition: left 300ms;
}

#nav-toggle {
  display: none;
}

header h2 {
  color: #222;
}

header label span {
  font-size: 1rem;
  padding-right: 1rem;
}

.search-wrapper {
  border: 1px solid #ccc;
  border-radius: 30px;
  height: 50px;
  display: flex;
  align-items: center;
  overflow-x: hidden;
}

.search-wrapper span {
  display: inline-block;
  padding: 0rem 1rem;
  font-size: 1.5rem;
}

.search-wrapper input {
  height: 100%;
  padding: 0.5rem;
  border: none;
  outline: none;
}

.user-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
}

.user-wrapper img {
  border-radius: 50%;
  margin-right: 0.5rem;
}

.user-wrapper small {
  display: inline-block;
  color: rgb(249, 0, 0);
  margin-top: -1px !important;
}

.user-info {
  font-family: "Philosopher", sans-serif;
  font-weight: 700;
  font-style: normal;
}

.user-info small {
  font-size: 17.5px;
}

main {
  margin-top: 90px;
  padding: 2px 1.5rem;
  min-height: calc(100vh - 90px);
}

.cards {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 2rem;
  margin-top: 1rem;
}

.card-single {
  display: flex;
  justify-content: space-between;
  background-color: #d4bee4;
  padding: 2rem;
  border-radius: 2px;
  color: #1a1a60;
}

.card-single div:last-child span {
  font-size: 3rem;
  color: #2a004e;
}

.card-single div:first-child span {
  color: #424642;
}

.card-single:last-child {
  background: #4a007e;
}
.card-single:last-child h1,
.card-single:last-child div:first-child span,
.card-single:last-child div:first-child {
  color: #fff;
}

.recent-grid {
  margin-top: 3.5rem;
  display: grid;
  grid-gap: 2rem;
  grid-template-columns: 65% auto;
}

.card {
  background: #d4bee4;
  color: #1a1a60;
  border-radius: 5px;
}

.card-header,
.card-body {
  padding: 1rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: #f0f0f0;
}

.card-header button {
  background: #4a007e;
  border-radius: 10px;
  color: white;
  font-size: 0.8rem;
  padding: 0.5rem 1rem;
  border: 1px solid #4a007e; /* Match the border color with the background */
  transition: background-color 0.3s ease, border-color 0.3s ease; /* Smooth transition for hover effects */
}

.card-header button:hover {
  background: #6a00b0; /* Slightly lighter shade for hover effect */
  border-color: #6a00b0; /* Match the border color with the hover background */
}

/* task */

.task-container {
  background-color: #d4bee4; /* Set background color */
  border-radius: 10px; /* Increase border radius for smoother corners */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* Subtle shadow for depth */
  padding: 20px; /* Increase padding for more space */
  color: #1a1a60; /* Set text color for contrast */
}
.task-header h3 {
  padding: 15px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #1a1a60; /* Set header text color */
}
.task-body {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  margin-left: 8px;
  margin-right: 8px;
}

.todo-list {
  display: flex;
  justify-content: space-between; /* Space items evenly */
  align-items: center; /* Align items vertically */
  width: 100%; /* Ensure full width */
}
.add-input {
  width: 80%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 50px;
  font-size: 16px;
  outline: none; /* Remove default focus outline */
  transition: border-color 0.3s ease; /* Smooth transition for border color */
  cursor: pointer; /* Custom cursor */
}

/* Style for the input box when focused */
.add-input:focus {
  border-color: #1a1a60; /* Change border color on focus */
}

.add-button {
  width: 20%; /* Set button width to 20% */
  margin-left: 10px; /* Add margin as needed */
}

ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin-bottom: 8px;
}

.plus-icon {
  font-size: 1.5rem;
  border-radius: 50%;
  background-color: #28a745;
  color: #fff;
  padding: 0.5rem;
}

/* Light green hover state */
.plus-icon:hover {
  background-color: #188a33;
}

/* Dark green hover state */
.plus-icon:hover:active {
  background-color: #218838;
}
.remove-btn {
  font-size: 1.5rem;
  border-radius: 50%;
  background-color: #dc3545;
  color: #fff;
  padding: 0.5rem;
  margin-left: 10px;
  cursor: pointer;
}

/* Light red hover state */
.remove-btn:hover {
  background-color: #ff6b6b;
}

/* Dark red hover state */
.remove-btn:hover:active {
  background-color: #c82333;
}

.task-table {
  width: 100%; /* Ensure the table takes full width */
  margin-top: 2%;
}

.task-table th {
  padding: 8px; /* Adjust padding as needed */
  text-align: left; /* Align text to the left */
  color: #1a1a60; /* Set header text color */
}

.task-table td.task-name-cell {
  width: 80%; /* Set 80% width for task name cells */
  padding: 8px; /* Adjust padding as needed */
}
.task-table td.action-cell {
  width: 20%; /* Set 20% width for action cells */
  text-align: center; /* Center align content in action cells */
}

/* */

table {
  border-collapse: collapse;
}

thead tr {
  border-top: 1px solid #000000;
  border-bottom: 1px solid #000000;
}

thead td {
  font-weight: 700;
  color: #2a004e;
}

td {
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
  color: #1a1a60;
}

td .status {
  height: 10px;
  width: 10px;
  border-radius: 50%;
  display: inline-block;
  margin-right: 1rem;
}

tr td:last-child {
  display: flex;
  align-items: center;
}

.status.purple {
  background: rebeccapurple;
}

.status.pink {
  background: deeppink;
}
.status.orange {
  background: orangered;
}

.table-responsive {
  width: 100%;
  overflow-x: auto;
}

.customer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0.7rem;
}
.info {
  display: flex;
  align-items: center;
}
.info img {
  border-radius: 50%;
  margin-right: 1rem;
}

.info h4 {
  font-size: 0.8rem;
  font-weight: 700;
  color: #222;
}

.info small {
  font-weight: 600;
  color: rgb(255, 0, 0);
}

.contact span {
  font-size: 1.2rem;
  display: inline-block;
  margin-left: 0.5rem;
  color: #838282;
}

@media only screen and (max-width: 1200px) {
  .slidebar {
    width: 70px;
  }

  .slidebar .slidebar-brand,
  .slidebar li {
    padding-left: 1rem;
  }

  .slidebar li a {
    padding-left: 0rem;
  }

  .slidebar .slidebar-brand h2 span:last-child,
  .slidebar li a span:last-child {
    display: none;
  }

  .main-content {
    margin-left: 70px;
  }

  .main-content header {
    width: calc(100% - 70px); /* Corrected */
    left: 70px;
  }

  .slidebar:hover {
    width: 345px;
    z-index: 200;
  }

  .slidebar:hover .slidebar-brand,
  .slidebar:hover li {
    padding-left: 2rem;
    text-align: center;
  }

  .slidebar:hover li a {
    padding-left: 1rem;
  }

  .slidebar:hover .slidebar-brand h2 span:last-child,
  .slidebar:hover li a span:last-child {
    display: inline;
  }
}

@media only screen and (max-width: 960px) {
  .cards {
    grid-template-columns: repeat(3, 1fr);
  }

  .recent-grid {
    grid-template-columns: 60% 40%;
  }
}

@media only screen and (max-width: 768px) {
  .cards {
    grid-template-columns: repeat(2, 1fr);
  }

  .recent-grid {
    grid-template-columns: 100%;
  }

  .search-wrapper {
    display: none;
  }

  .slidebar {
    left: -100% !important;
  }

  header h2 {
    display: flex;
    align-items: center;
  }

  header h2 label {
    display: inline-block;
    background: #838282;
    padding-right: 0rem;
    margin-right: 1rem;
    height: 40px;
    width: 40px;
    border-radius: 50%;
    color: #fff;
    display: flex;
    align-items: center;
    justify-content: center !important;
  }

  header h2 span {
    text-align: center;
    padding-right: 0rem;
  }

  header h2 {
    font-size: 1.1rem;
  }

  .main-content {
    width: 100%;
    margin-left: 0rem;
  }

  header {
    width: 100% !important;
    left: 0 !important;
  }

  #nav-toggle:checked + .slidebar {
    left: 0 !important;
    z-index: 100;
    width: 345px;
  }

  #nav-toggle:checked + .slidebar .slidebar-brand,
  #nav-toggle:checked + .slidebar li {
    padding-left: 2rem;
    text-align: left;
  }

  #nav-toggle:checked + .slidebar li a {
    padding-left: 1rem;
  }

  #nav-toggle:checked + .slidebar .slidebar-brand h2 span:last-child,
  #nav-toggle:checked + .slidebar li a span:last-child {
    display: inline;
  }

  #nav-toggle:checked ~ .main-content {
    margin-left: 0rem !important;
  }

  #nav-toggle:checked + .slidebar h2 {
    display: none;
  }
  #nav-toggle:checked + .slidebar .sliderbar-menu {
    justify-content: space-between;
    align-items: center;
    margin-left: 10;
  }
}

@media only screen and (max-width: 560px) {
  .cards {
    grid-template-columns: 100%;
  }
  #nav-toggle:checked + .slidebar {
    left: 0 !important;
    z-index: 100;
    width: 150px;
  }

  #nav-toggle:checked + .slidebar li {
    padding-left: 2rem;
    text-align: left;
  }

  #nav-toggle:checked + .slidebar li a {
    padding-left: 1rem;
    font-size: 12px;
  }
}

.main-content {
  padding-top: 80px; /* Adjust this value based on your header height */
}

/* Example of fixing header height */
.HeaderMain {
  height: 80px;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background-color: #fff; /* Change this to match your header background */
  z-index: 1000; /* Ensure the header is above other content */
}

/* Adjust main content position */
main {
  margin-top: 20px; /* Add margin to separate content from the fixed header */
}
</style>
