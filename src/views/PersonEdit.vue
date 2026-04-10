<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { store } from '@/store'

import PersonAvatarAgeInput from '@/components/PersonAvatarAgeInput.vue'

const route = useRoute()

const person = computed(() => {
  const id = Number(route.params.id)
  return store.people.find((p) => p.id === id)
})

const label = computed(() => (`${person.value?.name ?? ''} is`).toUpperCase())

</script>

<template>
  <div v-if="person" class="flex flex-col gap-4">
    <router-link to="/" class="text-violet-600 hover:underline text-sm">&larr; Back</router-link>
    <PersonAvatarAgeInput v-model="person.ageInHours" :name="person.name" :label="label" hint=" hours old" />
  </div>

  <div v-else>
    <p class="text-gray-600">Person not found</p>
    <router-link to="/" class="text-violet-600 hover:underline text-sm">Back to list</router-link>
  </div>
</template>
