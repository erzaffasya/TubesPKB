<!-- eslint-disable -->
<template>
  <v-container id="attd-list" fluid tag="section">
    <v-row justify="center">
      <v-col 
        cols="12"
        md="12">
        <base-material-card
          icon="mdi-account-multiple"
          title="Employee List"
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
          <div>
            <v-text-field
              v-model="search"
              label="Search employee by name"
            />
            <b-table
              id="table-employee"
              v-if="finishLoading"
              :per-page="perPage"
              :current-page="currentPage"
              striped 
              show-empty
              hover
              head-variant="dark"
              :fields="fields"
              :items="computedEmployee">
              <template v-slot:cell(action)="row">
                <v-btn v-if="row.item.active == 1" color="red" min-width=200 @click="willDeactivate(row.item.nik, row.item.nama)" class="mr-1">
                  deactivate
                </v-btn>
                <v-btn v-if="row.item.active == 0" color="grey" disabled min-width=200 class="mr-1">
                  deactivated
                </v-btn>
              </template>
            </b-table>
            <b-pagination
              v-model="currentPage"
              :total-rows="rows"
              :per-page="perPage"
              aria-controls="table-employee"
              pills
            ></b-pagination>
          </div>
          <b-modal centered ref="remove-confirmation" title="Confirmation" @ok="deactivate">
            <p class="my-4">Are you sure you want to remove "{{ employee_will_be_removed.nama }}" with ID number "{{ employee_will_be_removed.nik }}?"</p>
          </b-modal>
          <div align="right" class="mb-1">
            <b-form-checkbox
              v-model="showDeactivated"
            >
              show deactivated employees 
            </b-form-checkbox>
          </div>
        </base-material-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  import axios from 'axios'
  import { BTable, BModal, BFormCheckbox, BPagination } from 'bootstrap-vue'

  export default {
    name: 'ListEmployee',
    components: {
      'b-table': BTable,
      'b-modal': BModal,
      'b-form-checkbox': BFormCheckbox,
      'b-pagination': BPagination,
    },
    data () {
      return {
        search: '',
        perPage: 5,
        currentPage: 1,
        employee: [],
        isFirstTime: true,
        finishLoading: false,
        showDeactivated: false,
        employee_will_be_removed: {
          nik: '',
          nama: '',
        },
        fields: [
          {
            key: 'nik',
            label: 'ID',
          },
          {
            key: 'nama',
            label: 'Name',
            sortable: true,
            thStyle: { minWidth: 1000 },
          },
          {
            key: 'created_at',
            label: 'Created at',
            sortable: true,
          },
          {
            key: 'action',
            label: 'Action',
          },
        ],
      }
    },
    computed: {
      computedEmployee () {
        const searchQuery = this.search
        if (!this.showDeactivated) {
          return this.employee.filter(function (emp) {
            const containsSearchQuery = emp.nama.toLowerCase().includes(searchQuery.toLowerCase())
            return (emp.active === 1 && containsSearchQuery)
          })
        } return this.employee
      },
      rows () {
        return this.computedEmployee.length
      },
    },
    mounted () {
      this.fetchData()
    },
    methods: {
      fetchData () {
        axios
          .get('http://localhost:5000/employee')
          .then(response => {
            // console.log(response.data)
            this.employee = response.data
            this.finishLoading = true
          })
      },
      willDeactivate (nik, nama) {
        this.employee_will_be_removed = {
          nik: nik,
          nama: nama,
        }
        this.$refs['remove-confirmation'].show()
      },
      deactivate () {
        axios
          .get('http://localhost:5000/remove_employee/' + this.employee_will_be_removed.nik)
          .then(() => location.reload())
      },
    },
  }
</script>

<style>
</style>
