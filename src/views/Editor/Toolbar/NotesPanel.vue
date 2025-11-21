<template>
  <div class="notes-panel">
    <div class="container">
      <div class="notes" ref="notesRef">
        <div class="note" :class="{ 'active': activeNoteId === note.id }" v-for="note in notes" :key="note.id" @click="handleClickNote(note)">
          <div class="header note-header">
            <div class="user">
              <div class="avatar"><IconUser /></div>
              <div class="user-info">
                <div class="username">{{ note.user }}</div>
                <div class="time">{{ new Date(note.time).toLocaleString() }}</div>
              </div>
            </div>
            <div class="btns">
              <div class="btn reply" @click="replyNoteId = note.id">回复</div>
              <div class="btn delete" @click.stop="deleteNote(note.id)">删除</div>
            </div>
          </div>
          <div class="content">{{ note.content }}</div>
          <div class="replies" v-if="note.replies?.length">
            <div class="reply-item" v-for="reply in note.replies" :key="reply.id">
              <div class="header reply-header">
                <div class="user">
                  <div class="avatar"><IconUser /></div>
                  <div class="user-info">
                    <div class="username">{{ reply.user }}</div>
                    <div class="time">{{ new Date(reply.time).toLocaleString() }}</div>
                  </div>
                </div>
                <div class="btns">
                  <div class="btn delete" @click.stop="deleteReply(note.id, reply.id)">删除</div>
                </div>
              </div>
              <div class="content">{{ reply.content }}</div>
            </div>
          </div>
          <div class="note-reply" v-if="replyNoteId === note.id">
            <TextArea :padding="6" v-model:value="replyContent" placeholder="输入回复内容" :rows="1" @enter.prevent="createNoteReply()" />
            <div class="reply-btns">
              <Button class="btn" size="small" @click="replyNoteId = ''">取消</Button>
              <Button class="btn" size="small" type="primary" @click="createNoteReply()">回复</Button>
            </div>
          </div>
        </div>
        <div class="empty" v-if="!notes.length">本页暂无评论</div>
      </div>
      <div class="send">
        <TextArea 
          ref="textAreaRef"
          v-model:value="content"
          :padding="6"
          :placeholder="`输入评论（为${handleElementId ? '选中元素' : '当前页幻灯片' }）`"
          :rows="2"
          @focus="replyNoteId = ''; activeNoteId = ''"
          @enter.prevent="createNote()"
        />
        <div class="footer">
          <IconDelete class="btn icon" v-tooltip="'清空本页评论'" style="flex: 1" @click="clear()" />
          <Button type="primary" class="btn" style="flex: 12" @click="createNote()"><IconPlus /> 添加评论</Button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed, watch, nextTick, useTemplateRef } from 'vue'
import { storeToRefs } from 'pinia'
import { nanoid } from 'nanoid'
import { useMainStore, useSlidesStore } from '@/store'
import type { Note } from '@/types/slides'

import TextArea from '@/components/TextArea.vue'
import Button from '@/components/Button.vue'

const slidesStore = useSlidesStore()
const mainStore = useMainStore()
const { slideIndex, currentSlide } = storeToRefs(slidesStore)
const { handleElementId } = storeToRefs(mainStore)

const content = ref('')
const replyContent = ref('')
const notes = computed(() => currentSlide.value?.notes || [])
const activeNoteId = ref('')
const replyNoteId = ref('')
const textAreaRef = useTemplateRef<InstanceType<typeof TextArea>>('textAreaRef')
const notesRef = useTemplateRef<HTMLElement>('notesRef')

watch(slideIndex, () => {
  activeNoteId.value = ''
  replyNoteId.value = ''
})

const scrollToBottom = () => {
  if (notesRef.value) {
    notesRef.value.scrollTop = notesRef.value.scrollHeight
  }
}

const createNote = () => {
  if (!content.value) {
    if (textAreaRef.value) textAreaRef.value.focus()
    return
  }

  const newNote: Note = {
    id: nanoid(),
    content: content.value,
    time: new Date().getTime(),
    user: '测试用户',
  }
  if (handleElementId.value) newNote.elId = handleElementId.value

  const newNotes = [
    ...notes.value,
    newNote,
  ]
  slidesStore.updateSlide({ notes: newNotes })

  content.value = ''

  nextTick(scrollToBottom)
}

const deleteNote = (id: string) => {
  const newNotes = notes.value.filter(note => note.id !== id)
  slidesStore.updateSlide({ notes: newNotes })
}

const createNoteReply = () => {
  if (!replyContent.value) return
  
  const currentNote = notes.value.find(note => note.id === replyNoteId.value)
  if (!currentNote) return

  const newReplies = [
    ...currentNote.replies || [],
    {
      id: nanoid(),
      content: replyContent.value,
      time: new Date().getTime(),
      user: '测试用户',
    },
  ]
  const newNote: Note = {
    ...currentNote,
    replies: newReplies,
  }
  const newNotes = notes.value.map(note => note.id === replyNoteId.value ? newNote : note)
  slidesStore.updateSlide({ notes: newNotes })

  replyContent.value = ''
  replyNoteId.value = ''

  nextTick(scrollToBottom)
}

const deleteReply = (noteId: string, replyId: string) => {
  const currentNote = notes.value.find(note => note.id === noteId)
  if (!currentNote || !currentNote.replies) return
  
  const newReplies = currentNote.replies.filter(reply => reply.id !== replyId)
  const newNote: Note = {
    ...currentNote,
    replies: newReplies,
  }
  const newNotes = notes.value.map(note => note.id === noteId ? newNote : note)
  slidesStore.updateSlide({ notes: newNotes })
}

const handleClickNote = (note: Note) => {
  activeNoteId.value = note.id

  if (note.elId) {
    const elIds = currentSlide.value.elements.map(item => item.id)
    if (elIds.includes(note.elId)) {
      mainStore.setActiveElementIdList([note.elId])
    }
    else mainStore.setActiveElementIdList([])
  }
  else mainStore.setActiveElementIdList([])
}

const clear = () => {
  slidesStore.updateSlide({ notes: [] })
}
</script>

<style lang="scss" scoped>
.notes-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
  margin: -12px;
}
.panel-title {
  flex-shrink: 0;
  padding: 12px 12px 8px;
  font-size: 14px;
  font-weight: 500;
  color: #333;
}
.container {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding: 0 12px 12px;
}
.notes {
  flex: 1;
  overflow: auto;
  padding: 2px 0;
  
  @include overflow-overlay();
}
.empty {
  width: 100%;
  height: 100%;
  color: #999;
  font-style: italic;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 12px;
}
.note {
  border: 1px solid #eee;
  border-radius: 4px;
  padding: 10px;
  font-size: 12px;

  & + .note {
    margin-top: 10px;
  }
  &.active {
    background-color: #f7f7f7;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 8px;

    &:hover {
      .btns {
        opacity: 1;
      }
    }
  }
  .user {
    display: flex;
    align-items: center;

    .avatar {
      width: 30px;
      height: 30px;
      border-radius: 50%;
      background-color: #42ba97;
      color: #fff;
      font-size: 18px;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-right: 10px;
      flex-shrink: 0;
    }

    .username {
      font-size: 13px;
      font-weight: 500;
    }
    .time {
      font-size: 11px;
      color: #aaa;
      margin-top: 2px;
    }
  }
  .btns {
    display: flex;
    align-items: center;
    opacity: 0;
    transition: opacity 0.2s;

    .btn {
      margin-left: 5px;
      cursor: pointer;
      font-size: 12px;

      &:hover {
        text-decoration: underline;
        color: $themeColor;
      }
    }
  }
  .content {
    font-size: 13px;
    line-height: 1.6;
    word-break: break-word;
  }
  .replies {
    margin-left: 20px;
    margin-top: 15px;

    .reply-item {
      margin-top: 10px;

      .content {
        margin-top: 5px;
      }
    }
  }
}
.note-reply {
  margin-top: 15px;
}
.reply-btns {
  margin-top: 5px;
  text-align: right;

  .btn {
    margin-left: 8px;
  }
}
.send {
  height: 120px;
  flex-shrink: 0;
  text-align: right;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  
  .footer {
    margin-top: 10px;
    display: flex;

    .btn {      
      &.icon {
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 18px;
        color: #666;
        cursor: pointer;
        
        &:hover {
          color: $themeColor;
        }
      }
    }

    .btn + .btn {
      margin-left: 8px;
    }
  }
}
</style>

