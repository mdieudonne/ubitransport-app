<template>
  <div>
    <v-alert v-if="message" dense border="left" type="warning">
      {{message}}
    </v-alert>
    <datatable @alert:change="onAlert"/>
  </div>
</template>

<script>
import Datatable from "../components/Datatable";

export default {
  name: "Home",
  data: () => ({
    message: ''
  }),
  components: {
    Datatable
  },
  methods:{
    onAlert(val) {
      this.message = val
    }
  }
}
</script>

<style scoped>

</style>
