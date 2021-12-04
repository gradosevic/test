<template lang="pug">
  v-container
    v-row
      v-col(cols)
        DataTable(
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
      fakeServerDelay: 300, //ms
      loading: false,
      options: {},
      headers: [
        {text: '#', value: 'id', align: 'start'},
        {text: 'Full Name', value: 'fullname', visible: true},
        {text: 'Email', value: 'email', visible: true},
        {text: 'Gender', value: 'gender', visible: true},
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
        name: dataType[0].toUpperCase() + dataType.slice(1,dataType.length) +
          (dataType.indexOf('year') === 0 ? ' / Any':' / Show All'),
        value: ''
      }, ...new Set(sales.results
        .map(item => {
          return {
            name: item[dataType],
            value: item[dataType]
          }
      }))].sort((a,b) => a.value > b.value ? 1 : -1)
    },
    async fetchData(options) {
      const {
        page,
        itemsPerPage,
        dropdowns,
        filter,
        sortBy,
        sortDesc,
        multiSort,
        search,
        enabledHeaders
      } = options;
      const start = (page - 1) * itemsPerPage
      await this.delay(this.fakeServerDelay)

      let data = sales.results;

      //FILTERING
      filter && (
        data = sales.results.filter(item => {
          if(!filter) return true;

          let allFiltersSatisfied = true;

          //DROPDOWNS
          Object.keys(filter).forEach(filterName => {

            //If no filter set, skip
            if(!filter[filterName] || !filter[filterName].value) return false;

            filterName === 'year_min' && filter[filterName].value > item['year'] && (allFiltersSatisfied = false);
            filterName === 'year_max' && filter[filterName].value < item['year'] && (allFiltersSatisfied = false);
            filterName.indexOf('year') !== 0 && (filter[filterName].value !== item[filterName]) && (allFiltersSatisfied = false);

          });

          //SEARCH TERM
          if(search) {
            if (!enabledHeaders) {
              JSON.stringify(item).toLowerCase().indexOf(search.toLowerCase()) === -1 && (allFiltersSatisfied = false);
            }else {
              let anyFieldSatisfied = false;
              enabledHeaders.forEach(enabledField => {
                const field = enabledField === 'fullname' ? item.user.first_name + ' ' + item.user.last_name : item[enabledField];
                field && (field.toString().toLowerCase().indexOf(search.toLowerCase()) !== -1) && (anyFieldSatisfied = true)
              })
              !anyFieldSatisfied && (allFiltersSatisfied = false);
            }
          }

          return allFiltersSatisfied;
        })
      );

      //SORTING
      sortBy && sortBy.length && (
        multiSort ?
          sortBy.forEach((field, index) => {
            data = this.sortData(data, field, sortDesc[index]);
          }) :
          data = this.sortData(data, sortBy[0], sortDesc[0])
      );

      let total = data.length;

      return {
        items: itemsPerPage !== -1 ?
          data.slice(start, start + itemsPerPage) :
          data,
        total,
        dropdowns: !dropdowns ? null :
          ['gender', 'year', 'country', 'currency', 'color']
            .reduce((acc,type)=> (acc[type]= this.prepareDropdownData(type), acc),{})
      }
    },
    sortData(data, field, descending){
      return data.sort((a,b) => {

        const firstVal = field === 'fullname' ? a.user.first_name + ' ' + a.user.last_name : a[field];
        const secondVal = field === 'fullname' ? b.user.first_name + ' ' + b.user.last_name: b[field];

        return descending ?
          (firstVal > secondVal ? -1 : 1) :
          (firstVal < secondVal ? -1 : 1)
      });
    },
    async fetch(options) {

      //default options for first use
      if (!options) {
        options = {
          page: this.currentPage,
          itemsPerPage: this.pageSize,
          dropdowns: true
        };
      }

      this.options = options
      this.loading = true
      const result = await this.fetchData(options)
      this.loading = false

      this.items = result.items
      this.totalItems = result.total
      result.dropdowns && (this.dropdowns = result.dropdowns)
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
