<!-- eslint-disable -->
<template>
  <v-container id="attd-list" fluid tag="section">
    <v-row justify="center">
      <v-col 
        cols="12"
        md="12">
        <base-material-card
          icon="mdi-account-plus"
          title="Create Attendance Report"
          class="px-5 py-3"
        > 
          <v-form>
            <v-container class="py-0">
                <v-row justify="center">
                    <v-date-picker
                        v-model="picker"
                        full-width
                    >
                    </v-date-picker>
                </v-row>
                <v-row>
                <v-col cols="12" md="6">
                    <v-btn
                    block
                    color="success"
                    class="mr-0"
                    @click="check">
                    process report
                    </v-btn>
                </v-col>
                <v-col cols="12" md="6">
                  <v-btn
                    block
                    color="blue"
                    :disabled="!canDownload()"
                    class="mr-0"
                    @click="check">

                    <download-excel
                      :data   = "datalist.list"
                      :fields = "fields"
                      worksheet = "My Worksheet"
                      :name    = "'Attendance List ' + this.picker + '.xls'">
                    </download-excel>

                  </v-btn>
                </v-col>
                </v-row>

            </v-container>
          </v-form>
        </base-material-card>
      </v-col>

    </v-row>
  </v-container>
</template>

<script>
  import axios from 'axios'
  import JsonExcel from 'vue-json-excel'
  export default {
    name: 'CreateReport',
    components: {
      'download-excel': JsonExcel,
    },
    data () {
      return {
        picker: null,
        datalist: {
          date: '',
          list: [],
        },
        fields: {
          'ID Karyawan': 'id_karyawan',
          'Nama Karyawan': 'nama',
          'Jam Datang': 'datang',
          'Jam Pulang': 'pulang',
        },
      }
    },
    computed: {
    },
    mounted () {
      this.picker = this.getDate()
    },
    methods: {
      canDownload () {
        return this.picker === this.datalist.date && this.datalist.list.length > 0
      },
      check (e) {
        e.preventDefault()
        axios
          .get('http://localhost:5000/report/' + this.picker)
          .then(response => {
            if (response.data.length === 0) {
              alert('Tidak ada catatan kehadiran pada tanggal ' + this.picker)
            } else {
              this.datalist.date = this.picker
              this.datalist.list = response.data
            }
          })
      },
      getDate () {
        const today = new Date()
        var dd = String(today.getDate()).padStart(2, '0')
        var mm = String(today.getMonth() + 1).padStart(2, '0')
        var yyyy = today.getFullYear()
        return yyyy + '-' + mm + '-' + dd
      },
    },
  }
</script>

<style>
</style>
