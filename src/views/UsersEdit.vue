<template>
  <div class="users-edit">
    <div id="content">
      <div class="container">
        <div class="card p-4">
          <h3 class="card-title mb-2">User Information</h3>
          <hr class="mt-0" />
          <p>Username: {{ user.username }}</p>
          <p>Email: {{ user.email }}</p>
          <div>
            <button
              class="btn btn-primary"
              data-toggle="modal"
              data-target=".user-modal-lg"
            >
              Update
            </button>
            <button
              class="btn btn-primary"
              style="margin-left: 20px;"
              v-on:click="destroyUser()"
            >
              Delete Account
            </button>
          </div>
          <div
            class="modal fade user-modal-lg"
            tabindex="-1"
            role="dialog"
            aria-labelledby="myLargeModalLabel"
            aria-hidden="true"
          >
            <div class="modal-dialog modal-lg">
              <div class="modal-content">
                <div class="modal-header">
                  <button
                    type="button"
                    class="close"
                    data-dismiss="modal"
                    aria-label="Close"
                  >
                    <span aria-hidden="true">&times;</span>
                  </button>
                </div>
                <ul>
                  <li v-for="error in errors" class="text-danger">
                    {{ error }}
                  </li>
                </ul>
                <form v-on:submit.prevent="editUser()">
                  <div class="modal-body">
                    <div class="form-group">
                      <label>Username: </label>
                      <input type="text" v-model="user.username" />
                    </div>
                    <div class="form-group">
                      <label>Email: </label>
                      <input type="email" v-model="user.email" />
                    </div>
                  </div>
                  <div class="modal-footer">
                    <input
                      type="submit"
                      class="btn btn-primary"
                      value="Submit"
                    />
                  </div>
                </form>
              </div>
            </div>
          </div>
        </div>

        <div class="card p-4">
          <h3 class="card-title mb-2">
            Clothing Inventory
          </h3>
          <hr class="mt-0" />
          <div class="card">
            <h5 class="card-header">
              <select v-model="categoryFilter">
                <option value="">All Categories</option>
                <option v-for="category in categories" :value="category.name">{{
                  category.name
                }}</option>
              </select>
            </h5>
            <ul class="list-group">
              <div
                v-for="item in orderBy(
                  filterBy(items, categoryFilter, 'category_name'),
                  'name'
                )"
              >
                <li class="list-group-item" style="margin-right: 20px">
                  {{ item.name }}
                  <button
                    class="btn btn-primary"
                    data-toggle="modal"
                    data-target=".item-modal-lg"
                    v-on:click="currentItem = item"
                  >
                    Update
                  </button>
                  <div
                    class="modal fade item-modal-lg"
                    tabindex="-1"
                    role="dialog"
                    aria-labelledby="myLargeModalLabel"
                    aria-hidden="true"
                  >
                    <div class="modal-dialog modal-lg">
                      <div class="modal-content">
                        <div class="modal-header">
                          <button
                            type="button"
                            class="close"
                            data-dismiss="modal"
                            aria-label="Close"
                          >
                            <span aria-hidden="true">&times;</span>
                          </button>
                        </div>
                        <form v-on:submit.prevent="editItem(currentItem)">
                          <div class="form-group">
                            <div class="modal-body">
                              <label>Name: </label>
                              <input
                                type="text"
                                class="form-control"
                                v-model="currentItem.name"
                              />
                            </div>
                          </div>
                          <div class="modal-footer">
                            <input
                              type="submit"
                              class="btn btn-primary"
                              value="Submit"
                            />
                          </div>
                        </form>
                      </div>
                    </div>
                  </div>
                  <button
                    v-on:click="destroyItem(item)"
                    class="btn btn-primary"
                    style="margin-left: 20px"
                  >
                    Delete
                  </button>
                </li>
              </div>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
label {
  margin-right: 10px;
}
</style>

<script>
import axios from "axios";
import Vue2Filters from "vue2-filters";

export default {
  mixins: [Vue2Filters.mixin],
  data: function() {
    return {
      user: {},
      errors: [],
      items: [],
      categories: [],
      categoryFilter: "",
      currentItem: {},
    };
  },
  created: function() {
    axios.get("/api/users/me").then((response) => {
      console.log(response.data);
      this.user = response.data;
    });
    axios.get("/api/categories").then((response) => {
      console.log(response.data);
      this.categories = response.data;
    });
    axios.get("/api/items").then((response) => {
      console.log(response.data);
      this.items = response.data;
    });
  },
  methods: {
    editUser: function() {
      var params = {
        username: this.user.username,
        email: this.user.email,
      };
      axios
        .patch("/api/users/me", params)
        .then((response) => {
          console.log(response.data);
        })
        .catch((error) => {
          this.errors = error.response.data.errors;
        });
    },
    destroyUser: function() {
      if (confirm("Are you sure you want to delete your account?")) {
        axios.delete("/api/users/me").then((response) => {
          console.log(response.data);
          localStorage.removeItem("jwt");
          this.$router.push("/");
        });
      }
    },
    destroyItem: function(item) {
      if (confirm("Are you sure you want to delete this item?")) {
        axios.delete(`/api/items/${item.id}`).then((response) => {
          console.log(response.data);
          var index = this.items.indexOf(item);
          this.items.splice(index, 1);
        });
      }
    },
    editItem: function(item) {
      var params = {
        name: item.name,
      };
      axios
        .patch(`/api/items/${item.id}`, params)
        .then((response) => {
          console.log(response.data);
        })
        .catch((error) => {
          this.errors = error.response.data.errors;
        });
    },
  },
};
</script>
