<template>
  <div class="tabs"
    :class="{
      'card': card,
      'space-around': spaceAround,
      'space-between': spaceBetween,
    }" 
    :style="tabsStyle || {}"
  >
    <div 
      class="tab" 
      :class="{ 'active': tab.key === value }"
      v-for="tab in tabs" 
      :key="tab.key"
      :style="{
        ...(tabStyle || {}),
        '--color': tab.color,
      }"
      @click="emit('update:value', tab.key)"
    >{{tab.label}}</div>
  </div>
</template>

<script lang="ts" setup>
import { type CSSProperties } from 'vue'

interface TabItem {
  key: string
  label: string
  color?: string
}

withDefaults(defineProps<{
  value: string
  tabs: TabItem[]
  card?: boolean
  tabsStyle?: CSSProperties
  tabStyle?: CSSProperties
  spaceAround?: boolean
  spaceBetween?: boolean
}>(), {
  card: false,
  spaceAround: false,
  spaceBetween: false,
})

const emit = defineEmits<{
  (event: 'update:value', payload: string): void
}>()
</script>

<style lang="scss" scoped>
.tabs {
  display: flex;
  user-select: none;
  line-height: 1;

  &:not(.card) {
    font-size: 13px;
    align-items: center;
    justify-content: flex-start;
    border-bottom: 1px solid $borderColor;

    &.space-around {
      justify-content: space-around;
    }
    &.space-between {
      justify-content: space-between;
    }

    .tab {
      text-align: center;
      border-bottom: 2px solid transparent;
      padding: 8px 10px;
      cursor: pointer;

      &.active {
        border-bottom: 2px solid var(--color, $themeColor);
      }
    }
  }

  &.card {
    height: 32px;
    font-size: 13px;
    flex-shrink: 0;
    background-color: #f0f0f0;
    border-radius: 4px;
    padding: 2px;
    gap: 2px;

    .tab {
      flex: 1;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: transparent;
      border-radius: 3px;
      cursor: pointer;
      color: #666;
      font-weight: 400;
      transition: all 0.2s ease;

      &:hover {
        color: #333;
      }

      &.active {
        background-color: #fff;
        color: $themeColor;
        font-weight: 500;
        box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
      }
    }
  }
}
</style>