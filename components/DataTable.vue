<template lang="pug">
  v-data-table(
    :headers="fields"
    :items="items"
    :class=`striped ? 'striped': ''`
    item-key="email"
    :search="search"
    :options.sync="options"
    :server-items-length="totalItems"
    :loading="loading"
    :page="currentPage"
    :multi-sort="multiSort"
    :dark="darkTheme"
    :footer-props="{'items-per-page-options': [5, 10, 15, 20, 50, 100, -1]}"
    dense
  ).elevation-1.mt-10
     template(v-slot:item.fullname='{ item }')
      | {{ item.user.title + '. ' + item.user.first_name + ' ' + item.user.last_name }}
     template(v-slot:item.email='{ item }' v-html)
      | <a :href="`mailto:${item.email}`">{{item.email}}</a>
     template(v-slot:item.color='{ item }')
       div(class="item-color" :style="`color: ${item.color};`")
        | {{item.color}}
     template(v-slot:no-data)
       div(class="ma-4")
        | <v-icon>mdi-close-circle</v-icon> <span class="ma-1">The table is empty. Please change or reset your filters.</span>
     template(v-slot:top)
      v-container
        v-row
         v-col(cols
                v-for="type in enabledFilters"
                :key="type"
                v-if="type !== 'year_min' && type !== 'year_max'"
                )
              v-select(
              v-model="filter[type]"
              :hint="filter[type] && filter[type].value ? `Filtering data by ${type}` :`Filter not applied`"
              :items="dropdowns[type]"
              item-text="name"
              item-value="value"
              :label="`Filter by ${type}`"
              persistent-hint
              v-on:change="onFilterChanged(type)"
              style="flex:1"
              return-object
              single-line)
         v-col(cols
            class="d-flex"
            v-if="enabledFilters.indexOf('year_min') !== -1")
              v-select(
              v-model="filter.year_min"
              :hint="filter.year_min ? `Min year` :`Min year (not set)`"
              :items="minYear"
              item-text="name"
              item-value="value"
              :label="`Filter by Year (min)`"
              persistent-hint
              v-on:change="onFilterChanged('year_min')"
              style="flex:1"
              return-object
              single-line)

              v-select(
              v-model="filter.year_max"
              :hint="filter.year_max ? `Max year` :`Max year (not set)`"
              :items="maxYear"
              item-text="name"
              item-value="value"
              :label="`Filter by Year (max)`"
              persistent-hint
              v-on:change="onFilterChanged('year_max')"
              style="flex:1"
              return-object
              single-line)

        v-row
          v-col(class="flex-md-grow-1 d-flex search-box")
            <v-icon>mdi-magnify</v-icon>
            v-text-field(
              v-model="search"
              class="ma-2"
              v-on:change="onSearch"
              v-on:keyup="onSearchKeyUp"
              hint="Searching in all fields"
              label="Search")
          v-col(class="flex-md-grow-0 d-md-flex")
            v-btn(
               depressed
               v-on:click="onResetFilters"
               ).ma-3
                | <v-icon>mdi-refresh</v-icon> <span class="ma-2">Reset Filters</span>
            v-btn(
               :primary="manageFields"
               :depressed="!manageFields"
               v-on:click="manageFields = !manageFields"
               ).ma-3
                | <v-icon v-if="!manageFields">mdi-format-paint</v-icon> <v-icon v-if="manageFields">mdi-close-circle</v-icon> <span class="ma-2">{{!manageFields ? 'Customize': 'Close'}}</span>
        v-row(v-if="manageFields")
          v-col(cols)
            h2
              | Options
        v-row(v-if="manageFields")
          v-col(cols)
            v-checkbox(
             v-model="multiSort"
            :label="`Multisort columns`")
          v-col(cols)
            v-checkbox(
             v-model="instantSearch"
            :label="`Instant search`")
          v-col(cols)
            v-checkbox(
             v-model="searchInVisibleFieldsOnly"
            :label="`Search visible fields only`")
          v-col(cols)
            v-checkbox(
             v-model="striped"
            :label="`Zebra table`")
          v-col(cols)
            v-checkbox(
             v-model="darkTheme"
            :label="`Dark theme`")
        v-row(v-if="manageFields")
          v-col(cols)
            h2
              | Show Fields
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
      manageFields: false, //toggle on/of view for managing fields
      multiSort: false, //OPTION - toggle multisort
      instantSearch: true, //OPTION - toggle instant search
      searchInVisibleFieldsOnly: false, //OPTION - toggle search by only visible fields
      striped: true, //OPTION - toggle striped table
      darkTheme: false, //OPTION - toggle dark theme
      triggerSearchMinWordLength: 3, //Min search term length to trigger auto-search
      filter:{
        gender: '',
        country: '',
        currency: '',
        color: '',
        year_min: '',
        year_max: ''
      },
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
        return Object.keys(this.filter).filter(item => {
          if(item.indexOf('year') === 0) {
            return this.enabledHeaders.indexOf('year') !== -1
          }
          return this.enabledHeaders.indexOf(item) !== -1
        });
      },
      minYear(){
        return !this.filter.year_max ?
          this.dropdowns['year'] :
          this.dropdowns['year'].filter(year => !year.value || year.value <= this.filter.year_max.value)
      },
      maxYear(){
        return !this.filter.year_min ?
          this.dropdowns['year'] :
          this.dropdowns['year'].filter(year => !year.value || year.value >= this.filter.year_min.value)
      }
  },
  watch: {
    options: {
      handler(newOptions) {
        this.fetch(newOptions)
      },
    },
    deep: true,
  },
  methods: {
    onSearch(){
        this.fetch();
    },
    onSearchKeyUp(e){
      if(this.instantSearch && e.target.value.length >= this.triggerSearchMinWordLength){
        this.fetch();
      }
    },
    fetch(options){
      options = options ?? this.options;
      options.filter = this.filter;
      options.search = this.search;
      this.searchInVisibleFieldsOnly && (options.enabledHeaders = this.enabledHeaders);
      this.$emit('fetch', options);
    },
    onFilterChanged(filterType){
      this.fetch()
    },
    onResetFilters(){
      this.search = '';
      Object.keys(this.filter).forEach(filterType => this.filter[filterType] = '');
      this.fetch();
    }
  },
}
</script>

<style lang="sass" scoped>
  .v-data-table
    max-width: 100%
  .search-box
    min-width: 200px
  .item-color
    text-shadow: -1px 1px 1px #555
</style>
<style lang="sass">
.v-data-table.striped tr:nth-child(even)
  background-color: #f5f5f5
.v-data-table.striped.theme--dark tr:nth-child(even)
  background-color: #333333
</style>
