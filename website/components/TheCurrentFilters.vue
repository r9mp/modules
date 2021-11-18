<template>
  <div class="flex items-center gap-1">
    <div>Filter{{ addsAnSAtTheEndOfFilterIdNeeded }}</div>
    <FilterLabel
      v-for="filter in currentFilters"
      :key="filter.key"
      @close="$emit('clear-one', filter.key)"
    >
      {{ filter.value }}
    </FilterLabel>
    <a
      href="/"
      class="ml-2 opacity-70 hover:opacity-100 inline-flex items-center gap-1"
      @click.prevent="$emit('clear-all')"
    >
      <UnoIcon class="i-carbon-filter-remove" />
      Clear filter{{ addsAnSAtTheEndOfFilterIdNeeded }}
    </a>
  </div>
</template>

<script>
export default {
  name: 'TheCurrentFilters',
  props: {
    currentFilters: {
      type: Array,
      default () {
        return []
      }
    }
  },
  computed: {
    addsAnSAtTheEndOfFilterIdNeeded () {
      return this.currentFilters.length > 1 ? 's' : ''
    }
  }
}
</script>
