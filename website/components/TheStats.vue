<template>
  <div>
    <dl class="flex gap-4 md:gap-8 mt-auto h-full">
      <div class="flex-col space-y-2 p-4 rounded-md">
        <span
          class="font-normal leading-6 text-sm sm:mt-2 sm:text-sm sm:capitalize"
          aria-hidden="true"
        >Total</span>
        <dd
          class="text-4xl font-black text-primary dark:text-primary leading-none sm:text-6xl"
          aria-describedby="item-1"
        >
          {{ modules.length }}
        </dd>
        <dt
          id="item-1"
          class="font-normal leading-6 text-sm sm:mt-2 sm:text-sm sm:capitalize"
        >
          modules
        </dt>
      </div>
      <div class="flex-col space-y-2 p-4">
        <span
          class="font-normal leading-6 text-sm sm:mt-2 sm:text-sm sm:capitalize"
          aria-hidden="true"
        >Total</span>
        <dd
          class="text-4xl font-black text-primary dark:text-primary leading-none sm:text-6xl"
        >
          {{ numberFormatter(downloadsTotal) }}
        </dd>
        <dt
          class="font-medium leading-6 text-sm sm:mt-2 sm:text-sm sm:capitalize"
        >
          downloads last 30 days
        </dt>
      </div>
      <div class="flex-col space-y-2 p-4 rounded-md">
        <span
          class="font-normal leading-6 text-sm sm:mt-2 sm:text-sm sm:capitalize"
          aria-hidden="true"
        >Total</span>
        <dd
          class="text-4xl font-black text-primary dark:text-primary leading-none sm:text-6xl"
        >
          {{ maintainersTotal }}
        </dd>
        <dt
          class="font-medium leading-6 text-sm sm:mt-2 sm:text-sm sm:capitalize"
        >
          maintainers
        </dt>
      </div>
    </dl>
  </div>
</template>

<script setup lang="ts">
import { numberFormatter } from '~/utils/format'
import { ModuleInfo } from '~/../types'

defineProps<{
  modules: ModuleInfo[],
  maintainersTotal: number,
  downloadsTotal: number
}>()
</script>
