
<template>
  <q-page padding>
    <q-card>
      <q-table
        title="JADWAL PELAJARAN"
        :data="data"
        :columns="columns"
        row-key="id"
        :pagination.sync="pagination"
        :loading="loading"
        :filter="filter"
        @request="onRequest"
        binary-state-sort
      >
      <template v-slot:body="props">
          <q-tr :props="props">
            <q-td key="kodepelajaran" :props="props">{{ props.row.kodepelajaran }}</q-td>
            <q-td key="namapelajaran" :props="props">{{ props.row.namapelajaran }}</q-td>
            <q-td key="jam" :props="props">{{ props.row.jam }}</q-td>
            <q-td key="hari" :props="props">{{ props.row.hari }}</q-td>
            <q-td key="ruangan" :props="props">{{ props.row.ruangan }}</q-td>
            <q-td key="nign" :props="props">{{ props.row.nign }}</q-td>
            <q-td key="namaguru" :props="props">{{ props.row.namaguru }}</q-td>
            <q-td key="aksi" :props="props">
              <q-card-actions align="around" class="row q-col-gutter-md no-wrap">
                <div class="col q-gutter-md">
                  <q-btn round @click="openDialog(props.row)" color="secondary" icon="edit"></q-btn>
                  <q-btn round @click="confirm(props.row._id)" color="negative" icon="delete"></q-btn>
                </div>
              </q-card-actions>
            </q-td>
          </q-tr>
        </template>

    </q-table>
    </q-card>
      <q-dialog v-model="openedit" v-if="openedit">
        <q-card style="width: 700px; max-width: 80vw;">
          <q-card-section class="row items-center">
            <div class="text-h6">Edit Data</div>
          </q-card-section>

          <q-separator/>

          <q-card-section style="max-height: 50vh" class="scroll">
            <q-input label="kodepelajaran" v-model="activedata.kodepelajaran"></q-input>
            <q-input label="namapelajaran" v-model="activedata.namapelajaran"></q-input>
            <q-input label="jam" v-model="activedata.jam"></q-input>
            <q-input label="hari" v-model="activedata.hari"></q-input>
            <q-input label="ruangan" v-model="activedata.ruangan"></q-input>
            <q-input label="nign" v-model="activedata.nign"></q-input>
            <q-input label="namaguru" v-model="activedata.namaguru"></q-input>
          </q-card-section>
          <q-card-actions align="right">
            <q-btn flat label="Cancel" color="primary" v-close-popup/>
            <q-btn flat label="Edit" color="primary" @click="edit()" />
          </q-card-actions>
        </q-card>
      </q-dialog>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data () {
    return {
      filter: '',
      loading: false,
      pagination: {
        sortBy: 'name',
        descending: false,
        page: 1,
        rowsPerPage: 3,
        rowsNumber: 10
      },
      columns: [
        {
          name: 'kodepelajaran',
          required: true,
          label: 'kodepelajaran',
          align: 'left',
          field: 'kodepelajaran',
          format: val => `${val}`,
          sortable: true
        },
        { name: 'namapelajaran', align: 'center', label: 'namapelajaran', field: 'namapelajaran', sortable: true },
        { name: 'jam', align: 'center', label: 'jam', field: 'jam', sortable: true },
        { name: 'hari', align: 'center', label: 'hari', field: 'hari', sortable: true },
        { name: 'ruangan', align: 'center', label: 'ruangan', field: 'ruangan', sortable: true },
        { name: 'nign', align: 'center', label: 'nign', field: 'nign', sortable: true },
        { name: 'namaguru', align: 'center', label: 'namaguru', field: 'namaguru', sortable: true },
        { name: 'aksi', align: 'center', label: 'Action', field: 'aksi' }
      ],
      data: [],
      original: [],
      openedit: false,
      activedata: null
    }
  },
  created () {
    this.getData()
  },
  mounted () {
    this.onRequest({
      pagination: this.pagination,
      filter: undefined
    })
  },
  methods: {
    getData () {
      this.$axios.get('/jadwal/getalljadwal')
        .then(res => {
          console.log(res)
          this.original = res.data.data
        })
    },
    showNotif (msg, color) {
      this.$q.notify({
        message: msg,
        color: color
      })
    },
    onRequest (props) {
      let { page, rowsPerPage, rowsNumber, sortBy, descending } = props.pagination
      let filter = props.filter

      this.loading = true

      // emulate server
      setTimeout(() => {
        // update rowsCount with appropriate value
        this.pagination.rowsNumber = this.getRowsNumberCount(filter)

        // get all rows if "All" (0) is selected
        let fetchCount = rowsPerPage === 0 ? rowsNumber : rowsPerPage

        // calculate starting row of data
        let startRow = (page - 1) * rowsPerPage

        // fetch data from "server"
        let returnedData = this.fetchFromServer(startRow, fetchCount, filter, sortBy, descending)

        // clear out existing data and add new
        this.data.splice(0, this.data.length, ...returnedData)

        // don't forget to update local pagination object
        this.pagination.page = page
        this.pagination.rowsPerPage = rowsPerPage
        this.pagination.sortBy = sortBy
        this.pagination.descending = descending

        // ...and turn of loading indicator
        this.loading = false
      }, 1500)
    },
    fetchFromServer (startRow, count, filter, sortBy, descending) {
      let data = []

      if (!filter) {
        data = this.original.slice(startRow, startRow + count)
      } else {
        let found = 0
        for (let index = startRow, items = 0; index < this.original.length && items < count; ++index) {
          let row = this.original[index]
          // match filter?
          if (!row['name'].includes(filter)) {
            // get a different row, until one is found
            continue
          }
          ++found
          if (found >= startRow) {
            data.push(row)
            ++items
          }
        }
      }

      // handle sortBy
      if (sortBy) {
        data.sort((a, b) => {
          let x = descending ? b : a
          let y = descending ? a : b
          if (sortBy === 'desc') {
            // string sort
            return x[sortBy] > y[sortBy] ? 1 : x[sortBy] < y[sortBy] ? -1 : 0
          } else {
            // numeric sort
            return parseFloat(x[sortBy]) - parseFloat(y[sortBy])
          }
        })
      }

      return data
    },
    getRowsNumberCount (filter) {
      if (!filter) {
        return this.original.length
      }
      let count = 0
      this.original.forEach((treat) => {
        if (treat['name'].includes(filter)) {
          ++count
        }
      })
      return count
    },
    openDialog (data) {
      this.openedit = true
      this.activedata = data
    },
    edit () {
      this.$axios.put('/jadwal/upjadwal/' + this.activedata._id, this.activedata)
        .then(res => {
          if (res.data.error) {
            this.showNotif(res.data.pesan, 'negative')
          } else {
            this.showNotif(res.data.pesan, 'positive')
            this.openedit = false
            this.getData()
          }
        })
    },
    confirm (id) {
      this.$q.dialog({
        title: 'Confirm',
        message: 'Are you sure ?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        this.$axios.delete('/jadwal/deletejadwal/' + id)
          .then(res => {
            if (res.data.error) {
              this.showNotif(res.data.pesan, 'negative')
            } else {
              this.showNotif(res.data.pesan, 'positive')
              this.getData()
            }
          })
      })
    }
  }
}
</script>
