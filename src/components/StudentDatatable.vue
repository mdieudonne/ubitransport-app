<template>
  <v-data-table
      :headers="headers"
      :items="students"
      :server-items-length="totalItems"
      sort-by="calories"
      class="elevation-1"
      show-select
      single-select
      @update:page="onChangePage"
      @update:items-per-page="onChangeItemsPerPage"
      @item-selected="onSelect"
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
              Nouvel élève
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-form ref="form" v-model="valid" lazy-validation>
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
                            label="Date de naissance"
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                            clearable
                            @click:clear="editedItem.birthdate = null"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                          v-model="editedItem.birthdate"
                          @input="menu = false"
                          show-week
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
import {addYears, format, parseISO} from 'date-fns';

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
      {text: 'Actions', value: 'actions', sortable: false},
    ],
    students: [],
    editedIndex: -1,
    todayDate: new Date(),
    editedItem: {
      lastname: '',
      firstname: '',
      birthdate: null,
    },
    defaultItem: {
      lastname: '',
      firstname: '',
      birthdate: null,
    },
    totalItems: 0,
    itemsPerPage: 5,
    page: 1,
    menu: false,
    valid: false,
    nameRules: [
      v => !!v || 'Une valeur est requise',
      v => v.length <= 10 || 'La valeur ne peut être supérieure à 50 caractères',
    ],
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
        this.clearError()
        const response = await axios({
          method: 'GET',
          url: 'students',
          params: {
            itemsPerPage: this.itemsPerPage,
            page: this.page
          }
        })
        this.students = response.data.students.map(student => ({
          ...student,
          birthdate: this.formatDate(student.birthdate),
        }))
        this.totalItems = response.data.totalItems
      } catch (err) {
        this.onError(err)
      }
    },

    editItem(item) {
      this.editedIndex = this.students.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.students.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    async deleteItemConfirm() {
      await axios({
        method: 'DELETE',
        url: 'students/' + this.editedItem.id,
      })
      this.students.splice(this.editedIndex, 1)
      this.closeDelete()
    },

    close() {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    closeDelete() {
      this.dialogDelete = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },

    async save() {
      try {
        this.clearError()
        if (this.editedIndex > -1) {
          const response = await axios({
            method: 'PUT',
            url: 'students/' + this.editedItem.id,
            data: {...this.editedItem}
          })
          response.data.birthdate = this.formatDate(response.data.birthdate),
          Object.assign(this.students[this.editedIndex], response.data)
        } else {
          const response = await axios({
            method: 'POST',
            url: 'students',
            data: {...this.editedItem}
          })
          this.students.push({
            ...response.data,
            birthdate:this.formatDate(response.data.birthdate),
          })
        }
      } catch (err) {
        this.onError(err)
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
    },

    onSelect(val) {
      this.$emit('student:change', val.item)
    },

    formatDate(date) {
      return date ? format(date instanceof Date ? date : parseISO(date),'yyyy-MM-dd') : null
    },

    onError(err) {
      if (err.response) {
        console.log(err.response.data);
        console.log(err.response.status);
        console.log(err.response.headers);
        this.$emit(
            'alert:change',
            '' + err.response.status + ': ' + err.response.data.detail,
        )
      }
    },

    clearError() {
      this.$emit('alert:change', '',)
    }

  }
}
</script>

<style scoped>

</style>
