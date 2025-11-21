<template>
  <div class="pptist-editor">
    <EditorHeader class="layout-header" />
    <div class="layout-content">
      <Thumbnails v-if="showThumbnails" class="layout-content-left" />
      <div 
        class="layout-content-center" 
        :class="{ 
          'full-width': !showToolbar && showThumbnails,
          'full-width-no-thumbnails': !showThumbnails && showToolbar,
          'full-width-both': !showToolbar && !showThumbnails
        }"
      >
        <div v-if="!showThumbnails" class="expand-thumbnails-btn" @click="mainStore.setThumbnailsVisibility(true)" title="展开缩略图">
          <IconRight />
        </div>
        <CanvasTool class="center-top" />
        <Canvas class="center-body" :style="{ height: `calc(100% - ${remarkHeight + 40}px)` }" />
        <Remark
          class="center-bottom" 
          v-model:height="remarkHeight" 
          :style="{ height: `${remarkHeight}px` }"
        />
      </div>
      <Toolbar v-if="showToolbar" class="layout-content-right" />
    </div>
  </div>

  <SelectPanel v-if="showSelectPanel" />
  <SearchPanel v-if="showSearchPanel" />
  <NotesPanel v-if="showNotesPanel" />
  <MarkupPanel v-if="showMarkupPanel" />
  <SymbolPanel v-if="showSymbolPanel" />
  <ImageLibPanel v-if="showImageLibPanel" />

  <Modal
    :visible="!!dialogForExport" 
    :width="680"
    @closed="closeExportDialog()"
  >
    <ExportDialog />
  </Modal>

  <Modal
    :visible="showAIPPTDialog" 
    :width="720"
    :closeOnClickMask="false"
    :closeOnEsc="false"
    closeButton
    @closed="closeAIPPTDialog()"
  >
    <AIPPTDialog />
  </Modal>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore } from '@/store'
import useGlobalHotkey from '@/hooks/useGlobalHotkey'
import usePasteEvent from '@/hooks/usePasteEvent'

import EditorHeader from './EditorHeader/index.vue'
import Canvas from './Canvas/index.vue'
import CanvasTool from './CanvasTool/index.vue'
import Thumbnails from './Thumbnails/index.vue'
import Toolbar from './Toolbar/index.vue'
import Remark from './Remark/index.vue'
import ExportDialog from './ExportDialog/index.vue'
import SelectPanel from './SelectPanel.vue'
import SearchPanel from './SearchPanel.vue'
import NotesPanel from './NotesPanel.vue'
import SymbolPanel from './SymbolPanel.vue'
import MarkupPanel from './MarkupPanel.vue'
import ImageLibPanel from './ImageLibPanel.vue'
import AIPPTDialog from './AIPPTDialog.vue'
import Modal from '@/components/Modal.vue'

const mainStore = useMainStore()
const {
  dialogForExport,
  showSelectPanel,
  showSearchPanel,
  showNotesPanel,
  showSymbolPanel,
  showMarkupPanel,
  showImageLibPanel,
  showAIPPTDialog,
  showToolbar,
  showThumbnails,
} = storeToRefs(mainStore)

const closeExportDialog = () => mainStore.setDialogForExport('')
const closeAIPPTDialog = () => mainStore.setAIPPTDialogState(false)

const remarkHeight = ref(40)

useGlobalHotkey()
usePasteEvent()
</script>

<style lang="scss" scoped>
.pptist-editor {
  height: 100%;
}
.layout-header {
  height: 40px;
}
.layout-content {
  height: calc(100% - 40px);
  display: flex;
}
.layout-content-left {
  width: 196px;
  height: 100%;
  flex-shrink: 0;
  transition: width 0.3s ease;
}
.layout-content-center {
  width: calc(100% - 160px - 260px);
  transition: width 0.3s ease;
  position: relative;

  &.full-width {
    width: calc(100% - 160px);
  }
  &.full-width-no-thumbnails {
    width: calc(100% - 260px);
  }
  &.full-width-both {
    width: 100%;
  }

  .center-top {
    height: 50px;
  }

  .expand-thumbnails-btn {
    position: absolute;
    left: 10px;
    top: 10px;
    width: 32px;
    height: 32px;
    background-color: #fff;
    border: 1px solid $borderColor;
    border-radius: $borderRadius;
    display: flex;
    justify-content: center;
    align-items: center;
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
.layout-content-right {
  width: 260px;
  height: 100%;
}
</style>