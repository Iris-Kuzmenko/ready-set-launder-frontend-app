<template>
  <div class="items-launder">
    <div id="content">
      <div class="container">
        <div class="card p-4">
          <h3 class="card-title mb-2">
            Launder: Track the Process!
          </h3>
          <hr class="my-3" />
          <div class="row">
            <div class="col-lg-4">
              <div class="card p-4">
                <h4 class="card-title mb-2">
                  To Wash
                </h4>
                <hr class="my-3" />
                <form v-on:submit.prevent="updateStatusToDry()">
                  <div v-for="washSetting in washSettings">
                    <div
                      v-if="
                        filterBy(
                          itemsWash,
                          washSetting.name,
                          'wash_setting_name'
                        ).length
                      "
                    >
                      <div class="card">
                        <h5 class="card-header">
                          <input
                            type="checkbox"
                            :id="washSetting.id"
                            :value="washSetting.id"
                            v-model="washSettingIds"
                          />
                          <label :for="washSetting.name">{{
                            washSetting.name
                          }}</label>
                        </h5>
                        <ul class="list-group list-group-flush">
                          <div
                            v-for="item in filterBy(
                              itemsWash,
                              washSetting.name,
                              'wash_setting_name'
                            )"
                          >
                            <li class="list-group-item">
                              {{ item.name }}
                            </li>
                          </div>
                        </ul>
                      </div>
                    </div>
                  </div>
                  <input type="submit" class="btn btn-primary" value="Done" />
                </form>
              </div>
            </div>
            <div class="col-lg-4">
              <div class="card p-4">
                <h4 class="card-title mb-2">
                  To Dry
                </h4>
                <hr class="my-3" />
                <form v-on:submit.prevent="updateStatusToFold()">
                  <div v-for="drySetting in drySettings">
                    <div
                      v-if="
                        filterBy(itemsDry, drySetting.name, 'dry_setting_name')
                          .length
                      "
                    >
                      <div class="card">
                        <h5 class="card-header">
                          <input
                            type="checkbox"
                            :id="drySetting.id"
                            :value="drySetting.id"
                            v-model="drySettingIds"
                          />
                          <label :for="drySetting.name">{{
                            drySetting.name
                          }}</label>
                        </h5>
                        <ul class="list-group list-group-flush">
                          <div
                            v-for="item in filterBy(
                              itemsDry,
                              drySetting.name,
                              'dry_setting_name'
                            )"
                          >
                            <li class="list-group-item">
                              {{ item.name }}
                            </li>
                          </div>
                        </ul>
                      </div>
                    </div>
                  </div>
                  <input type="submit" class="btn btn-primary" value="Done" />
                </form>
              </div>
            </div>
            <div class="col-lg-4">
              <div class="card p-4">
                <h4 class="card-title mb-2">
                  To Fold
                </h4>
                <hr class="my-3" />
                <form v-on:submit.prevent="updateStatusToCreated()">
                  <div v-for="category in categories">
                    <div
                      v-if="
                        filterBy(itemsFold, category.name, 'category_name')
                          .length
                      "
                    >
                      <div class="card">
                        <h5 class="card-header">
                          <input
                            type="checkbox"
                            :id="category.id"
                            :value="category.id"
                            v-model="categoryIds"
                          />
                          <label :for="category.name">{{
                            category.name
                          }}</label>
                        </h5>
                        <ul class="list-group">
                          <div
                            v-for="item in filterBy(
                              itemsFold,
                              category.name,
                              'category_name'
                            )"
                          >
                            <li class="list-group-item">
                              {{ item.name }}
                            </li>
                          </div>
                        </ul>
                      </div>
                    </div>
                  </div>
                  <input type="submit" class="btn btn-primary" value="Done" />
                </form>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
h4 {
  color: #55a79a;
}
</style>

<script>
import axios from "axios";
import Vue2Filters from "vue2-filters";

export default {
  mixins: [Vue2Filters.mixin],
  data: function() {
    return {
      items: [],
      washSettings: [],
      drySettings: [],
      categories: [],
      itemsWash: [],
      itemsDry: [],
      itemsFold: [],
      itemIdsForDry: [],
      itemIdsToFold: [],
      itemIdsDone: [],
      washSettingIds: [],
      drySettingIds: [],
      categoryIds: [],
    };
  },
  created: function() {
    axios.get("/api/items").then((response) => {
      this.itemsWash = this.filterBy(response.data, "wash", "status");
      console.log("Items Wash List", this.itemsWash);
      this.itemsDry = this.filterBy(response.data, "dry", "status");
      console.log("Items Dry List", this.itemsDry);
      this.itemsFold = this.filterBy(response.data, "fold", "status");
      console.log("Items Fold List", this.itemsFold);
    });
    axios.get("/api/wash_settings").then((response) => {
      this.washSettings = response.data;
      console.log("Wash Settings", this.washSettings);
    });
    axios.get("/api/dry_settings").then((response) => {
      this.drySettings = response.data;
      console.log("Dry Settings", this.drySettings);
    });
    axios.get("/api/categories").then((response) => {
      this.categories = response.data;
      console.log("Categories", this.categories);
    });
  },
  methods: {
    updateStatusToDry: function() {
      this.washSettingIds.forEach((washSettingId) => {
        var items = this.itemsWash.filter(
          (item) => item.wash_setting_id === washSettingId
        );
        console.log(items);
        items.forEach((item) => {
          this.itemIdsForDry.push(item.id);
        });
      });
      console.log(this.itemIdsForDry);
      var params = {
        item_ids: this.itemIdsForDry,
        status: "dry",
      };
      axios.patch("/api/items_status", params).then((response) => {
        this.itemIdsForDry.forEach((itemId) => {
          var item = this.itemsWash.find((item) => item.id === itemId);
          item.status = "dry";
          var index = this.itemsWash.indexOf(item);
          this.itemsWash.splice(index, 1);
          this.itemsDry.push(item);
          this.itemIdsForDry = [];
        });
      });
      console.log(this.itemIdsForDry);
    },
    updateStatusToFold: function() {
      this.drySettingIds.forEach((drySettingId) => {
        var items = this.itemsDry.filter(
          (item) => item.dry_setting_id === drySettingId
        );
        console.log(items);
        items.forEach((item) => {
          this.itemIdsToFold.push(item.id);
        });
      });
      console.log(this.itemIdsToFold);
      var params = {
        item_ids: this.itemIdsToFold,
        status: "fold",
      };
      axios.patch("/api/items_status", params).then((response) => {
        this.itemIdsToFold.forEach((itemId) => {
          var item = this.itemsDry.find((item) => item.id === itemId);
          var index = this.itemsDry.indexOf(item);
          this.itemsDry.splice(index, 1);
          this.itemsFold.push(item);
          this.itemIdsToFold = [];
        });
      });
    },
    updateStatusToCreated: function() {
      this.categoryIds.forEach((categoryId) => {
        var items = this.itemsFold.filter(
          (item) => item.category_id === categoryId
        );
        console.log(items);
        items.forEach((item) => {
          this.itemIdsDone.push(item.id);
        });
      });
      console.log(this.itemIdsDone);
      var params = {
        item_ids: this.itemIdsDone,
        status: "created",
      };
      axios.patch("/api/items_status", params).then((response) => {
        this.itemIdsDone.forEach((itemId) => {
          var item = this.itemsFold.find((item) => item.id === itemId);
          var index = this.itemsFold.indexOf(item);
          this.itemsFold.splice(index, 1);
          this.itemIdsDone = [];
        });
      });
    },
  },
};
</script>
