<template>
  <div class="container mt-5">
    <b-row class="justify-content-center">
      <b-form inline>
        <b-input
          v-model="newItem"
          class="mb-2 mr-2"
          placeholder="New Item"
          @keyup.enter.prevent="addShoppingItem"
        ></b-input>
        <b-button
          class="mb-2"
          variant="primary"
          @click.prevent="addShoppingItem"
          >Add
        </b-button>
      </b-form>
    </b-row>

    <b-row class="mt-4 justify-content-center">
      <b-col md="6" sm="12">
        <b-list-group>
          <b-list-group-item v-for="(item, index) in listItems" :key="index">
            <b-row class="flex-row">
              <b-col class="d-flex justify-content-around align-items-center">
                {{ item.itemName }}
                <b-button
                  size="sm"
                  variant="danger"
                  @click="deleteShoppingItem(item.id)"
                >
                  Delete</b-button
                >
              </b-col>
            </b-row>
          </b-list-group-item>
        </b-list-group>
      </b-col>
    </b-row>
  </div>
</template>

<script>
// @ is an alias to /src
import { Auth, API, graphqlOperation } from "aws-amplify";

import * as queries from "../graphql/queries";
import * as mutations from "../graphql/mutations";
import * as subscriptions from "../graphql/subscriptions";

export default {
  name: "Home",
  data() {
    return {
      listItems: [],
      newItem: ""
    };
  },
  async mounted() {
    const currentUser = await Auth.currentAuthenticatedUser();

    const that = this;

    API.graphql(
      graphqlOperation(subscriptions.onCreateShoppingListItem, {
        owner: currentUser.username
      })
    ).subscribe({
      next: action => {
        that.listItems.push(action.value.data.onCreateShoppingListItem);
      }
    });

    API.graphql(
      graphqlOperation(subscriptions.onDeleteShoppingListItem, {
        owner: currentUser.username
      })
    ).subscribe({
      next: action => {
        const id = action.value.data.onDeleteShoppingListItem.id;
        const index = that.listItems.map(x => x.id).indexOf(id);
        that.listItems.splice(index, 1);
      }
    });
  },
  async created() {
    const { data } = await API.graphql(
      graphqlOperation(queries.listShoppingListItems)
    );

    data.listShoppingListItems.items.forEach(item => {
      this.listItems.push(item);
    });
  },
  methods: {
    async addShoppingItem() {
      await API.graphql(graphqlOperation(mutations.createShoppingListItem, {
        input: { itemName: this.newItem }
      }));

      this.newItem = "";
    },
    async deleteShoppingItem(id) {
       await API.graphql(graphqlOperation(mutations.deleteShoppingListItem, {
        input: { id: id }
      }));

      }
  }
};
</script>
