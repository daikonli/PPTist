<template>
  <div class="canvas-tool">
    <div class="left-handler">
      <IconBack class="handler-item" :class="{ 'disable': !canUndo }" v-tooltip="'撤销（Ctrl + Z）'" @click="undo()" />
      <IconNext class="handler-item" :class="{ 'disable': !canRedo }" v-tooltip="'重做（Ctrl + Y）'" @click="redo()" />
      <div class="more">
        <Divider type="vertical" style="height: 20px;" />
        <Popover class="more-icon" trigger="click" v-model:value="moreVisible" :offset="10">
          <template #content>
            <PopoverMenuItem class="popover-menu-item" center @click="toggleSelectPanel(); moreVisible = false"><IconMoveOne class="icon" />选择窗格</PopoverMenuItem>
          </template>
          <IconMore class="handler-item" />
        </Popover>
        <IconMoveOne class="handler-item" :class="{ 'active': showSelectPanel }" v-tooltip="'选择窗格'" @click="toggleSelectPanel()" />
      </div>
    </div>

    <div class="add-element-handler">
      <div class="insert-handler-item group-btn" :class="{ 'active': creatingElement?.type === 'text' }" v-tooltip="'插入文字'">
        <div class="group-btn-main" @click="drawText()"><IconFontSize class="icon" /> <span class="text">文本框</span></div>
        
        <Popover trigger="click" v-model:value="textTypeSelectVisible" style="height: 100%;" :offset="10">
          <template #content>
            <PopoverMenuItem center @click="() => { drawText(); textTypeSelectVisible = false }"><IconTextRotationNone class="icon" /> 横向文本框</PopoverMenuItem>
            <PopoverMenuItem center @click="() => { drawText(true); textTypeSelectVisible = false }"><IconTextRotationDown class="icon" /> 竖向文本框</PopoverMenuItem>
          </template>
          <IconDown class="arrow" />
        </Popover>
      </div>
      <div class="insert-handler-item group-btn" :class="{ 'active': creatingCustomShape || creatingElement?.type === 'shape' || creatingElement?.type === 'line' }" v-tooltip="'插入形状'" :offset="10">
        <Popover trigger="click" style="height: 100%;" v-model:value="shapePoolVisible" :offset="10">
          <template #content>
            <ShapePool @select="shape => drawShape(shape)" @selectLine="line => drawLine(line)" />
          </template>
          <div class="group-btn-main"><IconGraphicDesign class="icon" /> <span class="text">形状</span></div>
        </Popover>
        
        <Popover trigger="click" v-model:value="shapeMenuVisible" style="height: 100%;" :offset="10">
          <template #content>
            <PopoverMenuItem center @click="shapeMenuVisible = false; shapePoolVisible = true"><IconGraphicDesign class="icon" />预设形状</PopoverMenuItem>
            <PopoverMenuItem center @click="() => { drawCustomShape(); shapeMenuVisible = false }"><IconWritingFluently class="icon" />自由绘制</PopoverMenuItem>
          </template>
          <IconDown class="arrow" />
        </Popover>
      </div>
      <div class="insert-handler-item group-btn" v-tooltip="'插入图片'">
        <FileInput style="height: 100%;" @change="files => insertImageElement(files)">
          <div class="group-btn-main"><IconPicture class="icon" /> <span class="text">图片</span></div>
        </FileInput>
        
        <Popover trigger="click" v-model:value="imageMenuVisible" style="height: 100%;" :offset="10">
          <template #content>
            <FileInput @change="files => { insertImageElement(files); imageMenuVisible = false }">
              <PopoverMenuItem center><IconUpload class="icon" /> 上传图片</PopoverMenuItem>
            </FileInput>
            <PopoverMenuItem center @click="openImageLibPanel(); imageMenuVisible = false"><IconPicture class="icon" /> 在线图库</PopoverMenuItem>
          </template>
          <IconDown class="arrow" />
        </Popover>
      </div>
      <Popover trigger="click" v-model:value="chartPoolVisible" :offset="10">
        <template #content>
          <ChartPool @select="chart => { createChartElement(chart); chartPoolVisible = false }" />
        </template>
        <div class="insert-handler-item" v-tooltip="'插入图表'">
          <IconChartProportion class="icon" /> <span class="text">图表</span>
        </div>
      </Popover>
      <Popover trigger="click" v-model:value="tableGeneratorVisible" :offset="10">
        <template #content>
          <TableGenerator
            @close="tableGeneratorVisible = false"
            @insert="({ row, col }) => { createTableElement(row, col); tableGeneratorVisible = false }"
          />
        </template>
        <div class="insert-handler-item" v-tooltip="'插入表格'">
          <IconInsertTable class="icon" /> <span class="text">表格</span>
        </div>
      </Popover>
      <div class="insert-handler-item" v-tooltip="'插入公式'" @click="latexEditorVisible = true">
        <IconFormula class="icon" /> <span class="text">公式</span>
      </div>
      <Popover trigger="click" v-model:value="mediaInputVisible" :offset="10">
        <template #content>
          <MediaInput 
            @close="mediaInputVisible = false"
            @insertVideo="({ src, ext }) => { createVideoElement(src, ext); mediaInputVisible = false }"
            @insertAudio="({ src, ext }) => { createAudioElement(src, ext); mediaInputVisible = false }"
          />
        </template>
        <div class="insert-handler-item" v-tooltip="'插入音视频'">
          <IconVideoTwo class="icon" /> <span class="text">音视频</span>
        </div>
      </Popover>
      <div class="insert-handler-item" :class="{ 'active': showSymbolPanel }" v-tooltip="'插入符号'" @click="toggleSymbolPanel()">
        <IconSymbol class="icon" /> <span class="text">符号</span>
      </div>
      <div class="toolbar-divider"></div>
      <div class="insert-handler-item" :class="{ 'active': isFormatActive }" v-tooltip="'格式'" @click="openDesignPanel()">
        <IconTheme class="icon" /> <span class="text">格式</span>
      </div>
      <div class="insert-handler-item" :class="{ 'active': showToolbar && toolbarState === ToolbarStates.EL_ANIMATION }" v-tooltip="'动画'" @click="openAnimationPanel()">
        <IconEffects class="icon" /> <span class="text">动画</span>
      </div>
      <div class="toolbar-divider"></div>
      <div class="insert-handler-item" :class="{ 'active': showNotesPanel }" v-tooltip="'评论'" @click="toggleNotesPanel()">
        <IconComment class="icon" /> <span class="text">评论</span>
      </div>
    </div>

    <div class="right-handler">
      <IconMinus class="handler-item viewport-size" v-tooltip="'画布缩小（Ctrl + -）'" @click="scaleCanvas('-')" />
      <Popover trigger="click" v-model:value="canvasScaleVisible">
        <template #content>
          <PopoverMenuItem
            center
            v-for="item in canvasScalePresetList" 
            :key="item" 
            @click="applyCanvasPresetScale(item)"
          >{{item}}%</PopoverMenuItem>
          <PopoverMenuItem center @click="resetCanvas(); canvasScaleVisible = false">适应屏幕</PopoverMenuItem>
        </template>
        <span class="text">{{ canvasScalePercentage }}</span>
      </Popover>
      <IconPlus class="handler-item viewport-size" v-tooltip="'画布放大（Ctrl + =）'" @click="scaleCanvas('+')" />
      <IconFullScreen class="handler-item viewport-size-adaptation" v-tooltip="'适应屏幕（Ctrl + 0）'" @click="resetCanvas()" />
    </div>

    <Modal
      v-model:visible="latexEditorVisible" 
      :width="880"
    >
      <LaTeXEditor 
        @close="latexEditorVisible = false"
        @update="data => { createLatexElement(data); latexEditorVisible = false }"
      />
    </Modal>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore, useSnapshotStore } from '@/store'
import { getImageDataURL } from '@/utils/image'
import type { ShapePoolItem } from '@/configs/shapes'
import type { LinePoolItem } from '@/configs/lines'
import { ToolbarStates } from '@/types/toolbar'
import useScaleCanvas from '@/hooks/useScaleCanvas'
import useHistorySnapshot from '@/hooks/useHistorySnapshot'
import useCreateElement from '@/hooks/useCreateElement'

import ShapePool from './ShapePool.vue'
import ChartPool from './ChartPool.vue'
import TableGenerator from './TableGenerator.vue'
import MediaInput from './MediaInput.vue'
import LaTeXEditor from '@/components/LaTeXEditor/index.vue'
import FileInput from '@/components/FileInput.vue'
import Modal from '@/components/Modal.vue'
import Divider from '@/components/Divider.vue'
import Popover from '@/components/Popover.vue'
import PopoverMenuItem from '@/components/PopoverMenuItem.vue'

const mainStore = useMainStore()
const { creatingElement, creatingCustomShape, showSelectPanel, showSearchPanel, showNotesPanel, showSymbolPanel, toolbarState, showToolbar, activeElementIdList } = storeToRefs(mainStore)
const { canUndo, canRedo } = storeToRefs(useSnapshotStore())

// 格式按钮激活状态
const isFormatActive = computed(() => {
  return showToolbar.value && (
    toolbarState.value === ToolbarStates.SLIDE_DESIGN ||
    toolbarState.value === ToolbarStates.EL_STYLE ||
    toolbarState.value === ToolbarStates.EL_POSITION ||
    toolbarState.value === ToolbarStates.MULTI_STYLE ||
    toolbarState.value === ToolbarStates.MULTI_POSITION
  )
})

const { redo, undo } = useHistorySnapshot()

const {
  scaleCanvas,
  setCanvasScalePercentage,
  resetCanvas,
  canvasScalePercentage,
} = useScaleCanvas()

const canvasScalePresetList = [200, 150, 125, 100, 75, 50]
const canvasScaleVisible = ref(false)

const applyCanvasPresetScale = (value: number) => {
  setCanvasScalePercentage(value)
  canvasScaleVisible.value = false
}

const {
  createImageElement,
  createChartElement,
  createTableElement,
  createLatexElement,
  createVideoElement,
  createAudioElement,
} = useCreateElement()

const insertImageElement = (files: FileList) => {
  const imageFile = files[0]
  if (!imageFile) return
  getImageDataURL(imageFile).then(dataURL => createImageElement(dataURL))
}

const shapePoolVisible = ref(false)
const chartPoolVisible = ref(false)
const tableGeneratorVisible = ref(false)
const mediaInputVisible = ref(false)
const latexEditorVisible = ref(false)
const textTypeSelectVisible = ref(false)
const shapeMenuVisible = ref(false)
const imageMenuVisible = ref(false)
const moreVisible = ref(false)

// 绘制文字范围
const drawText = (vertical = false) => {
  mainStore.setCreatingElement({
    type: 'text',
    vertical,
  })
}

// 绘制形状范围
const drawShape = (shape: ShapePoolItem) => {
  mainStore.setCreatingElement({
    type: 'shape',
    data: shape,
  })
  shapePoolVisible.value = false
}
// 绘制自定义任意多边形
const drawCustomShape = () => {
  mainStore.setCreatingCustomShapeState(true)
  shapePoolVisible.value = false
}

// 绘制线条路径
const drawLine = (line: LinePoolItem) => {
  mainStore.setCreatingElement({
    type: 'line',
    data: line,
  })
  shapePoolVisible.value = false
}

// 打开选择面板
const toggleSelectPanel = () => {
  mainStore.setSelectPanelState(!showSelectPanel.value)
}

// 打开搜索替换面板
const toggleSraechPanel = () => {
  mainStore.setSearchPanelState(!showSearchPanel.value)
}

// 打开评论面板
const toggleNotesPanel = () => {
  mainStore.setNotesPanelState(!showNotesPanel.value)
}

// 打开符号面板
const toggleSymbolPanel = () => {
  mainStore.setSymbolPanelState(!showSymbolPanel.value)
}

// 切换设计面板
const openDesignPanel = () => {
  // 如果格式按钮已经激活，则关闭工具栏
  if (isFormatActive.value) {
    mainStore.setToolbarVisibility(false)
  }
  else {
    // 否则打开工具栏并根据是否有选中元素显示对应面板
    // 如果有选中元素，显示元素样式面板；否则显示幻灯片设计面板
    if (activeElementIdList.value.length > 0) {
      mainStore.setToolbarState(activeElementIdList.value.length === 1 ? ToolbarStates.EL_STYLE : ToolbarStates.MULTI_STYLE)
    }
    else {
      mainStore.setToolbarState(ToolbarStates.SLIDE_DESIGN)
    }
    mainStore.setToolbarVisibility(true)
  }
}

// 切换动画面板
const openAnimationPanel = () => {
  // 如果动画面板已经打开，则关闭工具栏
  if (showToolbar.value && toolbarState.value === ToolbarStates.EL_ANIMATION) {
    mainStore.setToolbarVisibility(false)
  }
  else {
    // 否则打开工具栏并显示动画面板
    mainStore.setToolbarState(ToolbarStates.EL_ANIMATION)
    mainStore.setToolbarVisibility(true)
  }
}

// 打开图库面板
const openImageLibPanel = () => {
  mainStore.setImageLibPanelState(true)
}
</script>

<style lang="scss" scoped>
.canvas-tool {
  position: relative;
  border-bottom: 1px solid $borderColor;
  background-color: $lightGray;
  display: flex;
  justify-content: space-between;
  padding: 0 10px;
  font-size: 13px;
  user-select: none;
}
.left-handler, .more {
  display: flex;
  align-items: center;
}
.more-icon {
  display: none;
}
.popover-menu-item {
  display: flex;
  padding: 8px 10px;

  &.center {
    justify-content: center;
  }

  .icon {
    font-size: 18px;
    margin-right: 8px;
  }
}
.add-element-handler {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  align-items: center;

  & > div {
    flex-shrink: 0;
  }

  .insert-handler-item {
    height: 30px;
    font-size: 14px;
    margin: 0 2px;
    padding: 0 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: $borderRadius;
    overflow: hidden;
    cursor: pointer;

    &:not(.group-btn):hover {
      background-color: #f1f1f1;
    }

    &.active {
      background-color: #dedef4;
      
      .icon {
        color: #5337a6;
      }
      
      .text {
        color: #5337a6;
      }
    }

    .icon {
      margin-right: 4px;
    }

    &.group-btn {
      margin-right: 6px;
      padding: 0;

      &:hover {
        background-color: #f3f3f3;
      }

      .group-btn-main {
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 0 5px;

        &:hover {
          background-color: #e9e9e9;
        }
      }

      .arrow {
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 12px;
        padding: 0 1px;
  
        &:hover {
          background-color: #e9e9e9;
        }
      }
    }
  }
}
.toolbar-divider {
  width: 1px;
  height: 20px;
  background-color: #e0e0e0;
  margin: 0 8px;
  flex-shrink: 0;
}
.handler-item {
  height: 30px;
  font-size: 14px;
  margin: 0 2px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: $borderRadius;
  overflow: hidden;
  cursor: pointer;

  &.disable {
    opacity: .5;
  }
}
.left-handler, .right-handler {
  .handler-item {
    padding: 0 8px;

    &.active,
    &:not(.disable):hover {
      background-color: #f1f1f1;
    }
  }
}
.right-handler {
  display: flex;
  align-items: center;

  .text {
    display: inline-block;
    width: 40px;
    text-align: center;
    cursor: pointer;
  }

  .viewport-size {
    font-size: 13px;
  }
}

@media screen and (width <= 1600px) {
  .add-element-handler {
    .insert-handler-item {
      .icon {
        margin-right: 0;
      }
      .text {
        display: none;
      }
    }
    
    .toolbar-divider {
      margin: 0 4px;
    }
  }
}
@media screen and (width <= 1366px) {
  .add-element-handler {
    .insert-handler-item {
      padding: 0 6px;
    }
    
    .toolbar-divider {
      height: 16px;
      margin: 0 2px;
    }
  }
}
@media screen and (width <= 1200px) {
  .right-handler .text {
    display: none;
  }
  .more > .handler-item {
    display: none;
  }
  .more-icon {
    display: block;
  }
}
@media screen and (width <= 1000px) {
  .left-handler, .right-handler {
    display: none;
  }
}
</style>