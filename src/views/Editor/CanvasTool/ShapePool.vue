<template>
  <div class="shape-pool">
    <div class="category" v-for="item in SHAPE_LIST" :key="item.type">
      <div class="category-name">{{item.type}}</div>
      <div class="shape-list">
        <ShapeItemThumbnail 
          class="shape-item"
          v-for="(shape, index) in item.children" 
          :key="index" 
          :shape="shape" 
          @click="selectShape(shape)"
        />
      </div>
    </div>

    <!-- 线条部分 -->
    <div class="category" v-for="(item, i) in LINE_LIST" :key="`line-${item.type}`">
      <div class="category-name">{{item.type}}</div>
      <div class="line-list">
        <div class="line-item" v-for="(line, j) in item.children" :key="j">
          <div class="line-content" @click="selectLine(line)">
            <svg
              overflow="visible" 
              width="20"
              height="20"
            >
              <defs>
                <LinePointMarker
                  class="line-marker"
                  v-if="line.points[0]"
                  :id="`preset-line-${i}-${j}`"
                  position="start"
                  :type="line.points[0]"
                  color="currentColor"
                  :baseSize="2"
                />
                <LinePointMarker
                  class="line-marker"
                  v-if="line.points[1]"
                  :id="`preset-line-${i}-${j}`"
                  position="end"
                  :type="line.points[1]"
                  color="currentColor"
                  :baseSize="2"
                />
              </defs>
              <path
                class="line-path"
                :d="line.path" 
                stroke="currentColor" 
                fill="none" 
                stroke-width="2" 
                :stroke-dasharray="line.style === 'solid' ? '0, 0' : '4, 1'"
                :marker-start="line.points[0] ? `url(#${`preset-line-${i}-${j}`}-${line.points[0]}-start)` : ''"
                :marker-end="line.points[1] ? `url(#${`preset-line-${i}-${j}`}-${line.points[1]}-end)` : ''"
              ></path>
            </svg>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { SHAPE_LIST, type ShapePoolItem } from '@/configs/shapes'
import { LINE_LIST, type LinePoolItem } from '@/configs/lines'
import ShapeItemThumbnail from './ShapeItemThumbnail.vue'
import LinePointMarker from '@/views/components/element/LineElement/LinePointMarker.vue'

const emit = defineEmits<{
  (event: 'select', payload: ShapePoolItem): void
  (event: 'selectLine', payload: LinePoolItem): void
}>()

const selectShape = (shape: ShapePoolItem) => {
  emit('select', shape)
}

const selectLine = (line: LinePoolItem) => {
  emit('selectLine', line)
}
</script>

<style lang="scss" scoped>
.shape-pool {
  width: 340px;
  max-height: 520px;
  overflow: auto;
  margin-top: -8px;
  margin-bottom: -8px;
  margin-right: -10px;
  padding-right: 10px;
  padding-top: 10px;
}
.category-name {
  width: 100%;
  font-size: 12px;
  margin-bottom: 10px;
  border-left: 4px solid #bbb;
  background-color: #f1f1f1;
  padding: 3px 0 3px 8px;
  color: #555;
}
.shape-list {
  @include flex-grid-layout();

  margin-bottom: 10px;
}
.shape-item {
  @include flex-grid-layout-children(10, 8%);

  height: 0;
  padding-bottom: 8%;
  flex-shrink: 0;
}

// 线条样式
.line-list {
  @include flex-grid-layout();

  margin-bottom: 10px;
}
.line-item {
  @include flex-grid-layout-children(10, 8%);

  height: 0;
  padding-bottom: 8%;
  flex-shrink: 0;
  position: relative;
  cursor: pointer;
}
.line-content {
  @include absolute-0();

  display: flex;
  justify-content: center;
  align-items: center;
  color: #999;

  &:hover {
    color: $themeColor;
  }

  svg:not(:root) {
    overflow: visible;
  }
}
</style>