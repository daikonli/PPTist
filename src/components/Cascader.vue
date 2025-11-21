<template>
  <div class="cascader-container">
    <Select 
      :value="selectedStage" 
      :options="stageOptions" 
      @update:value="handleStageChange"
      class="cascader-select"
    />
    <Select 
      :value="selectedSubject" 
      :options="subjectOptions" 
      @update:value="handleSubjectChange"
      class="cascader-select"
    />
  </div>
</template>

<script lang="ts" setup>
import { computed, watch } from 'vue'
import Select from './Select.vue'

interface SubjectOption {
  label: string
  value: string
}

interface CascaderOption {
  label: string
  value: string
  children: SubjectOption[]
}

const props = withDefaults(defineProps<{
  options: CascaderOption[]
  value?: string[]
}>(), {
  value: () => []
})

const emit = defineEmits<{
  (event: 'update:value', payload: string[]): void
}>()

const selectedStage = computed(() => props.value[0] || '')
const selectedSubject = computed(() => props.value[1] || '')

const stageOptions = computed(() => {
  return props.options.map(item => ({
    label: item.label,
    value: item.value
  }))
})

const subjectOptions = computed(() => {
  const stage = props.options.find(item => item.value === selectedStage.value)
  return stage?.children || []
})

const handleStageChange = (value: string | number) => {
  const stage = props.options.find(item => item.value === value)
  const firstSubject = stage?.children[0]?.value || ''
  emit('update:value', [value as string, firstSubject])
}

const handleSubjectChange = (value: string | number) => {
  emit('update:value', [selectedStage.value, value as string])
}

// 初始化：如果没有选中学科，自动选中第一个
watch(() => props.value, (newValue) => {
  if (newValue[0] && !newValue[1]) {
    const stage = props.options.find(item => item.value === newValue[0])
    if (stage?.children[0]) {
      emit('update:value', [newValue[0], stage.children[0].value])
    }
  }
}, { immediate: true })
</script>

<style lang="scss" scoped>
.cascader-container {
  display: flex;
  gap: 10px;
  padding: 10px;
}
.cascader-select {
  flex: 1;
}
</style>

