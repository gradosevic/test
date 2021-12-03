<template lang="pug">
  v-data-table(
    :headers="headers"
    :items="items"
    item-key="email"
    :search="search"
    :custom-filter="customFilterSearch"
    :options.sync="options"
    :server-items-length="totalItems"
    :loading="loading"
    dense
  ).elevation-1.mt-10
     template(v-slot:item.fullname='{ item }')
      | {{ item.user.title + '. ' + item.user.first_name + ' ' + item.user.last_name }}
     template(v-slot:top)
       v-text-field(v-model="search"  label="Search" class="mx-4")
</template>
<script>
export default {
  props: ['headers', 'items', 'loading'],
  data() {
    return {
      search: '',
      options: {},
      //desserts: [],
    }
  },
  computed: {
      totalItems(){
        return this.items.length
      }
  },
  methods: {
    customFilterSearch (value, search, item) {
      //console.log(value, search, item);
      return value != null &&
        search != null &&
        typeof value === 'string' &&
        value.toString().indexOf(search) !== -1
    },
  },
}
</script>

<style lang="sass" scoped>
  .v-data-table
    max-width: 100%
</style>
