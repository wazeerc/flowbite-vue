<template>
  <div>
    <div v-if="!dropzone">
      <label>
        <span :class="labelClasses">{{ label }}</span>
        <input
          :accept="accept"
          :class="fileInpClasses"
          :multiple="multiple"
          type="file"
          @change="handleChange"
        >
      </label>
      <slot />
    </div>
    <div
      v-else
      class="flex flex-col items-start justify-center"
      @change="handleChange"
      @dragover="dragOverHandler"
      @drop="dropFileHandler"
    >
      <span
        v-if="label !== ''"
        :class="labelClasses"
      >{{ label }}</span>
      <label :class="dropzoneLabelClasses">
        <div :class="dropzoneWrapClasses">
          <svg
            aria-hidden="true"
            class="size-8 text-gray-500 dark:text-gray-400"
            fill="none"
            viewBox="0 0 20 16"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M13 13h3a3 3 0 0 0 0-6h-.025A5.56 5.56 0 0 0 16 6.5 5.5 5.5 0 0 0 5.207 5.021C5.137 5.017 5.071 5 5 5a4 4 0 0 0 0 8h2.167M10 15V6m0 0L8 8m2-2 2 2"
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              stroke="currentColor"
            />
          </svg>
          <div v-if="!model">
            <p :class="dropzoneTextClasses">
              <span class="font-semibold">Click to upload</span>
              or drag and drop
            </p>
            <slot />
          </div>
          <p v-else>File: {{ dropZoneText }}</p>
        </div>
        <input
          :accept="accept"
          :multiple="multiple"
          class="hidden"
          type="file"
          @change="handleChange"
        >
      </label>
    </div>
  </div>
</template>

<script setup lang="ts">
import { isArray } from 'lodash-es'
import { computed } from 'vue'

import { useFileInputClasses } from './composables/useFileInputClasses'

interface FileInputProps {
  accept?: string
  dropzone?: boolean
  label?: string
  modelValue?: File | File[] | null
  multiple?: boolean
  size?: string
}

const props = withDefaults(defineProps<FileInputProps>(), {
  accept: '',
  dropzone: false,
  label: '',
  modelValue: null,
  multiple: false,
  size: 'sm',
})

const dropZoneText = computed(() => {
  if (isArray(props.modelValue)) {
    return props.modelValue.map(el => el.name).join(', ')
  } else if (props.modelValue instanceof FileList) {
    return Array.from(props.modelValue)
      .map(el => el.name)
      .join(',')
  } else if (props.modelValue instanceof File) {
    return props.modelValue.name || ''
  }

  return ''
})

const emit = defineEmits(['update:modelValue'])
const model = computed({
  get () {
    return props.modelValue
  },
  set (val) {
    emit('update:modelValue', val)
  },
})

const handleChange = (event: Event) => {
  const target = event.target as HTMLInputElement
  if (props.multiple) {
    const newFiles = Array.from(target.files ?? [])
    if (Array.isArray(model.value) && model.value.length > 0) {
      model.value = [...model.value, ...newFiles]
    } else {
      model.value = newFiles
    }
  } else {
    model.value = target.files?.[0] ?? null
  }
}

const dropFileHandler = (event: DragEvent) => {
  event.preventDefault()
  const arr: File[] = []
  if (event.dataTransfer?.items) {
    Object.values(event.dataTransfer.items)
      .forEach((item: DataTransferItem) => {
        if (item.kind === 'file') {
          arr.push(item.getAsFile() as File)
        }
      })
    if (props.multiple) {
      if (Array.isArray(model.value) && model.value.length > 0) {
        model.value = [...model.value, ...arr]
      } else {
        model.value = arr
      }
    } else {
      model.value = arr[0] || null
    }
  } else if (event.dataTransfer?.files) {
    const files = Array.from(event.dataTransfer.files)
    if (props.multiple) {
      if (Array.isArray(model.value) && model.value.length > 0) {
        model.value = [...model.value, ...files]
      } else {
        model.value = files
      }
    } else {
      model.value = files[0] || null
    }
  }
}

const dragOverHandler = (event: Event) => {
  event.preventDefault()
}

const {
  dropzoneLabelClasses,
  dropzoneTextClasses,
  dropzoneWrapClasses,
  fileInpClasses,
  labelClasses,
} = useFileInputClasses(props.size)
</script>
