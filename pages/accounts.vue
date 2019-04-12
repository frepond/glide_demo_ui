<template>
  <v-layout>
    <v-flex text-xs-center>
      <template>
        <div>
          <v-toolbar flat color="white">
            <v-toolbar-title>Accounts CRUD</v-toolbar-title>
            <v-divider
              class="mx-2"
              inset
              vertical
            ></v-divider>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on }">
                <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
              </template>
              <v-card>
                <v-card-title>
                  <span class="headline">{{ formTitle }}</span>
                </v-card-title>

                <v-card-text>
                  <v-container grid-list-md>
                    <v-layout wrap>
                      <v-flex xs12 sm12 md12>
                        <v-text-field v-model="editedItem.email" label="E-mail"
                          :rules="[rules.required, rules.email]"></v-text-field>
                      </v-flex>
                    </v-layout>
                  </v-container>
                </v-card-text>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
                  <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>
          </v-toolbar>
          <v-data-table
            :headers="headers"
            :items="accounts"
            class="elevation-1"
          >
            <template v-slot:items="props">
              <td class="text-xs-left">{{ props.item.id }}</td>
              <td class="text-xs-left">{{ props.item.email }}</td>
              <td class="justify-center layout px-0">
                <v-icon
                  small
                  class="mr-2"
                  @click="editItem(props.item)"
                >
                  edit
                </v-icon>
                <v-icon
                  small
                  @click="deleteItem(props.item)"
                >
                  delete
                </v-icon>
              </td>
            </template>
            <template v-slot:no-data>
              <v-btn color="primary" @click="initialize">Reset</v-btn>
            </template>
          </v-data-table>
        </div>
      </template>
      <blockquote class="blockquote">
        &#8220;First, solve the problem. Then, write the code.&#8221;
        <footer>
          <small>
            <em>&mdash;John Johnson</em>
          </small>
        </footer>
      </blockquote>
    </v-flex>
  </v-layout>
</template>
<script>
  import axios from "axios";
  export default {
    data: () => ({
      dialog: false,
      headers: [
        {
          text: 'Id',
          align: 'left',
          sortable: false,
          value: 'id'
        },
        { text: 'E-mail', value: 'email', align: 'left' },
      ],
      api_base: "http://localhost:5000/api/v1",
      accounts: [],
      editedIndex: -1,
      editedItem: {
        email: ''
      },
      defaultItem: {
        email: ''
      },
      rules: {
        required: value => !!value || 'Required.',
        email: value => {
          const pattern = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
          return pattern.test(value) || 'Invalid e-mail.'
        }
      }
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      }
    },

    watch: {
      dialog (val) {
        val || this.close()
      }
    },

    created () {
      this.initialize()
    },

    methods: {
      initialize () {
        let self = this
        axios.get(this.api_base + '/accounts')
        .then(function (response) {
          // handle success
          self.accounts = response.data
        })
        .catch(e => {
          if (e.response)
            console.log(e.response.data)
          else
            console.log(e)
        })
      },

      editItem (item) {
        this.editedIndex = this.accounts.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        const index = this.accounts.indexOf(item)
        let self = this
        confirm('Are you sure you want to delete this item?') &&
          axios.delete(this.api_base + '/accounts/' + item.id)
          .then(response => {
            self.accounts.splice(index, 1)
          })
          .catch(e => {
            if (e.response)
              alert(e.response.data.message)
            else
              console.log(e)
          })
      },

      close () {
        this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 300)
      },

      save () {
        let self = this
        if (this.editedIndex > -1) {
          axios.put(this.api_base + '/accounts/' + this.editedItem.id, this.editedItem)
          .then(function(response) {
            Object.assign(self.accounts[self.editedIndex], self.editedItem)
          })
          .catch(e => {
            if (e.response)
              alert(e.response.data.message)
            else
              console.log(e)
          })
        } else {
          axios.post(this.api_base + '/accounts', this.editedItem)
          .then(function(response) {
            let uri = response.headers.location
            return axios.get(uri)
          })
          .then(function(response) {
            self.accounts.push(response.data)
          })
          .catch(e => {
            if (e.response)
              alert(e.response.data.message)
            else
              console.log(e)
          })
        }
        this.close()
      }
    }
  }
</script>
