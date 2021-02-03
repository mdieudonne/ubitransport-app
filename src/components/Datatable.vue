<template>
  <v-data-table
      :headers="headers"
      :items="students"
      sort-by="calories"
      class="elevation-1"
      @update:page="onChangePage"
      @update:items-per-page="onChangeItemsPerPage"
  >
    <template v-slot:top>
      <v-toolbar
          flat
      >
        <v-toolbar-title>Gestion des élèves</v-toolbar-title>
        <v-divider
            class="mx-4"
            inset
            vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog
            v-model="dialog"
            max-width="500px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
                color="primary"
                dark
                class="mb-2"
                v-bind="attrs"
                v-on="on"
            >
              New Item
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-form v-model="valid">
              <v-container>
                <v-row>
                  <v-col
                      cols="12"
                  >
                    <v-text-field
                        v-model="editedItem.lastname"
                        label="Nom"
                    ></v-text-field>
                  </v-col>
                  <v-col
                      cols="12"
                  >
                    <v-text-field
                        v-model="editedItem.firstname"
                        label="Prénom"
                    ></v-text-field>
                  </v-col>
                  <v-col
                      cols="12"
                  >
                    <v-menu
                        v-model="menu"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                            v-model="editedItem.birthdate"
                            label="Picker without buttons"
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                          v-model="editedItem.birthdate"
                          @input="menu = false"
                          :max="getMax"
                          :min="getMin"
                      ></v-date-picker>
                    </v-menu>
                  </v-col>
                </v-row>
              </v-container>
              </v-form>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                  color="blue darken-1"
                  text
                  @click="close"
              >
                Cancel
              </v-btn>
              <v-btn
                  color="blue darken-1"
                  text
                  @click="save"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="headline">Are you sure you want to delete this item?</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
          small
          @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
  </v-data-table>
</template>

<script>
import axios from "axios";
import {addYears, format} from 'date-fns';

export default {
  name: "Datatable",
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      {
        text: 'Nom',
        align: 'start',
        value: 'lastname',
      },
      {text: 'Prénom', value: 'firstname'},
      {text: 'Date de naissance', value: 'birthdate'},
    ],
    students: [],
    editedIndex: -1,
    editedItem: {
      lastname: '',
      firstname: '',
      birthdate: '',
    },
    defaultItem: {
      lastname: '',
      firstname: '',
      birthdate: '',
    },
    itemsPerPage: 20,
    page: 1,
    menu: false,
    valid: false,
    nameRules: [
      v => !!v || 'Une valeur est requise',
      v => v.length <= 10 || 'La valeur ne peut être supérieure à 50 caractères',
    ],
    todayDate: new Date()
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Création' : 'Edition'
    },
    getMin() {
      return format(addYears(this.todayDate, -120), 'yyyy-MM-dd')
    },
    getMax() {
      return format(this.todayDate, 'yyyy-MM-dd')
    }
  },

  watch: {
    dialog(val) {
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },
  },

  created() {
    this.executeQuery()
  },

  methods: {
    async executeQuery() {
      try {
        const response = await axios({
          method: 'GET',
          url: 'students',
          params: {
            itemsPerPage: this.itemsPerPage,
            page: this.page
          }
        })
        this.students = response.data.students
        console.log(this.students)
      } catch (err) {
        console.log(err.message)
      }
    },

    editItem(item) {
      this.editedIndex = this.students.indexOf(item)
      this.editedItem = {...item}
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.students.indexOf(item)
      this.editedItem = {...item}
      this.dialogDelete = true
    },

    deleteItemConfirm() {
      this.students.splice(this.editedIndex, 1)
      this.closeDelete()
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = {...this.defaultItem}
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = {...this.defaultItem}
        this.editedIndex = -1
      })
    },

    async save() {
      // if (this.editedIndex > -1) {
      //   this.students[this.editedIndex] = {...this.editedItem}
      // } else {
      //   this.students.push(this.editedItem)
      // }

      try {
        const response = await axios({
          method: 'POST',
          url: 'students',
          data: {...this.editedItem}
        })
        console.log(response.data)
      } catch (err) {
        console.log(err.message)
      }
      this.close()
    },

    onChangePage(val) {
      this.page = val
      this.executeQuery()
    },

    onChangeItemsPerPage(val) {
      this.itemsPerPage = val
      this.executeQuery()
    }
  }
}
</script>

<style scoped>

</style>
