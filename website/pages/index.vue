<template>
  <div class="pb-16 relative bg-gray-100 dark:bg-secondary-black nuxt-text-default">
    <div
      class="relative bg-white shadow dark:bg-secondary-darkest w-full sticky top-0 z-50 bg-opacity-80 backdrop-filter backdrop-blur-[12px] border-none"
    >
      <TheSearch
        ref="theSearch"
        :search="getFilterFromKey('search').value"
        @update:search="updateFilterValueFromKey('search', $event)"
      />
    </div>
    <div
      class="pt-10 pb-16 px-3 lg:px-10 lg:pt-24 lg:pb-32 bg-white dark:bg-secondary-darkest dark:bg-secondary-darkest relative"
    >
      <div class="container mx-auto flex flex-col sm:flex-row justify-between">
        <div class="flex flex-wrap justify-between gap-y-5 w-full">
          <!-- Header -->
          <TheHeader />
          <!-- Stats -->
          <TheStats
            :modules="modules"
            :maintainers-total="maintainersTotal"
            :downloads-total="downloadsTotal"
          />
        </div>
      </div>
      <img
        loading="lazy"
        src="/img/page-hero/dark/mountains.svg"
        alt="A landscape image"
        class="absolute -bottom-1px left-0 w-full h-12 md:h-24 object-fill dark:hidden text-blue-600 pointer-events-none z-0"
      >
      <img
        loading="lazy"
        src="/img/page-hero/dark/dark_mountains.svg"
        alt="A landscape image"
        class="absolute -bottom-1px left-0 w-full h-12 md:h-24 object-fill light:hidden text-blue-600 pointer-events-none z-0"
      >
    </div>

    <!-- Body -->
    <div class="container px-4 mx-auto pt-8 grid grid-cols-1 lg:grid-cols-5 gap-8">
      <!-- Sidebar -->
      <div class="col-span-1 space-y-10 hidden lg:block">
        <!-- Nuxt versions -->
        <FilterButtons
          title="Nuxt version"
          subtitle="Show modules working with:"
          :items="versionsList"
          :selected-item="getFilterFromKey('version').value"
          @toggle="updateFilterValueFromKey('version', $event)"
        >
          <template #icon="{ icon }">
            <component :is="icon" class="h-6 w-6 mr-2 inline-block" />
          </template>
        </FilterButtons>

        <!-- Categories -->
        <FilterButtons
          title="Categories"
          :items="categoriesList"
          :selected-item="getFilterFromKey('category').value"
          @toggle="updateFilterValueFromKey('category', $event)"
        />
      </div>
      <!-- Main -->
      <div class="col-span-4">
        <!-- Filter -->
        <div class="h-10 -mt-5">
          <TheCurrentFilters
            v-if="displayFiltersBlock"
            :current-filters="currentFilters"
            @clear-one="clearOneFilter"
            @clear-all="clearFilters"
          />
        </div>

        <!-- Result, Sort -->
        <div class="flex flex-col items-center justify-between min-h-18 sm:flex-row p-5 mb-4 nuxt-border nuxt-card-bg rounded-lg">
          <div>
            <span class="font-black text-2xl">{{ filteredModules.length }}</span>
            module{{ filteredModules.length > 1 ? 's' : '' }} found
          </div>
          <div>
            <div v-show="!getFilterFromKey('search').value" class="flex items-center text-forest-night">
              <label
                for="options-menu"
                class="mr-3"
                @click="sortByMenuVisible = !sortByMenuVisible"
              >Sort by</label>
              <div class="flex border border-gray-400/20 rounded-md">
                <div class="relative w-28 my-auto">
                  <button
                    type="button"
                    :aria-label="`change sort`"
                    class="flex items-center justify-center w-full p-1 px-2 hover:bg-skborder-sky-lightest focus:bg-skborder-sky-lightest focus:outline-none hover:border-grey-light"
                    :class="sortByBtnClass"
                    @click="sortByMenuVisible = !sortByMenuVisible"
                  >
                    {{ sortByComp.label }}
                  </button>
                  <div
                    v-show="sortByMenuVisible"
                    class="absolute left-0 z-10 origin-top-right rounded-b-md shadow-lg border border-gray-400/20 shadow-xs bg-white dark:bg-secondary-darkest"
                  >
                    <div
                      id="options-menu"
                      role="menu"
                      aria-orientation="vertical"
                      aria-labelledby="options-menu"
                    >
                      <button
                        v-for="(option, key) in sortByOptions"
                        :key="key"
                        type="button"
                        :aria-label="`sort by ${key}`"
                        class="flex items-center justify-center p-1 px-2 w-28 hover:bg-cloudy-grey focus:text-grey-darkest text-forest-night focus:outline-none rounded-b-md"
                        @click="selectSortBy(key)"
                      >
                        {{ option.label }}
                      </button>
                    </div>
                  </div>
                </div>
                <div class="relative">
                  <button
                    type="button"
                    :aria-label="orderBy === 'asc' ? 'sort ascending' : 'sort descending'"
                    class="flex items-center p-2 hover:bg-skborder-sky-lightest focus:bg-skborder-sky-lightest focus:outline-none rounded-r-md"
                    @click="toggleOrderBy"
                  >
                    <UnoIcon :class="orderBy === 'asc' ? 'i-carbon-sort-ascending' : 'i-carbon-sort-descending'" />
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div
          class="grid gap-x-6 gap-y-8 mt-10"
          style="grid-template-columns: repeat(auto-fit, minmax(300px, 1fr))"
        >
          <template v-for="mod of pageFilteredModules">
            <LazyHydrate :key="mod.name" when-visible>
              <CardModule :mod="mod" />
            </LazyHydrate>
          </template>
          <Observer @intersect="intersectedModulesLoading" />
        </div>
      </div>
    </div>

    <!-- Footer -->
    <TheFooter />
  </div>
</template>

<script>
import LazyHydrate from 'vue-lazy-hydration'
import Fuse from 'fuse.js/dist/fuse.basic.esm'
import { isMobile } from '~/utils/detectUserAgent.ts'
import { CATEGORIES_ICONS, FIELDS, MODULE_INCREMENT_LOADING, ORDERS, VERSIONS } from '~/composables/constants'
import { fetchModules } from '~/composables/fetch'

const sort = (a, b, asc) => asc ? a - b : b - a

const sortFields = {
  [FIELDS.DOWNLOADS]: {
    label: 'Downloads'
  },
  [FIELDS.STARS]: {
    label: 'Stars'
  }
}

export default {
  components: { LazyHydrate },
  directives: {
    focus: {
      // directive definition
      inserted (el) {
        el?.focus()
      }
    }
  },
  async asyncData () {
    return await fetchModules()
  },
  data () {
    return {
      filters: [
        { key: 'search', value: '' },
        { key: 'category', value: '' },
        { key: 'version', value: '' }
      ],
      orderBy: ORDERS.DESC,
      sortBy: 'downloads',
      sortByMenuVisible: false,
      moduleLoaded: MODULE_INCREMENT_LOADING,
      CATEGORIES_ICONS,
      versionsList: VERSIONS
    }
  },
  computed: {
    currentFilters () {
      return this.filters.filter(filter => !!filter.value)
    },
    displayFiltersBlock () {
      return this.nbFiltersApplied >= 1
    },
    nbFiltersApplied () {
      return this.filters.filter(filter => !!filter.value).length
    },
    categoriesList () {
      const categoriesList = []
      for (const [key, icon] of Object.entries(CATEGORIES_ICONS)) {
        categoriesList.push({
          key,
          icon,
          label: key
        })
      }

      return categoriesList
    },
    filteredModules () {
      let modules = this.modules
      const search = this.getFilterFromKey('search').value
      if (search) {
        modules = this.fuse.search(search).map(r => r.item)
      } else {
        // Sort only if no search
        modules.sort((a, b) => sort(a[this.sortBy], b[this.sortBy], this.orderBy === ORDERS.ASC))
      }
      const category = this.getFilterFromKey('category').value
      if (category) {
        modules = modules.filter(module => module.category === category)
      }
      const version = this.getFilterFromKey('version').value
      if (version) {
        modules = modules.filter(module => module.compatibility[version] === 'working')
      }

      return modules
    },
    pageFilteredModules () {
      const filteredModules = Object.assign([], this.filteredModules)
      return filteredModules.splice(0, this.moduleLoaded)
    },
    sortByComp () {
      return sortFields[this.sortBy]
    },
    sortByOptions () {
      const options = {}

      for (const key in sortFields) {
        if (key === this.sortBy) { continue }

        options[key] = {
          ...sortFields[key]
        }
      }

      return options
    },
    sortByBtnClass () {
      return this.sortByMenuVisible ? 'rounded-bl-none border-b-grey-light' : ''
    }
  },
  watch: {
    filters: {
      deep: true,
      handler () {
        this.syncURL()
      }
    },
    orderBy () {
      this.syncURL()
    },
    sortBy () {
      this.syncURL()
    },
    $route () {
      this.applyURLQueryParams()
    }
  },
  mounted () {
    const fuseOptions = {
      threshold: 0.1,
      keys: [
        'name',
        'npm',
        'category',
        'maintainers.name',
        'maintainers.github',
        'description',
        'repo',
        'tags'
      ]
    }
    const index = Fuse.createIndex(fuseOptions.keys, this.modules)
    this.fuse = new Fuse(this.modules, fuseOptions, index)

    this.applyURLQueryParams()

    // In case of desktop, auto focus the search input
    if (!isMobile()) {
      this.focusSearchInput()
    }
  },
  beforeMount () {
    window.addEventListener('keypress', this.searchFocusListener)
  },
  beforeDestroy () {
    window.removeEventListener('keypress', this.searchFocusListener)
  },
  methods: {
    getFilterFromKey (key) {
      return this.filters.find(filter => filter.key === key)
    },
    updateFilterValueFromKey (key, value) {
      const filter = this.getFilterFromKey(key)
      filter.value = value
    },
    getFiltersKeys () {
      return this.filters.map(filter => filter.key)
    },
    getVersionFromKey (key) {
      const version = this.versionsList.find(version => version.key === key)

      return version ?? {}
    },
    clearFilters () {
      this.filters.forEach((filter) => { filter.value = '' })
      this.moduleLoaded = MODULE_INCREMENT_LOADING
    },
    clearOneFilter (key) {
      this.getFilterFromKey(key).value = ''
    },
    syncURL () {
      this.resetModuleLoaded()

      const query = {}
      this.filters.forEach((filter) => {
        if (filter.value) {
          query[filter.key] = filter.value
        }
      })

      if (this.orderBy !== ORDERS.DESC) {
        query.orderBy = this.orderBy
      }
      if (this.sortBy !== FIELDS.DOWNLOADS) {
        query.sortBy = this.sortBy
      }

      this.$router.push({ path: this.$route.path, query })
    },
    applyURLQueryParams () {
      const { sortBy, orderBy } = this.$route.query
      if (sortBy) { this.sortBy = sortBy }
      if (orderBy) { this.orderBy = orderBy }

      this.getFiltersKeys().forEach((filter) => {
        if (this.$route.query[filter]) {
          this.updateFilterValueFromKey(filter, this.$route.query[filter])
        }
      })
    },
    toggleOrderBy () {
      this.orderBy = (this.orderBy === ORDERS.ASC) ? ORDERS.DESC : ORDERS.ASC
    },
    selectSortBy (field) {
      this.sortBy = field
      this.sortByMenuVisible = false
    },
    intersectedModulesLoading () {
      this.moduleLoaded += MODULE_INCREMENT_LOADING
    },
    resetModuleLoaded () {
      this.moduleLoaded = MODULE_INCREMENT_LOADING
    },
    searchFocusListener (event) {
      // Add `/` shortcut for search input only if not already focused
      if (event.keyCode === 47 && !(event.target instanceof HTMLInputElement)) {
        event.preventDefault()
        this.focusSearchInput()
      }
    },
    focusSearchInput () {
      this.$refs.theSearch?.$refs.searchModuleInput?.focus()
    }
  }
}
</script>
