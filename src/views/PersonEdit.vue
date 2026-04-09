<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { store } from '@/store'

import NumericInput from '@/components/input/NumericInput'

const route = useRoute()

const person = computed(() => {
  const id = Number(route.params.id)
  return store.people.find((p) => p.id === id)
})

function updateAge(value: string) {
  if (person.value) {
    person.value.ageInHours = Number(value) || 0
  }
}

const label = computed<string>(() => (`${person.value?.name || ""} is`).toUpperCase(), {})

</script>

<template>
  <div v-if="person" class="flex flex-col gap-4">
    <router-link to="/" class="text-violet-600 hover:underline text-sm">&larr; Back</router-link>

    <div class="flex items-center gap-3">
      <img
        src="/img.png"
        :alt="person.name"
        class="w-14 h-14 rounded-full border-2 border-violet-500 object-cover"
      />
      <NumericInput v-model="person.ageInHours" :label="label" hint=" hours old"/>
    </div>
  </div>

  <div v-else>
    <p class="text-gray-600">Person not found</p>
    <router-link to="/" class="text-violet-600 hover:underline text-sm">Back to list</router-link>
  </div>
</template>
