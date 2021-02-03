<template>
  <div>
    <v-alert v-if="message" dense border="left" type="warning">
      {{message}}
    </v-alert>
    <student-datatable @alert:change="onAlert" @student:change="onSelectStudent"/>
    <score-datatable @alert:change="onAlert" :student="student"/>
  </div>
</template>

<script>
import StudentDatatable from "../components/StudentDatatable";
import ScoreDatatable from "../components/ScoreDatatable";

export default {
  name: "Home",
  data: () => ({
    message: '',
    student: '',
  }),
  components: {
    StudentDatatable,
    ScoreDatatable,
  },
  methods:{
    onAlert(val) {
      this.message = val
    },
    onSelectStudent(obj) {
      this.student = obj
    }
  }
}
</script>

<style scoped>

</style>
