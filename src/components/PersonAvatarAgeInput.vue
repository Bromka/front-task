<script setup lang="ts">
import { computed, ref } from 'vue'
import NumericInput from '@/components/input/NumericInput'

const props = withDefaults(
  defineProps<{
    name: string
    label: string
    hint?: string
    avatarSrc?: string
    modelValue: number | null
  }>(),
  {
    hint: '',
  },
)

const emit = defineEmits<{
  (e: 'update:modelValue', value: number | null): void
}>()

const isFocused = ref(false)

const avatarClasses = computed(() => ({
  'person-avatar-age-input__avatar--input-focused': isFocused.value,
}))
</script>

<template>
  <div class="flex items-start gap-3">
    <div class="person-avatar-age-input__avatar__wrapper" :class="avatarClasses">
      <div
        class="person-avatar-age-input__avatar shrink-0 overflow-hidden rounded-full"
      >
        <img src="/img.png" :alt="props.name" class="w-full h-full object-cover" />
      </div>
    </div>

    <NumericInput
      :model-value="props.modelValue"
      :label="props.label"
      :hint="props.hint"
      @update:model-value="emit('update:modelValue', $event)"
      @focused-change="isFocused = $event"
    />
  </div>
</template>
<style>
.person-avatar-age-input__avatar__wrapper{
  width: 88px;
  height: 88px;
  padding: 4px;
  background-color: white;
  overflow: hidden;
  flex-shrink: 0;
  border-radius: 100%;
  align-items: center;
  justify-content: center;
  display: flex;
  box-sizing: border-box;
  border: 1px solid transparent;
}

.person-avatar-age-input__avatar__wrapper.person-avatar-age-input__avatar--input-focused {
  border: 1px solid var(--ni-avatar-border);
}

.person-avatar-age-input__avatar {
  width: 100%;
  height: 100%;
}

</style>
