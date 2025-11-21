<template>
  <div class="animation-panel">
    <div class="tabs-wrapper">
    <Tabs 
      :tabs="animationTabs" 
      :value="activeAnimationTab" 
      card 
      @update:value="key => setActiveAnimationTab(key as AnimationTabType)"
    />
    </div>
    <div class="content">
      <component :is="currentAnimationComponent"></component>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'
import ElementAnimationPanel from './ElementAnimationPanel.vue'
import SlideAnimationPanel from './SlideAnimationPanel.vue'
import Tabs from '@/components/Tabs.vue'

type AnimationTabType = 'element' | 'slide'

const animationTabs = [
  { label: '元素动画', key: 'element' },
  { label: '页面切换', key: 'slide' },
]

const activeAnimationTab = ref<AnimationTabType>('element')

const setActiveAnimationTab = (key: AnimationTabType) => {
  activeAnimationTab.value = key
}

const currentAnimationComponent = computed(() => {
  return activeAnimationTab.value === 'element' ? ElementAnimationPanel : SlideAnimationPanel
})
</script>

<style lang="scss" scoped>
.animation-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
  margin: -12px;
}
.tabs-wrapper {
  flex-shrink: 0;
  padding: 0 12px;
}
.content {
  flex: 1;
  overflow: auto;
  padding: 12px;
  
  @include overflow-overlay();
}
</style>

