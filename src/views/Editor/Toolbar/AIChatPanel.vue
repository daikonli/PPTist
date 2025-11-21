<template>
  <div class="ai-chat-panel">
    <div class="messages" ref="messagesRef">
      <div class="message" v-for="message in messages" :key="message.id" :class="{ 'ai-message': message.role === 'assistant' }">
        <div class="header">
          <div class="user">
            <div class="avatar" :class="message.role">
              <IconUser v-if="message.role === 'user'" />
              <span v-else class="ai-icon">AI</span>
            </div>
            <div class="user-info">
              <div class="username">{{ message.role === 'user' ? '我' : 'AI 助手' }}</div>
              <div class="time">{{ new Date(message.time).toLocaleString() }}</div>
            </div>
          </div>
          <div class="btns" v-if="message.role === 'user'">
            <div class="btn delete" @click="deleteMessage(message.id)">删除</div>
          </div>
        </div>
        <div class="content">{{ message.content }}</div>
      </div>
      <div class="empty" v-if="!messages.length">
        <div class="empty-icon">💬</div>
        <div class="empty-text">开始与 AI 对话</div>
        <div class="empty-hint">您可以询问关于 PPT 制作的任何问题</div>
      </div>
      <div class="message ai-message" v-if="isLoading">
        <div class="header">
          <div class="user">
            <div class="avatar assistant">
              <span class="ai-icon">AI</span>
            </div>
            <div class="user-info">
              <div class="username">AI 助手</div>
            </div>
          </div>
        </div>
        <div class="content loading">
          <div class="typing-indicator">
            <span></span>
            <span></span>
            <span></span>
          </div>
        </div>
      </div>
    </div>
    <div class="send">
      <TextArea 
        ref="textAreaRef"
        v-model:value="content"
        :padding="8"
        placeholder="输入您的问题..."
        :rows="2"
        @enter.prevent="sendMessage()"
      />
      <div class="footer">
        <IconDelete class="btn icon" v-tooltip="'清空对话'" style="flex: 1" @click="clearMessages()" />
        <Button type="primary" class="btn" style="flex: 12" @click="sendMessage()" :disabled="isLoading || !content.trim()">
          <IconSend /> 发送
        </Button>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, nextTick, useTemplateRef } from 'vue'
import { nanoid } from 'nanoid'

import TextArea from '@/components/TextArea.vue'
import Button from '@/components/Button.vue'

interface Message {
  id: string
  role: 'user' | 'assistant'
  content: string
  time: number
}

const content = ref('')
const messages = ref<Message[]>([])
const isLoading = ref(false)
const textAreaRef = useTemplateRef<InstanceType<typeof TextArea>>('textAreaRef')
const messagesRef = useTemplateRef<HTMLElement>('messagesRef')

const scrollToBottom = () => {
  nextTick(() => {
    if (messagesRef.value) {
      messagesRef.value.scrollTop = messagesRef.value.scrollHeight
    }
  })
}

const sendMessage = async () => {
  if (!content.value.trim() || isLoading.value) {
    if (textAreaRef.value) textAreaRef.value.focus()
    return
  }

  const userMessage: Message = {
    id: nanoid(),
    role: 'user',
    content: content.value,
    time: new Date().getTime(),
  }

  messages.value.push(userMessage)
  content.value = ''
  scrollToBottom()

  // 模拟 AI 回复
  isLoading.value = true
  scrollToBottom()

  try {
    // 这里应该调用实际的 AI API
    // 目前使用模拟响应
    await simulateAIResponse(userMessage.content)
  } finally {
    isLoading.value = false
  }
}

const simulateAIResponse = async (userInput: string) => {
  // 模拟 API 延迟
  await new Promise(resolve => setTimeout(resolve, 1500))

  const responses = [
    '我理解您的问题。关于 PPT 制作，我建议您可以从以下几个方面入手：\n\n1. 确定主题和目标受众\n2. 设计统一的视觉风格\n3. 控制每页的信息量\n4. 使用高质量的图片和图表',
    '这是一个很好的问题！让我为您详细解答...',
    '根据您的需求，我建议采用以下方案：\n\n• 使用简洁的设计风格\n• 突出重点内容\n• 保持视觉一致性',
    '您可以尝试使用模板功能，这样能够快速创建专业的演示文稿。',
  ]

  const aiMessage: Message = {
    id: nanoid(),
    role: 'assistant',
    content: responses[Math.floor(Math.random() * responses.length)],
    time: new Date().getTime(),
  }

  messages.value.push(aiMessage)
  scrollToBottom()
}

const deleteMessage = (id: string) => {
  messages.value = messages.value.filter(msg => msg.id !== id)
}

const clearMessages = () => {
  messages.value = []
}
</script>

<style lang="scss" scoped>
.ai-chat-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
  margin: -12px;
  overflow: hidden;
}

.messages {
  flex: 1;
  overflow: auto;
  padding: 12px;
  
  @include overflow-overlay();
}

.empty {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: #999;
  
  .empty-icon {
    font-size: 48px;
    margin-bottom: 16px;
    opacity: 0.6;
  }
  
  .empty-text {
    font-size: 16px;
    font-weight: 500;
    margin-bottom: 8px;
  }
  
  .empty-hint {
    font-size: 12px;
    font-style: italic;
  }
}

.message {
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 12px;
  font-size: 12px;
  margin-bottom: 12px;
  background-color: #fff;

  &.ai-message {
    background-color: #f8f9fa;
    border-color: #e3e8ef;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 10px;

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
      width: 32px;
      height: 32px;
      border-radius: 50%;
      color: #fff;
      font-size: 16px;
      display: flex;
      justify-content: center;
      align-items: center;
      margin-right: 10px;
      flex-shrink: 0;

      &.user {
        background-color: #42ba97;
      }

      &.assistant {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      }

      .ai-icon {
        font-weight: 700;
        font-size: 14px;
      }
    }

    .username {
      font-size: 13px;
      font-weight: 500;
      color: #333;
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
      color: #666;

      &:hover {
        text-decoration: underline;
        color: $themeColor;
      }
    }
  }

  .content {
    font-size: 13px;
    line-height: 1.8;
    word-break: break-word;
    white-space: pre-wrap;
    color: #333;

    &.loading {
      padding: 8px 0;
    }
  }
}

.typing-indicator {
  display: flex;
  align-items: center;
  gap: 4px;

  span {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background-color: #999;
    animation: typing 1.4s infinite;

    &:nth-child(2) {
      animation-delay: 0.2s;
    }

    &:nth-child(3) {
      animation-delay: 0.4s;
    }
  }
}

@keyframes typing {
  0%, 60%, 100% {
    opacity: 0.3;
    transform: translateY(0);
  }
  30% {
    opacity: 1;
    transform: translateY(-8px);
  }
}

.send {
  flex-shrink: 0;
  padding: 12px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  border-top: 1px solid #eee;
  background-color: #fff;
  
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

