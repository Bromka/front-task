<template>
  <div class="numeric-input" :class="{ disabled, focused, error: hasError, hover}"
       @mouseenter="hover = true"
       @mouseleave="hover = false"
  >
    <label v-if="label" class="numeric-input__label">
      {{ label }}
      <span v-if="required" class="numeric-input__required">*</span>
    </label>

    <div class="numeric-input__field_wrapper">
      <div class="numeric-input__wrapper">
        <input
          ref="inputRef"
          class="numeric-input__field"
          type="text"
          inputmode="numeric"
          :size="inputSize"
          :value="displayValue"
          :disabled="disabled"
          :placeholder="placeholder"
          :aria-label="label || placeholder"
          :aria-invalid="hasError"
          @focus="onFocus"
          @blur="onBlur"
          @input="onInput"
          @keydown="onKeydown"
        />
      </div>
      <span v-if="hint" class="numeric-input__hint">{{ hint }}</span>
    </div>



  </div>
</template>

<script>
export default {
  name: 'NumericInput'
}
</script>
<script setup>
import { ref, computed, watch, nextTick } from 'vue'
import './NumericInput.css'

const props = defineProps({
  modelValue: {
    type: Number,
    default: null,
  },
  min: {
    type: Number,
    default: -Infinity,
  },
  max: {
    type: Number,
    default: Infinity,
  },
  step: {
    type: Number,
    default: 1,
  },
  label: {
    type: String,
    default: '',
  },
  placeholder: {
    type: String,
    default: '0',
  },
  hint: {
    type: String,
    default: '',
  },
  required: {
    type: Boolean,
    default: false,
  },
  disabled: {
    type: Boolean,
    default: false,
  },
  repeatDelay: {
    type: Number,
    default: 500,
  },
  repeatInterval: {
    type: Number,
    default: 80,
  },
})

const emit = defineEmits(['update:modelValue', 'change', 'focus', 'blur', 'focused-change'])

const inputRef = ref(null)
const focused = ref(false)
const rawInput = ref('')
const isEditing = ref(false)
const hover = ref(false)

const clampedValue = computed(() => {
  if (props.modelValue === null || props.modelValue === undefined) return null
  return clamp(props.modelValue)
})

function formatDisplay(v) {
  return v.toLocaleString('ru-RU')
}

function formatEditableInput(value) {
  if (value === '' || value === '-') return value

  const negative = value.startsWith('-')
  const digits = value.replace('-', '')
  const groupedDigits = digits.replace(/\B(?=(\d{3})+(?!\d))/g, ' ')

  return negative ? `-${groupedDigits}` : groupedDigits
}

const displayValue = computed(() => {
  if (isEditing.value) return formatEditableInput(rawInput.value)
  if (clampedValue.value === null) return ''
  return formatDisplay(clampedValue.value)
})

const inputSize = computed(() => {
  const text = displayValue.value
  // `size` is in characters; keep a sensible minimum so it doesn't jitter.
  return Math.max(4, String(text || props.placeholder || '').length)
})

const errorMessage = computed(() => {
  if (props.required && clampedValue.value === null) return 'Обязательное поле'
  if (clampedValue.value !== null) {
    if (clampedValue.value < props.min) return `Минимум: ${props.min}`
    if (clampedValue.value > props.max) return `Максимум: ${props.max}`
  }
  return ''
})
const hasError = computed(() => !!errorMessage.value)

function clamp(v) {
  return Math.min(props.max, Math.max(props.min, v))
}

function emitValue(v) {
  const result = clamp(Math.trunc(v))
  emit('update:modelValue', result)
  emit('change', result)
}

function increment() {
  emitValue((clampedValue.value ?? 0) + props.step)
}

function decrement() {
  emitValue((clampedValue.value ?? 0) - props.step)
}

let repeatTimeout = null
let repeatInterval_ = null

function startRepeat(fn) {
  stopRepeat()
  repeatTimeout = setTimeout(() => {
    repeatInterval_ = setInterval(fn, props.repeatInterval)
  }, props.repeatDelay)
}

function stopRepeat() {
  clearTimeout(repeatTimeout)
  clearInterval(repeatInterval_)
}

function onFocus(e) {
  focused.value = true
  isEditing.value = true
  rawInput.value = clampedValue.value !== null ? String(clampedValue.value) : ''
  emit('focus', e)
  emit('focused-change', true)
}

function onBlur(e) {
  focused.value = false
  isEditing.value = false
  const parsed = parseInt(rawInput.value, 10)
  if (!isNaN(parsed)) {
    emitValue(parsed)
  } else if (rawInput.value === '' && !props.required) {
    emit('update:modelValue', null)
  }
  emit('blur', e)
  emit('focused-change', false)
}

function onInput(e) {
  const inputEl = e.target
  const cursor = inputEl.selectionStart ?? inputEl.value.length
  const textBeforeCursor = inputEl.value.slice(0, cursor)
  const digitsBeforeCursor = (textBeforeCursor.match(/\d/g) || []).length
  const minusBeforeCursor = textBeforeCursor.includes('-')

  // Allow only digits and a leading minus sign.
  const filtered = inputEl.value.replace(/[^\d-]/g, '').replace(/(?!^)-/g, '')
  rawInput.value = filtered

  const formatted = formatEditableInput(filtered)
  inputEl.value = formatted

  nextTick(() => {
    const activeInput = inputRef.value
    if (!activeInput) return

    let nextCursor = 0
    let digitsSeen = 0

    if (minusBeforeCursor && formatted.startsWith('-')) {
      nextCursor = 1
    }

    while (nextCursor < formatted.length && digitsSeen < digitsBeforeCursor) {
      if (/\d/.test(formatted[nextCursor])) {
        digitsSeen += 1
      }
      nextCursor += 1
    }

    activeInput.setSelectionRange(nextCursor, nextCursor)
  })
}

function onKeydown(e) {
  if (e.key === 'ArrowUp')   { e.preventDefault(); increment() }
  if (e.key === 'ArrowDown') { e.preventDefault(); decrement() }
  if (e.key === 'Enter')     { inputRef.value?.blur() }
}

watch(() => props.modelValue, (val) => {
  if (!isEditing.value) {
    rawInput.value = val !== null && val !== undefined ? String(val) : ''
  }
})
</script>
