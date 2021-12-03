<template lang="pug">
  v-container
    v-row
      v-col(cols)
        DataTable(
          v-if="items.length"
          :headers="headers"
          :items="items"
          :totalItems="totalItems"
          :loading="loading"
          :options="options",
          :currentPage="currentPage",
          :pageSize="pageSize"
          :dropdowns="dropdowns"
          @fetch="fetch"
        )
        v-progress-circular(
          v-else
          width="2"
          color="rs__primary"
          indeterminate
        ).mx-auto


</template>

<script>
import DataTable from '~/components/DataTable.vue'
import sales from '~/api/sales.js'

export default {
  components: {
    DataTable
  },
  data() {
    return {
      sales,
      items: [],
      totalItems: 0,
      dropdowns: {},
      pageSize: 10,
      currentPage: 1,
      fakeServerDelay: 10, //ms
      loading: false,
      options: {},
      headers: [
        {text: '#', value: 'id', align: 'start'},
        {text: 'Name', value: 'fullname', visible: true},
        {text: 'Email', value: 'email', visible: true},
        {text: 'Gender', value: 'gender'},
        {text: 'IP Address', value: 'ip_address'},
        {text: 'Year', value: 'year', visible: true},
        {text: 'Sales', value: 'sales', visible: true},
        {text: 'Country', value: 'country', visible: true},
        {text: 'Currency', value: 'currency'},
        {text: 'Color', value: 'color'},
      ],
    }
  },
  async created() {
    this.fetch()
  },

  methods: {
    prepareDropdownData(dataType){
      return [{
        name: '-- Show All --',
        value: ''
      }, ...new Set(sales.results
        .map(item => {
          return {
            name: item[dataType],
            value: item[dataType]
          }
      }))].sort()
    },
    async fetchData(options) {
      const {page, itemsPerPage, dropdowns, filter} = options;
      const start = (page - 1) * itemsPerPage
      await this.delay(this.fakeServerDelay)

      let data = sales.results;

      if(filter) {
        data = sales.results.filter(item => {
          if(!filter) return true;

          let allFiltersSatisfied = true;
          Object.keys(filter).forEach(fieldName => {

            //If no filter set, skip
            if(!filter[fieldName] || !filter[fieldName].value) return false;

            if(filter[fieldName].value !== item[fieldName]){
              allFiltersSatisfied = false;
            }
          });
          return allFiltersSatisfied;
        })
      }

      let total = data.length;

      data = itemsPerPage !== -1 ? data.slice(start, start + itemsPerPage) : data

      return {
        items: data ,
        total,
        dropdowns: !dropdowns ? null :
          {
            gender: this.prepareDropdownData('gender'),
            year: this.prepareDropdownData('year'),
            country: this.prepareDropdownData('country'),
            currency: this.prepareDropdownData('currency'),
            color: this.prepareDropdownData('color'),
          }
      }
    },
    async fetch(options) {
      //TODO: tmp fix
      if (!options) {
        options = {
          page: this.currentPage,
          itemsPerPage: this.pageSize,
          dropdowns: true
        };
      }

      this.options = options

      const result = await this.fetchData(options)
      this.items = result.items
      this.totalItems = result.total
      result.dropdowns && (this.dropdowns = result.dropdowns)
      //console.log(this.items)
      this.loading = false
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    }
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%
</style>
