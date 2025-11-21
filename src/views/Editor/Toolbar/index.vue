<template>
  <div class="toolbar">
    <div class="toolbar-header">
      <div class="header-title">
        <component :is="currentIcon" class="title-icon" />
        <span class="title-text">{{ currentTitle }}</span>
      </div>
      <div class="close-btn" v-tooltip="'关闭侧边栏'" @click="closeToolbar">
        <IconClose />
      </div>
    </div>
    <div class="tabs-wrapper" v-if="currentTabs.length > 0">
      <Tabs 
        :tabs="currentTabs" 
        :value="toolbarState" 
        card 
        @update:value="key => setToolbarState(key as ToolbarStates)"
      />
    </div>
    <div class="content">
      <component :is="currentPanelComponent"></component>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, watch } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore } from '@/store'
import { ToolbarStates } from '@/types/toolbar'

import ElementStylePanel from './ElementStylePanel/index.vue'
import ElementPositionPanel from './ElementPositionPanel.vue'
import AnimationPanel from './AnimationPanel.vue'
import SlideDesignPanel from './SlideDesignPanel/index.vue'
import MultiPositionPanel from './MultiPositionPanel.vue'
import MultiStylePanel from './MultiStylePanel.vue'
import NotesPanel from './NotesPanel.vue'
import AIChatPanel from './AIChatPanel.vue'
import Tabs from '@/components/Tabs.vue'

const mainStore = useMainStore()
const { activeElementIdList, activeElementList, activeGroupElementId, toolbarState } = storeToRefs(mainStore)

const elementTabs = [
  { label: '样式', key: ToolbarStates.EL_STYLE },
  { label: '位置', key: ToolbarStates.EL_POSITION },
]
const multiSelectTabs = [
  { label: '样式（多选）', key: ToolbarStates.MULTI_STYLE },
  { label: '位置（多选）', key: ToolbarStates.MULTI_POSITION },
]

const setToolbarState = (value: ToolbarStates) => {
  mainStore.setToolbarState(value)
}

const closeToolbar = () => {
  mainStore.setToolbarVisibility(false)
}

const currentTabs = computed(() => {
  // 如果是动画面板、评论面板或 AI 聊天面板，不显示任何tab
  if (toolbarState.value === ToolbarStates.EL_ANIMATION || 
      toolbarState.value === ToolbarStates.SLIDE_ANIMATION ||
      toolbarState.value === ToolbarStates.NOTES ||
      toolbarState.value === ToolbarStates.AI_CHAT) {
    return []
  }
  
  if (!activeElementIdList.value.length) return []
  else if (activeElementIdList.value.length > 1) {
    if (!activeGroupElementId.value) return multiSelectTabs

    const activeGroupElement = activeElementList.value.find(item => item.id === activeGroupElementId.value)
    if (activeGroupElement) return elementTabs
    return multiSelectTabs
  }
  return elementTabs
})

watch(currentTabs, () => {
  const currentTabsValue: ToolbarStates[] = currentTabs.value.map(tab => tab.key)
  if (currentTabsValue.length > 0 && !currentTabsValue.includes(toolbarState.value)) {
    mainStore.setToolbarState(currentTabsValue[0])
  }
})

// 监听元素选中状态变化，未选中元素时切换到幻灯片设计面板
watch(activeElementIdList, (newVal, oldVal) => {
  // 当从有选中元素变为无选中元素时
  if (oldVal && oldVal.length > 0 && newVal.length === 0) {
    // 如果当前工具栏是元素相关的面板，则切换到幻灯片设计面板
    if (toolbarState.value === ToolbarStates.EL_STYLE || 
        toolbarState.value === ToolbarStates.EL_POSITION ||
        toolbarState.value === ToolbarStates.EL_ANIMATION ||
        toolbarState.value === ToolbarStates.MULTI_STYLE ||
        toolbarState.value === ToolbarStates.MULTI_POSITION) {
      mainStore.setToolbarState(ToolbarStates.SLIDE_DESIGN)
    }
  }
})

const currentPanelComponent = computed(() => {
  const panelMap = {
    [ToolbarStates.EL_STYLE]: ElementStylePanel,
    [ToolbarStates.EL_POSITION]: ElementPositionPanel,
    [ToolbarStates.EL_ANIMATION]: AnimationPanel,
    [ToolbarStates.SLIDE_DESIGN]: SlideDesignPanel,
    [ToolbarStates.SLIDE_ANIMATION]: AnimationPanel,
    [ToolbarStates.MULTI_STYLE]: MultiStylePanel,
    [ToolbarStates.MULTI_POSITION]: MultiPositionPanel,
    [ToolbarStates.NOTES]: NotesPanel,
    [ToolbarStates.AI_CHAT]: AIChatPanel,
  }
  return panelMap[toolbarState.value] || null
})

const currentTitle = computed(() => {
  const titleMap = {
    [ToolbarStates.EL_STYLE]: '格式',
    [ToolbarStates.EL_POSITION]: '格式',
    [ToolbarStates.EL_ANIMATION]: '动画',
    [ToolbarStates.SLIDE_DESIGN]: '格式',
    [ToolbarStates.SLIDE_ANIMATION]: '动画',
    [ToolbarStates.MULTI_STYLE]: '格式',
    [ToolbarStates.MULTI_POSITION]: '格式',
    [ToolbarStates.NOTES]: '评论',
    [ToolbarStates.AI_CHAT]: 'AI 助手',
  }
  return titleMap[toolbarState.value] || ''
})

const currentIcon = computed(() => {
  const iconMap = {
    [ToolbarStates.EL_STYLE]: 'IconTheme',
    [ToolbarStates.EL_POSITION]: 'IconFullSelection',
    [ToolbarStates.EL_ANIMATION]: 'IconEffects',
    [ToolbarStates.SLIDE_DESIGN]: 'IconTheme',
    [ToolbarStates.SLIDE_ANIMATION]: 'IconEffects',
    [ToolbarStates.MULTI_STYLE]: 'IconTheme',
    [ToolbarStates.MULTI_POSITION]: 'IconFullSelection',
    [ToolbarStates.NOTES]: 'IconComment',
    [ToolbarStates.AI_CHAT]: 'IconMagic',
  }
  return iconMap[toolbarState.value] || 'IconTheme'
})
</script>

<style lang="scss" scoped>
.toolbar {
  border-left: solid 1px $borderColor;
  background-color: #fff;
  display: flex;
  flex-direction: column;
}
.toolbar-header {
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  flex-shrink: 0;

  .header-title {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 15px;
    font-weight: 500;
    color: #333;

    .title-icon {
      font-size: 18px;
      color: #666;
    }
  }
  
  .close-btn {
    width: 24px;
    height: 24px;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: $borderRadius;
    cursor: pointer;
    font-size: 14px;
    color: #666;

    &:hover {
      background-color: #f1f1f1;
      color: $themeColor;
    }
  }
}
.tabs-wrapper {
  flex-shrink: 0;
  padding: 0 12px;
}
.content {
  padding: 12px;
  font-size: 13px;
  flex: 1;
  overflow: auto;

  @include overflow-overlay();
}
</style>