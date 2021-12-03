<template lang="pug">
  v-data-table(
    :headers="fields"
    :items="items"
    item-key="email"
    :search="search"
    :custom-filter="customFilterSearch"
    :options.sync="options"
    :server-items-length="totalItems"
    :loading="loading"
    :page="currentPage"
    update:sortBy="onSortByUpdate"
    dense
  ).elevation-1.mt-10
     template(v-slot:item.fullname='{ item }')
      | {{ item.user.title + '. ' + item.user.first_name + ' ' + item.user.last_name }}
     template(v-slot:top)
      v-container
        v-row
         v-col(cols
                v-for="type in enabledFilters"
                :key="type"
                )
              v-select(
              v-model="filter[type]"
              :hint="`Filter by ${type}`"
              :items="dropdowns[type]"
              item-text="name"
              item-value="value"
              :label="`Filter by ${type}`"
              persistent-hint
              v-on:change="onFilterChanged(type)"
              style="flex:1"
              return-object
              single-line)
        v-row
          v-col
            v-text-field(v-model="search" label="Search")
            v-date-picker(
              class="mt-3 mb-6"
              v-model="computeRange"
              range
              )
          v-col(cols)
            v-btn(
               depressed
               v-on:click="onResetFilters"
               ).ma-3
                | Reset filters
            v-btn(
               depressed
               v-on:click="onManageFields"
               ).ma-3
                | Manage fields
        v-row(v-if="manageFields")
          v-col(cols
                v-for="header in headers"
                :key="header.value"
                style="white-space:nowrap"
                )
            v-checkbox(
             v-model="header.visible"
            :label="`${header.text}`")


</template>
<script>
export default {
  props: [
    'headers',
    'items',
    'loading',
    'currentPage',
    'totalItems',
    'dropdowns'
  ],
  data() {
    return {
      search: '',
      options: {},
      manageFields: false,
      checkbox: false,
      range: [],
      filter:{
        gender: '',
        country: '',
        currency: '',
        color: ''
      },
      //desserts: [],
    }
  },
  computed: {
      fields(){
        return this.headers.filter(header => header.visible)
      },
      enabledHeaders(){
        return this.headers
          .filter(header => header.visible)
          .map(header => header.value);
      },
      enabledFilters(){
        return Object.keys(this.filter).filter(item => this.enabledHeaders.indexOf(item) !== -1);
      },
    computeRange: {
      get() {
        return this.range;
      },
      set([firstDay]) {
        const fourthDay = new Date(new Date(firstDay)
          .setDate(new Date(firstDay).getDate() + 4))
          .toISOString()
          .slice(0, 10);
        this.range = [firstDay, fourthDay];
      },
    },
  },
  watch: {
    options: {
      handler(newOptions, oldOptions) {
        this.fetch(newOptions)
      },
    },
    deep: true,
  },
  methods: {
    customFilterSearch (value, search, item) {
      console.log('s', value, search, item);
      return value != null &&
        search != null &&
        typeof value === 'string' &&
        value.toString().indexOf(search) !== -1
    },
    fetch(options){
      options = options ?? this.options
      options.filter = this.filter
      this.$emit('fetch', options);
    },
    onPageUpdate(){
      console.log('opu')
    },
    onSortByUpdate(){
      console.log('sort')
    },
    onFilterChanged(filterType){
      console.log(this.filter[filterType].value);
      this.fetch()
    },
    onResetFilters(){
      Object.keys(this.filter).forEach(filterType => this.filter[filterType] = '')
    },
    onManageFields(){
      this.manageFields = !this.manageFields
    }
  },
}
</script>

<style lang="sass" scoped>
  .v-data-table
    max-width: 100%
</style>
