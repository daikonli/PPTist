<template>
  <div class="pptist-editor">
    <!-- 顶部工具栏 -->
    <EditorHeader class="layout-header" />
    
    <!-- 主内容区域 -->
    <div class="layout-content">
      <!-- 左侧缩略图面板 -->
      <Thumbnails v-if="showThumbnails" class="layout-content-left" />
      
      <!-- 中间画布区域 -->
      <div class="layout-content-center" :class="centerContentClass">
        <!-- 展开缩略图按钮 -->
        <div 
          v-if="!showThumbnails" 
          class="expand-thumbnails-btn" 
          title="展开缩略图"
          @click="handleExpandThumbnails"
        >
          <IconRight />
        </div>
        
        <!-- 画布工具栏 -->
        <CanvasTool class="center-top" />
        
        <!-- 画布主体 -->
        <Canvas class="center-body" :style="canvasBodyStyle" />
        
        <!-- 备注区域 -->
        <Remark
          v-model:height="remarkHeight" 
          class="center-bottom" 
          :style="remarkStyle"
        />
      </div>
      
      <!-- 右侧工具栏 -->
      <Toolbar v-if="showToolbar" class="layout-content-right" />
    </div>
  </div>

  <!-- 各种弹出面板 -->
  <SelectPanel v-if="showSelectPanel" />
  <SearchPanel v-if="showSearchPanel" />
  <MarkupPanel v-if="showMarkupPanel" />
  <SymbolPanel v-if="showSymbolPanel" />
  <ImageLibPanel v-if="showImageLibPanel" />

  <!-- 导出对话框 -->
  <Modal
    :visible="!!dialogForExport" 
    :width="680"
    @closed="handleCloseExportDialog"
  >
    <ExportDialog />
  </Modal>

  <!-- AI PPT 对话框 -->
  <Modal
    :visible="showAIPPTDialog" 
    :width="720"
    :closeOnClickMask="false"
    :closeOnEsc="false"
    closeButton
    @closed="handleCloseAIPPTDialog"
  >
    <AIPPTDialog />
  </Modal>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore } from '@/store'
import useGlobalHotkey from '@/hooks/useGlobalHotkey'
import usePasteEvent from '@/hooks/usePasteEvent'

// 组件导入
import EditorHeader from './EditorHeader/index.vue'
import Canvas from './Canvas/index.vue'
import CanvasTool from './CanvasTool/index.vue'
import Thumbnails from './Thumbnails/index.vue'
import Toolbar from './Toolbar/index.vue'
import Remark from './Remark/index.vue'
import ExportDialog from './ExportDialog/index.vue'
import SelectPanel from './SelectPanel.vue'
import SearchPanel from './SearchPanel.vue'
import SymbolPanel from './SymbolPanel.vue'
import MarkupPanel from './MarkupPanel.vue'
import ImageLibPanel from './ImageLibPanel.vue'
import AIPPTDialog from './AIPPTDialog.vue'
import Modal from '@/components/Modal.vue'

// Store
const mainStore = useMainStore()
const {
  dialogForExport,
  showSelectPanel,
  showSearchPanel,
  showSymbolPanel,
  showMarkupPanel,
  showImageLibPanel,
  showAIPPTDialog,
  showToolbar,
  showThumbnails,
} = storeToRefs(mainStore)

// 响应式数据
const remarkHeight = ref(40)

// 计算属性
const centerContentClass = computed(() => ({
  'full-width': !showToolbar.value && showThumbnails.value,
  'full-width-no-thumbnails': !showThumbnails.value && showToolbar.value,
  'full-width-both': !showToolbar.value && !showThumbnails.value,
}))

const canvasBodyStyle = computed(() => ({
  height: `calc(100% - ${remarkHeight.value + 40}px)`,
}))

const remarkStyle = computed(() => ({
  height: `${remarkHeight.value}px`,
}))

// 方法
const handleExpandThumbnails = () => {
  mainStore.setThumbnailsVisibility(true)
}

const handleCloseExportDialog = () => {
  mainStore.setDialogForExport('')
}

const handleCloseAIPPTDialog = () => {
  mainStore.setAIPPTDialogState(false)
}

// 初始化钩子
useGlobalHotkey()
usePasteEvent()
</script>

<style lang="scss" scoped>
// 布局常量
$header-height: 64px;
$thumbnails-width: 196px;
$toolbar-width: 294px;
$canvas-tool-height: 52px;

// 主容器
.pptist-editor {
  height: 100%;
}

// 顶部工具栏
.layout-header {
  height: $header-height;
}

// 主内容区域
.layout-content {
  height: calc(100% - $header-height);
  display: flex;
}

// 左侧缩略图面板
.layout-content-left {
  width: $thumbnails-width;
  height: 100%;
  flex-shrink: 0;
  transition: width 0.3s ease;
}

// 中间画布区域
.layout-content-center {
  width: calc(100% - 160px - $toolbar-width);
  height: 100%;
  position: relative;
  transition: width 0.3s ease;

  // 只隐藏工具栏
  &.full-width {
    width: calc(100% - 160px);
  }

  // 只隐藏缩略图
  &.full-width-no-thumbnails {
    width: calc(100% - $toolbar-width);
  }

  // 隐藏缩略图和工具栏
  &.full-width-both {
    width: 100%;
  }

  // 画布工具栏
  .center-top {
    height: $canvas-tool-height;
  }

  // 展开缩略图按钮
  .expand-thumbnails-btn {
    position: absolute;
    left: 10px;
    top: 10px;
    width: 32px;
    height: 32px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #fff;
    border: 1px solid $borderColor;
    border-radius: $borderRadius;
    cursor: pointer;
    z-index: 10;
    transition: all 0.2s;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

    &:hover {
      background-color: $lightGray;
      box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
    }
  }
}

// 右侧工具栏
.layout-content-right {
  width: $toolbar-width;
  height: 100%;
}
</style>