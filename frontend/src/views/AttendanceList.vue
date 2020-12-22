<!-- eslint-disable -->
<template>
  <v-container id="attd-list" fluid tag="section">
    <v-row justify="center">
      <v-col 
        cols="12"
        md="12">
        <base-material-card
          icon="mdi-view-dashboard"
          :title="'Attendance List ' + todayDate()"
          class="px-5 py-3"
        >
          <div
            align="center"
            v-if="!finishLoading">
            <v-progress-circular
              indeterminate
              rotate
              color="success"
            >
            </v-progress-circular>
          </div>
          <v-text-field
            v-model="search"
            label="Search attendance record by employee's name"
          />
          <b-table
            show-empty
            v-if="finishLoading"
            striped 
            hover
            head-variant="dark"
            :fields="fields"
            :items="computedAttendance">
          </b-table>
        </base-material-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import axios from 'axios'
  import { BTable } from 'bootstrap-vue'

  export default {
    name: 'AttendanceList',
    components: {
      'b-table': BTable,
    },
    data () {
      return {
        search: '',
        listAttendance: [],
        isFirstTime: true,
        finishLoading: false,
        fields: [
          {
            key: 'nik',
            label: 'ID',
            sortable: false,
          },
          {
            key: 'nama',
            label: 'Name',
            sortable: false,
          },
          {
            key: 'timestamp',
            label: 'Time',
          },
        ],
      }
    },
    computed: {
      computedAttendance () {
        const searchQuery = this.search
        return this.listAttendance.filter(function (att) {
          return att.nama.toLowerCase().includes(searchQuery.toLowerCase())
        })
      },
    },
    mounted () {
      if (this.isFirstTime) {
        this.fetchData()
        this.isFirstTime = false
      }
      window.setInterval(() => {
        this.fetchData()
      }, 2000)
    },
    methods: {
      fetchData () {
        axios
          .get('http://localhost:5000/attendance')
          .then(response => {
            // console.log('fetching')
            this.listAttendance = response.data
            this.finishLoading = true
          })
      },
      todayDate () {
        var today = new Date()
        var dd = String(today.getDate()).padStart(2, '0')
        var mm = String(today.getMonth() + 1).padStart(2, '0')
        var yyyy = today.getFullYear()
        today = mm + '/' + dd + '/' + yyyy
        return today
      },
    },
  }
</script>

<style>
</style>
