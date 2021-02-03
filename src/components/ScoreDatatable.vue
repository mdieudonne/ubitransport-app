<template>
  <v-data-table
      v-if="student"
      :headers="headers"
      :items="scores"
      :server-items-length="totalItems"
      sort-by="calories"
      class="elevation-1"
      @update:page="onChangePage"
      @update:items-per-page="onChangeItemsPerPage"
  >
    <template v-slot:top>
      <v-toolbar
          flat
      >
        <v-toolbar-title>Gestion des note de {{ student.lastname }} {{ student.firstname }}</v-toolbar-title>
        <v-divider
            class="mx-4"
            inset
            vertical
        ></v-divider>
        <span>Moyenne : {{ averageStudentScore }}</span>
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
              Nouvelle note
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
                    <v-col cols="12">
                      <v-text-field
                          v-model.number="editedItem.subject"
                          label="Matière"
                      ></v-text-field>
                      <v-text-field
                          type="number"
                          v-model.number="editedItem.value"
                          label="Note"
                      ></v-text-field>
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
  props: {
    student: [Object, String],
    averageStudentScore: Number,
  },
  data: () => ({
    dialog: false,
    dialogDelete: false,
    headers: [
      {text: 'Matière', align: 'start', value: 'subject',},
      {text: 'Note', align: 'start', value: 'value',},
      {text: 'Actions', value: 'actions', sortable: false},
    ],
    scores: [],
    editedIndex: -1,
    editedItem: {
      subject: '',
      value: '',
    },
    defaultItem: {
      subject: '',
      value: '',
    },
    totalItems: 0,
    itemsPerPage: 5,
    page: 1,
    valid: false,
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Création' : 'Edition'
    },
  },

  watch: {
    student(val) {
      this.executeQuery()
    },
    dialog(val) {
      val || this.close()
    },
    dialogDelete(val) {
      val || this.closeDelete()
    },

  },

  methods: {
    async executeQuery() {
      try {
        this.clearError()
        const response = await axios({
          method: 'GET',
          url: 'scores',
          params: {
            itemsPerPage: this.itemsPerPage,
            page: this.page,
            idStudent: this.student.id,
          }
        })
        this.scores = response.data.scores
        this.totalItems = response.data.totalItems
      } catch (err) {
        this.scores = []
        this.onError(err)
      }
    },

    editItem(item) {
      this.editedIndex = this.scores.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },

    deleteItem(item) {
      this.editedIndex = this.scores.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialogDelete = true
    },

    async deleteItemConfirm() {
      this.clearError()
      await axios({
        method: 'DELETE',
        url: 'scores/' + this.editedItem.id,
      })
      this.scores.splice(this.editedIndex, 1)
      this.$emit('score:update')
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
            url: 'scores/' + this.editedItem.id,
            data: {...this.editedItem}
          })
          response.data.birthdate = this.formatDate(response.data.birthdate),
              Object.assign(this.scores[this.editedIndex], response.data)
          this.$emit('score:update')
        } else {
          const response = await axios({
            method: 'POST',
            url: 'scores',
            data: {...this.editedItem, idStudent: this.student.id,}
          })
          this.scores.push({
            ...response.data,
            birthdate: this.formatDate(response.data.birthdate),
          })
          this.$emit('score:update')
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

    formatDate(date) {
      return date ? format(date instanceof Date ? date : parseISO(date), 'yyyy-MM-dd') : null
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
