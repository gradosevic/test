<template lang="pug">
  v-container
    h2
      | Filters
    v-row
      v-col(cols
            v-for="type in Object.keys(this.filter)"
            :key="type"
            )
          v-select(
          v-model="filter[type]"
          :hint="filter[type] ? `Filter by ${type}` :''"
          :items="dropdowns[type]"
          item-text="state"
          item-value="abbr"
          :label="`Filter by ${type}`"
          persistent-hint
          v-on:change="onFilterChanged(type)"
          return-object
          single-line)
      v-col(cols)
         v-btn(
           depressed
           v-on:click="onResetFilters"
           )
            | Reset filters
    h2
      | Data
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
      filter:{
        gender: '',
        country: '',
        currency: '',
        color: ''
      },
      pageSize: 10,
      currentPage: 1,
      fakeServerDelay: 10, //ms
      loading: false,
      options: {},
      headers: [
        {text: '#', value: 'id', align: 'start'},
        {text: 'Name', value: 'fullname'},
        {text: 'Email', value: 'email'},
        {text: 'Gender', value: 'gender'},
        {text: 'IP Address', value: 'ip_address'},
        {text: 'Year', value: 'year'},
        {text: 'Sales', value: 'sales'},
        {text: 'Country', value: 'country'},
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
      return ['-- Show All --', ...new Set(sales.results.map(item => {
        return item[dataType]
      }))].sort()
    },
    async fetchData(options) {
      const {page, itemsPerPage, dropdowns} = options;
      const start = (page - 1) * itemsPerPage
      await this.delay(this.fakeServerDelay)

      console.log('showing...', dropdowns)

      console.log()
      return {
        items: itemsPerPage !== -1 ? sales.results.slice(start, start + itemsPerPage) : sales.results,
        total: sales.results.length,
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
      console.log('options', options)

      //TODO: tmp fix
      if (!options) {
        options = {
          page: this.currentPage,
          itemsPerPage: this.pageSize,
          dropdowns: true
        };
      }

      const result = await this.fetchData(options)
      this.items = result.items
      this.totalItems = result.total
      result.dropdowns && (this.dropdowns = result.dropdowns)
      //console.log(this.items)
      this.loading = false
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },
    onFilterChanged(filterType){
      console.log(this.filter[filterType]);
    },
    onResetFilters(){
      Object.keys(this.filter).forEach(filterType => this.filter[filterType] = '')
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
