<template>
  <div>
    <v-alert v-if="message" border="left" dense type="warning">
      {{ message }}
    </v-alert>
    <student-datatable :average-score="averageScore" @alert:change="onAlert" @student:change="onSelectStudent"
                       @student:delete="updateAverageScore"/>
    <score-datatable :average-student-score="averageStudentScore" :student="student" @alert:change="onAlert"
                     @score:update="updateAverageScore"/>
  </div>
</template>

<script>
import StudentDatatable from "../components/StudentDatatable";
import ScoreDatatable from "../components/ScoreDatatable";
import axios from "axios";

export default {
  name: "Home",
  data: () => ({
    message: '',
    student: '',
    averageScore: 0,
    averageStudentScore: 0,
  }),
  components: {
    StudentDatatable,
    ScoreDatatable,
  },
  created() {
    this.updateAverageScore()
  },
  methods: {
    onAlert(val) {
      this.message = val
    },
    async onSelectStudent(obj) {
      this.student = obj
      await this.updateAverageStudentScore()
    },
    async updateAverageScore() {
      try {
        this.message = ''
        const response = await axios({
          method: 'GET',
          url: 'scores/getAverage',
          params: {},
        })
        this.averageScore = response.data
        if ('id' in this.student) {
          this.updateAverageStudentScore()
        }
      } catch (err) {
        if (err.response) {
          this.message = '' + err.response.status + ': ' + err.response.data.detail
        }
      }
    },
    async updateAverageStudentScore() {
      try {
        this.message = ''
        const response = await axios({
          method: 'GET',
          url: 'student/' + this.student.id + '/getAverage',
          params: {},
        })
        this.averageStudentScore = response.data
      } catch (err) {
        if (err.response) {
          this.message = '' + err.response.status + ': ' + err.response.data.detail
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
